# Audio Recitations

## Table of Contents
- [Audio Types](#audio-types)
- [Sources](#sources)
- [Playback Modes](#playback-modes)
- [Word-Level Highlighting](#word-level-highlighting)
- [Offline Audio](#offline-audio)
- [Best Practices](#best-practices)

## Audio Types

| Type | Description | Use Case |
|------|-------------|----------|
| **Ayah-by-ayah** | Separate audio file per ayah | Memorization, repeat mode, precise navigation |
| **Continuous (full surah)** | Single file per surah with timestamp markers | Listening sessions, background recitation |
| **Word-level** | Audio segmented or timestamped per word | Word-by-word learning, pronunciation practice |
| **Page-level** | Audio for a mushaf page | Mushaf-following mode |

## Sources

| Source | Types Available | Notes |
|--------|----------------|-------|
| **MP3Quran** | Full surah, ayah-level | Large library of reciters, multiple qira'at. API available |
| **EveryAyah** | Ayah-by-ayah MP3 | Simple file structure: `/{reciter_id}/{surah}{ayah}.mp3` |
| **QuranicAudio** | Full & ayah-level | Streaming-friendly |
| **Quran Foundation Audio API** | Ayah + word-level timestamps | Best for synchronized highlighting |
| **QuranPedia** | Reciters + ayah-timing ZIPs | Multi-riwayah reciters with downloadable timing files ([quranpedia-api.md](quranpedia-api.md)) |
| **Buraaq Word Audio Dataset** | Word-level segments | HuggingFace dataset, useful for ML/speech apps |

## Model Rawi, Reciter, and Recitation Separately

A common data-modeling conflation: **rawi** = the transmission (defines the *text*), **reciter** = the person, **recitation** = one recording set of a reciter in a specific rawi (murattal vs mujawwad, different bitrates). Audio files, timings, and availability belong to the *recitation*. See [data-models.md](data-models.md).

## Filling Per-Ayah Audio Gaps

Many reciters exist only as full-surah recordings. To offer ayah-level playback for them, split with ffmpeg using published ayah timings where available, or a duration-based heuristic otherwise — and put a **human review step** (listen, approve, re-slice a boundary) before publishing. Never ship machine-split ayah audio unreviewed: a cut mid-word violates the adab of recitation.

## Playback Modes

Design for these common user needs:

### Verse-by-verse
- Play one ayah, pause, wait for user action (or auto-advance).
- Support repeat count per ayah (common for memorization: play 3x, then move to next).
- Show active ayah highlighted in text.

### Continuous
- Play through a range (page, surah, juz) without stopping.
- Use timestamp data to highlight the current ayah as audio progresses.
- Support background playback with lock-screen controls (mobile).

### Word-by-word
- Highlight each word as it's recited.
- Requires word-level timestamp data (Quran Foundation API).
- Essential for learning pronunciation and tajweed.

### Repeat/Loop
- Repeat a single ayah N times.
- Repeat a range of ayahs (e.g., ayah 1-5, loop).
- Adjustable speed (0.5x–1.5x) if the audio engine supports it.

## Word-Level Highlighting

**Data requirement:** Word-level timestamps mapping each word to its start/end time in the audio file.

**Sources for timestamps:**
- Quran Foundation Audio API — provides word segments for select reciters
- Buraaq dataset — pre-segmented word audio

**Implementation pattern:**
1. Load timestamp data for the selected reciter + surah.
2. During playback, track `currentTime` against timestamp ranges.
3. Apply a highlight class/style to the word whose time range contains `currentTime`.
4. Handle edge cases: pauses between words, tajweed elongations, idgham (merging).

**Performance:** Pre-process timestamps into a sorted array for binary search lookup rather than linear scanning on every time update.

## Offline Audio

- **Download strategy:** Let users download by surah, juz, or full mushaf.
- **Storage estimates:** ~500MB–1GB per reciter for full Quran (MP3 128kbps).
- **Show download progress** and allow partial downloads (resume-capable).
- **Store metadata** about what's downloaded so the app knows when to stream vs. play locally.
- **Cache management:** Provide a way to delete downloaded audio per reciter/surah.

## Audio Alignment & Timestamp Tools

For generating timestamps from audio recordings:

| Tool | Method | Level |
|------|--------|-------|
| **quran-align** | Forced alignment | Ayah-level. `github.com/cpfair/quran-align` |
| **whisper-timestamped** | Whisper + confidence | Word-level. `github.com/linto-ai/whisper-timestamped` |
| **WhisperX** | Whisper + wav2vec 2.0 | Word-level (precise). `github.com/m-bain/whisperX` |
| **Aeneas** | Text-audio sync | Segment-level. Python/C. AGPL v3. `github.com/readbeyond/aeneas` |
| **quran_timing_files** | Pre-computed | Ready-to-use. `github.com/anassaiyed/quran_timing_files` |

**Critical:** Audio must be WAV Mono 16kHz for ASR models. Use `pydub` + `ffmpeg` for format conversion.

## Verse Recognition (Audio-to-Ayah Identification)

For identifying which surah/ayah is being recited from audio (not just transcription), see **[verse-recognition.md](verse-recognition.md)**. The recommended solution is **offline-tarteel** (`github.com/yazinsai/offline-tarteel`): a quantized ONNX model (131 MB) that runs in browsers, React Native, and Python with 87% recall and 0.33s latency -- no internet required.

## ASR for Recitation Evaluation

| Model | Notes |
|-------|-------|
| **tarteel-ai/whisper-base-ar-quran** | Most popular (30+ HuggingFace spaces). Fine-tuned on Quran |
| **wav2vec2-base-word-by-word-quran-asr** | Word-by-word Quran ASR |

**Warning:** low WER on clean benchmarks (Quran-tuned Whisper models report roughly 5–15%) does not mean tajweed competence. Standard ASR treats Arabic like any language — it misses the difference between "correct pronunciation" and "tajweed-compliant recitation," and errors concentrate exactly where tajweed matters (idgham, ghunnah, madd). A Quran-specific evaluation metric accounting for tajweed rules is still needed; never present ASR output as a tajweed judgment.

**Evaluation tools:**
- **check-telawa** (`github.com/engsaleh/check-telawa`) — Flask + Whisper + jiwer for WER calculation
- **WER (Word Error Rate)** — standard but insufficient for Quran
- **PER (Phoneme Error Rate)** — better for tajweed analysis at makhraj level

## Best Practices

- **Reciter attribution is mandatory.** Always show the reciter name when playing audio.
- **Never auto-play** without explicit user action.
- **Pre-buffer the next ayah** during playback for gapless transitions.
- **Handle missing audio gracefully.** Not all reciters have complete recordings. Show which surahs/ayahs are available.
- **Support multiple reciters.** Users have strong preferences — offer a reciter selection UI.
- **Audio should start at ayah boundaries**, never mid-ayah.
- **Background audio** must work on mobile (lock screen, notification controls).
- **Respect the user's qira'a selection.** A Warsh reciter should not be offered when the user is reading the Hafs mushaf (unless explicitly browsing reciters).
- **Noisy recordings cause poor alignment.** Alignment tools work best on clean, studio-quality audio.
- **Separate reading and listening architectures.** Community experience shows audio streaming apps differ fundamentally from page-based mushaf display — consider separate modules or even separate apps.
