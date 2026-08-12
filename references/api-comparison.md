# Quran API Comparison

## Table of Contents
- [API Overview](#api-overview)
- [Feature Matrix](#feature-matrix)
- [Detailed Comparison](#detailed-comparison)
- [Choosing the Right API](#choosing-the-right-api)

## API Overview

| API | URL | Auth | Rate Limits | Status |
|-----|-----|------|-------------|--------|
| **Quran Foundation (v4)** | api-docs.quran.foundation | OAuth2 optional | Generous | Active, well-maintained |
| **Al Quran Cloud** | alquran.cloud/api | None | Open | Active, no auth required |
| **Tanzil.net** | tanzil.net/download | None | N/A (download) | Static files, verified |
| **QuranPedia API** | api.quranpedia.net/v1 | None | Open | Active, comprehensive |
| **QUL (Tarteel)** | qul.tarteel.ai | None | Open | Active, open-sourced |
| **QuranHub / QuranAI** | api.quranhub.com | API key | — | Active, mushaf layouts |
| **EveryAyah** | everyayah.com | None | N/A (static) | Active, static files |
| **MP3Quran** | mp3quran.net/ar/api | None | — | Active |
| **Alfanous** | alfanous.org | None | — | Active |
| **QuranEnc** | quranenc.com/en/home/api | — | — | Active |
| **Quran Tafseer API** | api.quran-tafseer.com | None | — | Active |
| **Fawaz Ahmed** | GitHub repo | None | N/A (static) | Community-maintained, 400+ translations |
| **Quran MCP** | mcp.quran.ai | MCP protocol | — | Active (AI integration) |

## Feature Matrix

| Feature | Quran Foundation | QuranPedia | QUL | Tanzil | EveryAyah | MP3Quran |
|---------|-----------------|------------|-----|--------|-----------|---------|
| Quran text (Uthmani) | ✅ | ✅ | ✅ | ✅ | ✅ | ❌ |
| Multiple mushafs | ❌ | ✅ | ✅ (19 layouts) | ❌ | ❌ | ❌ |
| Translations | ✅ (50+) | ✅ (largest) | ✅ (193) | ✅ (40+) | ❌ | ❌ |
| Tafsir | ✅ | ✅ (largest) | ✅ (114) | ❌ | ❌ | ❌ |
| Print-book comparison | ❌ | ✅ (unique) | ❌ | ❌ | ❌ | ❌ |
| I'rab | ❌ | ✅ (books + morphology + syntax tree) | ❌ | ❌ | ❌ | ❌ |
| Audio (ayah) | ✅ | ✅ (+ timing ZIPs) | ✅ (62 segmented) | ❌ | ✅ | ✅ |
| Audio (word-level) | ✅ | ❌ (ayah-level timings; word audio for qira'at variants) | ✅ (timestamps) | ❌ | ❌ | ❌ |
| Word morphology | ✅ | ✅ (QAC segments + treebank) | ✅ (77K entries) | ❌ | ❌ | ❌ |
| Topics/themes | ❌ | ✅ | ✅ (2.5K topics) | ❌ | ❌ | ❌ |
| Mutashabihat | ❌ | ✅ (services) | ✅ (5.2K entries) | ❌ | ❌ | ❌ |
| Delta sync + bulk dumps | ❌ | ✅ (unique) | ✅ (downloads) | ✅ (static) | ❌ | ❌ |
| Fonts | ❌ | ✅ (per mushaf) | ✅ (18 fonts) | ❌ | ❌ | ❌ |
| Search | ✅ | ✅ | ❌ | ❌ | ❌ | ❌ |
| Multiple qira'at | ❌ (Hafs) | ✅ | ❌ | ❌ | ❌ | ✅ |
| Fatwas | ❌ | ✅ | ❌ | ❌ | ❌ | ❌ |
| User accounts/sync | ✅ (OAuth2) | ❌ | ❌ | ❌ | ❌ | ❌ |
| Auth required | Optional | No | No | N/A | No | No |

**Alfanous** — semantic/root-based search only.
**Quran MCP** — semantic search for AI applications only.
**Al Quran Cloud** — free, no auth, CORS-friendly, good for prototypes.

## Detailed Comparison

### Quran Foundation API (v4)
**Best for:** Full-featured Quran apps needing text + translations + audio + morphology.
- Most comprehensive single API.
- OAuth2 for user features (bookmarks, last-read sync across quran.com and integrated apps).
- Word-level timestamps for audio synchronization.
- Active development, good documentation at api-docs.quran.foundation.
- **Limitation:** Hafs only. No multi-qira'a text support.

### QuranPedia API
**Best for:** The most comprehensive Quran API — largest tafsir/translation library, 16 riwayat mushafs, i'rab, fatwas, and unique print-book image comparison. **Full endpoint reference and use cases: [quranpedia-api.md](quranpedia-api.md).**
- Free, no authentication required. `api.quranpedia.net/v1`. Rate limit 120/min, 10,000/day per IP.
- **Largest collection of tafsirs and translations** of any Quran API.
- **16 riwayat mushafs** with downloadable SVG + PNG page-pack ZIPs and per-mushaf font files (`/v1/mushafs`).
- **Unique: print-book image comparison** — compare digital text against scanned images of the original printed books for verification.
- **I'rab three ways:** classical i'rab books + computed word morphology (Quranic Arabic Corpus) + dependency-treebank syntax — the only public API with all three.
- 14 per-ayah services: tafsir, e3rab, morphology, syntax, asbab, nasekh, meanings, mutshabeh, similar, topics, fatwa, notes, qiraat (word-level variants with per-rawi audio), translations.
- Reciters with downloadable timing ZIPs; whole-book JSON downloads for translations and books.
- **Unique: delta sync** — `/v1/changes?since=` + bulk `/dumps` (gzipped, SHA-256 manifest, exact API schema) for offline-first apps.
- **Caution:** response shapes vary per endpoint (no envelope, object-vs-array polymorphism); Arabic-first fields; verify CORS before browser-side fetch. Details in [quranpedia-api.md](quranpedia-api.md).
- Also provides an **embeddable widget + oEmbed provider** — see [quranpedia-embed.md](quranpedia-embed.md).

### QUL (Quranic Universal Library by Tarteel)
**Best for:** Feature-rich apps needing the widest content library — translations, tafsirs, fonts, morphology, and audio with word-level timestamps.
- Open-sourced on GitHub. `qul.tarteel.ai`.
- 193 translations, 16 word-by-word translation sets.
- 114 tafsirs (32 concise + 82 detailed).
- 62 segmented audio sets with word-level timestamps.
- 19 approved mushaf layouts + 7 in development.
- 18 Quran-specific fonts (glyph-based and Unicode).
- 77,429 morphological entries (grammar, roots, lemmas, stems).
- 2,512 topics with semantic relations, 5,277 mutashabihat entries, 4,001 similar ayahs.
- Community contribution tools for proofreading and annotation.
- Backed by Tarteel.ai (known for Quran AI/speech recognition).
- **Strength:** Most comprehensive single resource collection for Quran app data. The "toolkit for Muslim developers."

### Al Quran Cloud API
**Best for:** Prototypes, simple apps, or projects needing zero authentication.
- Free, no API key needed.
- Text in multiple scripts + translations + audio.
- Supports CORS (unlike some other APIs).
- **Limitation:** Less feature-rich than Quran Foundation. No word-level data.

### Tanzil.net
**Best for:** Offline apps needing verified, static Quran text.
- Gold standard for verified Quranic text accuracy.
- Download once — no API dependency.
- Multiple formats: BSV, XML, MySQL dumps.
- **Limitation:** Static download only, no API. No audio/tafsir. Requires attribution.

### EveryAyah
**Best for:** Simple ayah-by-ayah audio with predictable URL patterns.
- Dead-simple file structure: `/{reciter_id}/{surah_number}{ayah_number}.mp3`.
- Also provides mushaf page images (PNG/JPG).
- **Limitation:** Audio only (and images). No text API, no translations.

### MP3Quran
**Best for:** Large reciter library across multiple qira'at.
- Extensive collection of reciters.
- Supports multiple qira'at (not just Hafs).
- API for discovering available reciters and their recordings.
- **Limitation:** Audio only. No text or translation data.

### Alfanous
**Best for:** Semantic and root-based Quran search.
- Unique offering: meaning-based search not available elsewhere.
- **Limitation:** Search-only service. Not a general Quran data API.

## Choosing the Right API

**Starting a new app?**
→ Start with **QUL** for data resources (widest content library) + **Quran Foundation API** for live features (OAuth, search). Supplement with **MP3Quran** for more reciters.

**Need multiple mushafs and qira'at?**
→ **QuranPedia API** for mushaf editions with fonts + **quranpedia repos** for ayah mapping data.

**Need i'rab (grammatical analysis)?**
→ **QuranPedia API** is the only API with i'rab data — classical books, computed morphology, and syntax trees. See [quranpedia-api.md](quranpedia-api.md).

**Need to keep an offline app's data fresh?**
→ **QuranPedia** dumps + `/v1/changes` delta sync is the only ready-made mechanism. Seed from dumps, poll changes.

**Embedding Quran content in a website with no backend?**
→ **QuranPedia embed widget / oEmbed** — see [quranpedia-embed.md](quranpedia-embed.md).

**Need the most translations and tafsirs?**
→ **QuranPedia API** has the largest collection of tafsirs and translations of any API, with the unique ability to compare against print-book images. **QUL** also has a very large collection (193 translations, 114 tafsirs).

**Need word-level features?**
→ **Quran Foundation API** and **QUL** both provide word morphology + word-level audio timestamps.

**Need verified offline text?**
→ Use **Tanzil.net** for the text corpus. Bundle it with your app.

**Building an AI/LLM integration?**
→ Use **Quran MCP** (mcp.quran.ai) for semantic retrieval in Claude/ChatGPT apps.

**Need multi-qira'a audio?**
→ **MP3Quran** has the widest reciter/qira'a coverage.

**Combine sources strategically.** No single API covers everything. A typical full-featured app uses 2-3:
- Content library (translations, tafsirs, morphology, fonts) → QUL
- Live API (search, user sync, OAuth) → Quran Foundation
- Multi-qira'a mushafs with i'rab → QuranPedia API
- Additional reciters → MP3Quran / EveryAyah
- Ayah mapping across counting systems → quranpedia/qiraat-ayah-map
- Semantic search → Alfanous or Quran MCP
