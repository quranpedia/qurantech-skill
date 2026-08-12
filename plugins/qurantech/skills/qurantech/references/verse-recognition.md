# Verse Recognition (Audio-to-Ayah Identification)

## Table of Contents
- [Overview](#overview)
- [Offline Tarteel](#offline-tarteel)
- [Pipeline Architecture](#pipeline-architecture)
- [Platform Integration](#platform-integration)
- [Model Details](#model-details)
- [Data Files](#data-files)
- [Experiment Landscape](#experiment-landscape)
- [Best Practices](#best-practices)

## Overview

Verse recognition identifies which surah and ayah a person is reciting from an audio recording. Unlike ASR (which produces a transcript), the goal is structured output: `{surah, ayah, ayah_end, score}`. This enables features like:

- **Mushaf auto-follow:** Scroll to the verse being recited
- **Memorization validation:** Check if the user recited the correct verse
- **Live verse identification:** Show the verse while someone recites nearby
- **Hifz testing:** Automated oral examination

## Offline Tarteel

**Repository:** `github.com/yazinsai/offline-tarteel`
**License:** CC-BY-4.0 (NVIDIA model)

The best open-source solution for offline verse recognition. Ships a quantized ONNX model that runs in browsers, React Native, and Python with no internet required.

**Best model:** NVIDIA FastConformer -- **87% recall**, **115 MB**, **0.33s latency** on Apple Silicon.

**Caveat:** the recall/size/latency figures throughout this file come from the project's own v0.1 benchmark notes; artifact sizes and scores change between releases — verify against the current release before quoting them in designs.

### Getting the Model

Download the quantized ONNX model (131 MB, uint8) from GitHub Releases:

```bash
curl -L -o fastconformer_ar_ctc_q8.onnx \
  https://github.com/yazinsai/offline-tarteel/releases/download/v0.1.0/fastconformer_ar_ctc_q8.onnx
```

Required data files from the repo:
- `data/vocab.json` -- CTC vocabulary (token ID -> character mapping)
- `data/quran.json` -- All 6,236 verses (for matching decoded text to surah:ayah)

### Generate Model from NeMo Checkpoint

```bash
pip install nemo_toolkit[asr]
python -c "
from nemo.collections.asr.models import EncDecHybridRNNTCTCBPEModel
import torch, onnx
from onnxruntime.quantization import quantize_dynamic, QuantType

model = EncDecHybridRNNTCTCBPEModel.from_pretrained('nvidia/stt_ar_fastconformer_hybrid_large_pcd_v1.0')
model.change_decoding_strategy(decoder_type='ctc')
model.export('fastconformer_ar_ctc.onnx')
quantize_dynamic('fastconformer_ar_ctc.onnx', 'fastconformer_ar_ctc_q8.onnx', weight_type=QuantType.QUInt8)
"
```

## Pipeline Architecture

The pipeline has 4 stages:

```
Audio (16kHz mono WAV)
  -> Mel Spectrogram (80-bin, NeMo-compatible)
    -> ONNX Inference (CTC logprobs)
      -> Decode + Match (greedy CTC decode, then fuzzy-match against 6,236 verses)
```

### Stage 1: Audio Input
- 16 kHz sample rate, mono channel
- WAV format preferred; convert from other formats with ffmpeg/pydub
- Dithering applied: `audio + 1e-5 * random_noise`

### Stage 2: Mel Spectrogram
- 80-bin mel filterbank
- NeMo-compatible parameters: `n_fft=512, hop_length=160, win_length=400, fmax=8000, htk=True, norm="slaney"`
- Preemphasis: `0.97` coefficient
- Per-feature normalization (zero mean, unit variance)

### Stage 3: ONNX Inference
- Input: `[1, 80, T]` float32 tensor + `[1]` int64 length tensor
- Output: `[1, T, vocab_size]` CTC logprobs
- Model: `fastconformer_ar_ctc_q8.onnx` (131 MB, uint8 quantized)
- Vocabulary: 1,025-token Arabic BPE (or 69-phoneme Buckwalter variant)

### Stage 4: Decode + Match
- **CTC greedy decode:** argmax per timestep, collapse repeats, remove blanks, join tokens
- **Fuzzy match:** Levenshtein distance against all 6,236 verses in `quran.json`
- **Multi-verse spans:** Also scores consecutive verse combinations (e.g., 2:1-3)
- Arabic text normalization before matching (strip diacritics, normalize alef/taa)

## Platform Integration

### Web / React (ONNX Runtime Web)

Runs entirely in the browser using WebAssembly.

```bash
npm install onnxruntime-web @huggingface/transformers
```

```typescript
import * as ort from "onnxruntime-web/wasm";

// 1. Create session
ort.env.wasm.numThreads = 1;
ort.env.wasm.simd = true;
const session = await ort.InferenceSession.create(modelBuffer, {
  executionProviders: ["wasm"],
});

// 2. Compute mel spectrogram (80-bin, NeMo-compatible)
//    Uses @huggingface/transformers mel_filter_bank + spectrogram
const { features, timeFrames } = computeMelSpectrogram(audioFloat32Array);

// 3. Run inference
const input = new ort.Tensor("float32", features, [1, 80, timeFrames]);
const length = new ort.Tensor("int64", BigInt64Array.from([BigInt(timeFrames)]), [1]);
const results = await session.run({
  [session.inputNames[0]]: input,
  [session.inputNames[1]]: length,
});
const logprobs = results[session.outputNames[0]];

// 4. CTC greedy decode -> fuzzy match against QuranDB
```

Key implementation files in `web/frontend/src/`:
- `worker/mel.ts` -- Mel spectrogram (NeMo-compatible)
- `worker/ctc-decode.ts` -- CTC greedy decoder
- `lib/quran-db.ts` -- Verse matching with Levenshtein distance
- `lib/normalizer.ts` -- Arabic text normalization

**Latency:** ~0.5-1s in browser WASM.

### React Native (ONNX Runtime Mobile)

```bash
npm install onnxruntime-react-native
```

```typescript
import { InferenceSession, Tensor } from "onnxruntime-react-native";

// Bundle the model in app assets, or download on first launch
const session = await InferenceSession.create("path/to/fastconformer_ar_ctc_q8.onnx");

// Same inference pattern as web:
// 1. Compute 80-bin mel spectrogram from 16kHz audio
// 2. Create input tensors: features [1, 80, T] + length [1]
// 3. session.run() -> CTC logprobs
// 4. Greedy decode + QuranDB match
```

The mel spectrogram, CTC decoder, and QuranDB matching logic from the web implementation are pure TypeScript with no browser-specific APIs -- they work directly in React Native.

### Python

**Option A: ONNX Runtime (recommended for production)**

```bash
pip install onnxruntime numpy soundfile librosa
```

```python
import numpy as np
import onnxruntime as ort
import json, librosa

# Load model + vocab
session = ort.InferenceSession("fastconformer_ar_ctc_q8.onnx")
vocab = json.load(open("vocab.json"))
id_to_char = {int(k): v for k, v in vocab.items()}
blank_id = max(id_to_char.keys())

# Load audio at 16kHz
audio, sr = librosa.load("recitation.wav", sr=16000)

# Compute NeMo-compatible mel spectrogram
audio = audio + 1e-5 * np.random.randn(len(audio))  # dither
audio = np.append(audio[0], audio[1:] - 0.97 * audio[:-1])  # preemphasis
mel = librosa.feature.melspectrogram(
    y=audio, sr=16000, n_fft=512, hop_length=160, win_length=400,
    n_mels=80, fmax=8000, htk=True, norm="slaney"
)
mel = np.log(mel + 1e-5)
mel = (mel - mel.mean(axis=1, keepdims=True)) / (mel.std(axis=1, keepdims=True) + 1e-10)

# Run inference
features = mel.astype(np.float32)[np.newaxis]  # [1, 80, T]
length = np.array([mel.shape[1]], dtype=np.int64)
logprobs = session.run(None, {
    session.get_inputs()[0].name: features,
    session.get_inputs()[1].name: length,
})[0]  # [1, T, vocab_size]

# CTC greedy decode
ids = logprobs[0].argmax(axis=1)
prev, tokens = -1, []
for i in ids:
    if i != prev and i != blank_id:
        tokens.append(id_to_char.get(i, ""))
    prev = i
transcript = "".join(tokens).replace("\u2581", " ").strip()
# Then match against quran.json using Levenshtein distance
```

**Option B: Use the repo directly**

```bash
git clone https://github.com/yazinsai/offline-tarteel.git
cd offline-tarteel && pip install -e ".[nemo]"
```

```python
from experiments.nvidia_fastconformer.run import predict
result = predict("recitation.wav")
# {"surah": 1, "ayah": 1, "ayah_end": 3, "score": 0.92, "transcript": "..."}
```

### Streaming Recognition

For real-time verse identification during recitation, feed 300ms audio chunks to a `RecitationTracker` that accumulates predictions. Streaming accuracy is lower than full-file (74% vs 87% on test corpus) -- the streaming tracker pipeline is the main bottleneck, not the model.

## Model Details

| Property | Value |
|----------|-------|
| **Base model** | `nvidia/stt_ar_fastconformer_hybrid_large_pcd_v1.0` |
| **ONNX file** | `fastconformer_ar_ctc_q8.onnx` (131 MB, uint8 quantized) |
| **Input** | 80-bin mel spectrogram, 16 kHz, mono |
| **Output** | CTC logprobs over 1025-token Arabic BPE vocabulary |
| **Recall** | 87% on 54-sample benchmark (user recordings, professional, crowdsourced) |
| **Latency** | 0.33s on Apple Silicon, ~0.5-1s in browser WASM |
| **License** | CC-BY-4.0 (NVIDIA model) |

### Phoneme Variant

A 69-phoneme Buckwalter CTC variant (`fastconformer_phoneme_q8.onnx`, 131 MB) is the current best for streaming. Decodes to phoneme sequences instead of Arabic BPE, then matches via phoneme-level Levenshtein distance.

- Non-streaming: 83% (v1), 74% (v2)
- Streaming: 51% (v1), 74% (v2)

## Data Files

| File | Description | Size |
|------|-------------|------|
| `vocab.json` | Token ID -> character mapping for CTC decode | Small |
| `quran.json` | All 6,236 verses (uthmani + cleaned text) | Small |
| `phoneme_cache.pkl` | Pre-computed phoneme reference for all verses | 7.6 MB |
| `phoneme_ngram_index_5.pkl` | N-gram index for phoneme matching | 6.0 MB |

## Experiment Landscape

The offline-tarteel repo tested 12+ approaches. Key findings:

| Approach | Recall | Size | Latency | Verdict |
|----------|--------|------|---------|---------|
| **FastConformer (shipped)** | 87% | 115 MB | 0.33s | Best practical balance |
| w2v-phonemes/large | 100% | 970 MB | ~12s | Perfect but too large/slow |
| w2v-phonemes/base | 89% | 116 MB | ~3-6s | Accurate but 10x slower than FastConformer |
| FastConformer phoneme (streaming) | 74% | 131 MB | real-time | Best streaming model |
| CTC forced alignment | 83% | 1.2 GB | 3.2s | Strong baseline, too large |
| Rabah pruned CTC | 72% | 145 MB | 7.0s | Viable but slow |
| Tarteel whisper-base | 72% | 290 MB | ~3s | Not competitive |
| Embedding search (HuBERT) | 0% cross-speaker | 397 MB | 0.3s | Encodes speaker, not content |
| Contrastive (CLIP-style) | 0% | 900 MB | 0.1s | English encoder, useless for Arabic |

**Key insights:**
- Phoneme-based matching works perfectly (100% with large model) -- proves the approach
- CTC beam search without a language model produces near-identical hypotheses
- `first_n` layer pruning vastly outperforms `evenly_spaced` (72% vs 56%)
- TLOG/synthetic data helps in small doses (5/verse) but regresses when scaled up
- Streaming accuracy gap (51% vs 83% non-streaming) indicates tracker pipeline, not model, is the bottleneck

## Design Constraints

- **Offline-first:** No network calls at inference time. Model + index + reference data all ship with the app
- **Size budget:** Under 200 MB total (model + any index)
- **Speed target:** Sub-1-second latency on modern devices
- **Speaker-invariant:** Must work across accents, recording quality, and recitation styles
- **Full Quran coverage:** All 6,236 verses, including short verses (3-4 words)

## Best Practices

- **Audio must be 16 kHz mono.** Convert with ffmpeg/pydub before processing. Higher sample rates waste compute without improving accuracy.
- **Normalize Arabic text before matching.** Strip diacritics, normalize alef forms (alef madda, alef hamza -> bare alef), normalize taa marbuta.
- **Use Levenshtein fuzzy matching, not exact matching.** ASR transcripts are never perfect -- fuzzy matching recovers most errors.
- **Score multi-verse spans.** Users often recite multiple consecutive ayahs. Score spans of 2-6 verses, not just individual verses.
- **Bundle `quran.json` with the app.** The verse database is small and must be available offline.
- **Download the model on first launch, not at install.** 131 MB is too large for app bundle on some platforms. Show download progress.
- **Handle short verses carefully.** Verses with 3-4 words (e.g., Al-Ikhlas, Ya-Sin letters) are the hardest to identify. Consider requiring longer audio clips for confident matches.
- **Use confidence scores.** Only show results above a threshold (e.g., 0.7). Low-confidence matches are usually wrong.
- **Test with diverse audio.** Professional studio recordings, phone recordings with ambient noise, and crowdsourced recordings from various accents behave very differently.
- **ONNX Runtime has non-determinism.** Run benchmarks 3x and average for reliable measurements.
- **For streaming, feed 300ms chunks.** Shorter chunks don't accumulate enough signal; longer chunks add latency.
