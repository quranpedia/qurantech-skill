# Quran Data Sources & APIs

## Table of Contents
- [Quran Text & Script](#quran-text--script)
- [Audio Recitations](#audio-recitations)
- [Translations](#translations)
- [Tafsir](#tafsir)
- [Mushaf Images & SVG](#mushaf-images--svg)
- [Qira'at & Ayah Mapping](#qiraat--ayah-mapping)
- [Word-Level Data](#word-level-data)
- [Search Services](#search-services)
- [Pre-built Packages & Libraries](#pre-built-packages--libraries)
- [CMS & Management Platforms](#cms--management-platforms)

## Quran Text & Script

| Source | Formats | Qira'at | Notes |
|--------|---------|---------|-------|
| **King Fahd Quran Complex** | Downloads (formats vary per riwayah — verify what's actually published) | Hafs, Warsh, Shubah, Qalun, Duri, Susi, Hesham | Official, most authoritative. Fonts at `fonts.qurancomplex.gov.sa` |
| **Tanzil.net** | BSV, XML, MySQL | Hafs (verified) | Thoroughly verified text in multiple scripts (Uthmani, Simple, Simple Enhanced). Requires attribution |
| **Quran Foundation API (v4)** | REST JSON | Hafs | `api-docs.quran.foundation` — rich metadata, OAuth2 support. Use `fields=text_uthmani` and specific `translation IDs` to reduce payload |
| **Al Quran Cloud API** | REST JSON | Hafs | `alquran.cloud/api` — Free, no auth required. Good for prototypes |
| **EveryAyah** | XML, PNG, JPG | Hafs | Images + text; useful for mushaf page rendering |
| **QUL (by Tarteel)** | API + GitHub | Hafs | `qul.tarteel.ai` — The Toolkit for Muslim Developers. 193 translations, 114 tafsirs, 62 segmented audio sets with word-level timestamps, 19 mushaf layouts, 18 Quran fonts, 77K morphological entries, 2.5K topics, mutashabihat, transliterations. Open-sourced on GitHub |
| **QuranPedia API** | REST JSON + dumps | 16 riwayat | `api.quranpedia.net/v1` — Free, no auth. Largest tafsir/translation library of any API. 16 mushafs with SVG/PNG page packs + fonts, i'rab (books + morphology + syntax), fatwas, notes, topics, reciters with timing ZIPs, search, print-book image verification, delta sync (`/v1/changes`) + bulk dumps. Full reference: [quranpedia-api.md](quranpedia-api.md). Embed widget: [quranpedia-embed.md](quranpedia-embed.md) |
| **QuranHub / QuranAI** | REST JSON | Multiple | `api.quranhub.com` — mushaf layouts, semantic search, QRC. Multiple mushaf layouts (IndoPak 13-line, Madinah 15-line) |
| **fawazahmed0/quran-api** | JSON (GitHub) | Hafs, Warsh | 400+ translations/tafsirs. Includes Warsh text split by ayah. Community-maintained, verify accuracy |
| **KSU Electronic Mushaf** | Web | Hafs | `quran.ksu.edu.sa` — text + tafsirs + audio |

## Audio Recitations

| Source | Type | Features |
|--------|------|----------|
| **MP3Quran** | Full surahs, ayah-by-ayah | `mp3quran.net/ar/api` — largest reciter library, multiple qira'at. Community's top pick for free audio |
| **EveryAyah** | Ayah-by-ayah | MP3 files per ayah: `/{reciter_id}/{surah}{ayah}.mp3`. XML metadata at `everyayah.com/data/XML/` |
| **QuranicAudio** | Full & ayah-level | Streaming-friendly |
| **Quran Foundation Audio API** | Ayah + word-level timestamps | `api-docs.quran.foundation/docs/sdk/audio/` — best for word-by-word highlighting |
| **Tahbeer Project** | All 10 qira'at | Professional recordings of 20 riwayat. ~7 years of work by Sheikh Saber Abdelhakm. iOS + Android apps |
| **QuranPedia reciters** | Full surah + ayah-level, timings | `/v1/reciters` — reciters across riwayat with downloadable ayah-timing ZIPs per recitation. See [quranpedia-api.md](quranpedia-api.md) |
| **Buraaq Word Audio Dataset** | Word-level | HuggingFace dataset for ML/speech apps |

## Translations

| Source | Languages | Update Model |
|--------|-----------|-------------|
| **Tanzil.net** | 40+ languages | Static download |
| **QUL (Tarteel)** | Multiple | API-driven |
| **Quran Foundation API** | 50+ languages | API — always current |
| **QuranEnc** | Multiple | Verified translations |
| **Fawaz Ahmed quran-api** | Multiple | GitHub repo |

**Important:** Translations are human-authored and may contain errors. Prefer API-driven sources for continuous updates. Always verify copyright before including a translation.

## Tafsir

| Source | Content | Notes |
|--------|---------|-------|
| **Quran Foundation API** | Multiple tafsirs | API-driven, current |
| **QUL** | Curated tafsirs | Includes popular tafsirs |
| **Spa5k tafsir API** | Multiple tafsirs | Open API |

**Legal:** Verify copyright status before including tafsir. Many tafsirs require publisher approval. Digitized versions may contain OCR errors.

## Mushaf Images & SVG

| Source | Format | Mushafs | Features |
|--------|--------|---------|----------|
| **quranpedia/quran-svg** | SVG + JSON | Hafs, Warsh, Qalun, Duri, Shubah | Interactive clickable ayah regions, brotli-compressed |
| **QuranPedia API mushaf packages** | SVG + PNG ZIPs | 16 riwayat | `/v1/mushafs` lists per-mushaf `images` (SVG pack), `images_png`, and `font_file` URLs — the widest riwayah coverage. See [quranpedia-api.md](quranpedia-api.md) |
| **King Fahd Complex** | Images | Multiple | Official mushaf pages |
| **Shamarly** | Images | Hafs | High-quality scans |

## Qira'at & Ayah Mapping

| Source | Data | Notes |
|--------|------|-------|
| **quranpedia/qiraat-ayah-map** | JSON | Maps ayah numbers across 6 counting systems (Kufan, Madinan, Makkan, Basran, Damascene). Covers all 10 qira'at and 20 ruwat |

**Critical:** Different qira'at have different ayah counts (Hafs: 6,236 / Warsh: 6,214 / Basran: 6,204). Never hardcode ayah counts.

## Word-Level Data

| Source | Data |
|--------|------|
| **Quran Foundation API** | Word-by-word translation, transliteration, morphology |
| **QuranWBW** | Word-by-word breakdown platform and repo |
| **Buraaq Word Audio** | Word-level audio segments (HuggingFace) |

## Search Services

| Source | Type | Notes |
|--------|------|-------|
| **Kalimat.dev** | Full-text Arabic search | Production-ready |
| **Alfanous.org** | Semantic/root-based search | Meaning-based discovery |
| **Quran Foundation API** | Text search | API-integrated search |

Prefer production-ready search services over custom implementations.

## Pre-built Packages & Libraries

| Package | Platform | Features |
|---------|----------|----------|
| **MushafImad** (github.com/ibo2001/MushafImad) | Swift/iOS 17+, macOS 14+ | Full mushaf reader: 604 bundled page images, audio playback with verse sync (ayah timing JSON), reciter selection, RealmSwift offline DB, SwiftUI components (MushafView, QuranPlayer), RTL paging, theming, haptic feedback, AirPlay. MIT licensed. Part of Itqan community. |
| **mushaf-imad-android** | Kotlin/Android | Android equivalent — mushaf rendering, bookmarks, audio |
| **mushaf-imad-flutter** | Flutter | Cross-platform: display, bookmarks, search, offline storage |
| **Quran MCP** (mcp.quran.ai) | AI (Claude/ChatGPT) | Semantic search & retrieval via MCP protocol |

**Always check for existing packages before building from scratch.** These libraries handle the hardest parts of Quran app development — mushaf rendering, audio synchronization, offline storage — that are complex and error-prone to reimplement.

### MushafImad Quick Start (iOS/macOS)

MushafImad is the most comprehensive open-source Quran package for Apple platforms:

```swift
// 1. Add package: github.com/ibo2001/MushafImad
// 2. Initialize
try? RealmService.shared.initialize()
FontRegistrar.registerFontsIfNeeded()
// 3. Use
MushafView(initialPage: 1)
    .environmentObject(ReciterService())
    .environmentObject(ToastManager())
```

Key components:
- `MushafView` — main reader with 604 pages, RTL paging, theming
- `QuranPlayer` / `QuranPlayerViewModel` — audio with verse timing sync
- `ReciterService` — reciter selection and persistence
- `AyahTimingService` — loads verse timing JSON for word/ayah highlighting
- Custom layouts: `horizontalPageView()`, `verticalPageView()`, `pageContent()`
- Asset override: `MushafAssets.configuration` for custom colors/images

## Fonts & Typography

| Font | Source | Notes |
|------|--------|-------|
| **KFGQPC HAFS Uthmanic Script** | `fonts.qurancomplex.gov.sa` | Official King Fahd Complex font for Hafs |
| **KFGQPC Uthman Taha Naskh** | King Fahd Complex | Naskh style |
| **KFGQPC Nastaleeq** | King Fahd Complex | For Urdu/Indo-Pak audiences |
| **Amiri / Amiri Quran** | Open source (OFL) | Popular, good browser/cross-platform support |
| **Me Quran** | `qul.tarteel.ai/resources/font/243` | Modern digital Quran typography |
| **PDMS Saleem Quran Font** | `pakdata.com/products/arabicfont` | Nastaleeq for Indo-Pak script |
| **Scheherazade** | SIL International | SIL Arabic font |
| **Quran Foundation per-page fonts** | CDN | Per-page woff2 for pixel-perfect rendering: `verses.quran.foundation/fonts/quran/hafs/v2/woff2/p{PAGE}.woff2` (V4 COLRv1 also available) |

## AI/ML Datasets & Models

| Resource | Type | Notes |
|----------|------|-------|
| **tarteel-ai/whisper-base-ar-quran** | ASR model | Most popular Quran ASR on HuggingFace (30+ spaces) |
| **wav2vec2-base-word-by-word-quran-asr** | ASR model | Word-by-word Quran ASR |
| **QDAT (Univ. of Mosul)** | Audio dataset | Quran recitation dataset for training |
| **Iqra'Eval** | Audio dataset | ~79 hours, various qira'at |
| **prepare-quran-dataset** | Tool | `github.com/obadx/prepare-quran-dataset` |
| **QurSim** | NLP dataset | Ayah similarity pairs (strong/weak/unrelated) |
| **CAMeL Tools** | NLP toolkit | `github.com/CAMeL-Lab/camel_tools` — Arabic morphology, dialect ID |
| **AraBERT** | Language model | `github.com/aub-mind/arabert` — Arabic BERT |

## Open-Source Quran Projects

| Project | Stack | Description |
|---------|-------|-------------|
| **OpenMushaf** | React Native (Expo) | Offline mushaf with Hafs + Warsh, tafsirs. `github.com/adelpro/open-mushaf-native` |
| **OpenTarteel** | Next.js, GunDB | Audio streaming, 30+ reciters, PWA. `github.com/adelpro/open-tarteel` |
| **BAHETH** | Python, Elasticsearch, FAISS | Hybrid semantic+lexical search. `github.com/engsaleh/Baheth-Quran` |
| **SyncQuran** | HTML/JS, WebRTC (PeerJS) | Real-time synchronized mushaf for halaqat. `github.com/hadealahmad/SyncQuran` |
| **check-telawa** | Flask, Whisper | Recitation evaluation. `github.com/engsaleh/check-telawa` |
| **Dhikr al-Huda** | Flutter | Accessibility-focused, by a blind developer. `github.com/ahmed1hegazy/Thikr_al-huda` |
| **Al-Bayan** | — | Accessible Quran app. `github.com/tecwindow/albayan` |
| **QuranApp** | Web | Word-by-word Uthmani rendering. `github.com/oazabir/QuranApp` |

## CMS & Management Platforms

| Platform | Purpose |
|----------|---------|
| **Itqan CMS** | Django 5.2 + Angular 19 + PostgreSQL. `github.com/Itqan-community/cms-backend` |
| **QuranPedia** | Qira'at data and mushaf resources |
| **RATQ** | Development guidelines + tech catalog. `github.com/Itqan-community/RATQ` |

## Community Resources

- **Itqan Platform:** `itqan.dev` — Community of developers serving the Quran
- **Itqan Community Forum:** `community.itqan.dev`
- **Quran Apps Directory:** `quran-apps.itqan.dev` — curated catalog of Quran apps
- **Quran.com Developers:** `quran.com/developers` — API docs, OAuth2, MCP
