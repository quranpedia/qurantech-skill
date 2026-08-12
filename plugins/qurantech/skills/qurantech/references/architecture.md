# Architecture Patterns for Quran Apps

## Table of Contents
- [Common Architectures](#common-architectures)
- [Offline-First Pattern](#offline-first-pattern)
- [Data Layer Design](#data-layer-design)
- [Multi-Qira'a Architecture](#multi-qiraa-architecture)
- [Anti-Patterns](#anti-patterns)
- [Pre-built Packages](#pre-built-packages)

## Common Architectures

### Reading-focused app
```
UI Layer (Mushaf/Text view, Navigation)
    ↓
Feature Layer (Bookmarks, Search, Audio, Tajweed)
    ↓
Data Layer (Local DB + API sync)
    ↓
Sources (Quran text DB, Audio files, Translation API)
```

### API-first backend
```
Client apps (Web, iOS, Android)
    ↓
API Gateway (REST/GraphQL)
    ↓
Services (Text, Audio, Search, User data)
    ↓
Data stores (Quran DB, Audio CDN, Search index, User DB)
```

### Mushaf viewer
```
Rendering engine (SVG/Image viewer)
    ↓
Interaction layer (Ayah selection, Audio sync, Tajweed toggle)
    ↓
Data provider (Page data, Ayah metadata, Audio timestamps)
    ↓
Asset manager (SVG/Image cache, Font loader)
```

## Offline-First Pattern

Quran apps are expected to work offline. Users recite in mosques, during travel, and in areas without connectivity.

### Strategy

1. **Bundle essential data at install time:**
   - Full Quran text (all ayahs for the default mushaf) — typically ~2-5MB in JSON
   - Mushaf metadata (surah info, juz/hizb mappings, page mappings)
   - At least one translation in the user's language
   - Quranic font files

2. **Download on demand:**
   - Audio files (large — let users choose what to download)
   - Additional translations and tafsirs
   - Additional mushaf SVGs/images for other qira'at

3. **Sync when online:**
   - User data (bookmarks, reading progress, memorization state)
   - Translation/tafsir updates (corrections happen over time)
   - New content (additional reciters, translations)

### Local storage options

| Platform | Storage | Best For |
|----------|---------|----------|
| Web | IndexedDB + Cache API | Text data + audio/font caching |
| iOS | Core Data / SQLite + FileManager | Structured data + audio files |
| Android | Room/SQLite + internal storage | Structured data + audio files |
| Flutter | sqflite / Hive + path_provider | Cross-platform local storage |

### Sync conflict resolution

- **Quran text never conflicts** — it's read-only from source.
- **User data (bookmarks, progress):** Last-write-wins is usually sufficient. More complex: merge bookmarks as a set (union), use latest timestamp for progress.
- **Downloaded content:** Check version/hash before re-downloading.

## Data Layer Design

### Separate Quran data from user data
- **Quran data** (text, metadata, translations) is read-only and shared.
- **User data** (bookmarks, progress, preferences) is mutable and personal.
- Keep them in separate databases/tables for clean backup, sync, and upgrade paths.

### Version your Quran data
- Include a version identifier with bundled Quran data.
- When an update is available (corrected text, new translations), apply it without losing user data.

### Index for search at build/install time
- Pre-build a search index with normalized text (stripped diacritics, normalized alef).
- Don't rebuild it at runtime.

### One data layer, many surfaces
If the same ayah content appears in multiple places (an in-app modal, an embeddable widget, a public API), render all of them from **one shared service layer** — parity is then guaranteed by construction instead of by testing. The same applies to data exports: generate bulk dumps from the same code paths that serve the API, so dump schemas never drift from API schemas.

### Data freshness: seed + delta sync
For content that receives corrections (translations, tafsir, scholarly notes), the robust offline pattern is: **seed from bulk dumps, then poll a changes feed** (`?since=<date>` returning changed record references) and refetch only what changed. This avoids both stale static bundles and full re-downloads. QuranPedia ships this ready-made ([quranpedia-api.md](quranpedia-api.md)); if you run your own backend, expose the same two surfaces.

### AI/LLM discoverability (GEO)
If your Quran content is web-facing, consider serving machine-readable twins: a markdown version of each content URL and an `llms.txt` index. Use plain Unicode Quranic text in these surfaces (never glyph-encoded text — it's garbage outside its font), so LLMs and agents quote the Quran correctly from your site.

## Multi-Qira'a Architecture

Supporting multiple qira'at requires careful separation:

```
Mushaf Selector (User picks qira'a)
    ↓
Mushaf Config (loads correct text, page data, font, ayah count)
    ↓
Feature Layer (all features use active mushaf config)
    ↓
Data Layer (each mushaf has its own text + page data)
```

**Key design decisions:**
- **One database per mushaf** or **one database with mushaf_id partitioning.** Partitioning is simpler; separate DBs are cleaner for download/delete.
- **Ayah references must always include the mushaf context.** See data-models.md.
- **User data (bookmarks) should store mushaf_id.** A bookmark in Hafs page 300 ≠ Warsh page 300.
- **Audio is tied to qira'a.** A Hafs reciter's audio should not play over Warsh text.

## Anti-Patterns

### Hardcoded ayah counts
**Wrong:** `if (ayah > 6236) throw error`
**Right:** Derive max from the active mushaf's metadata.

### Universal surah:ayah references
**Wrong:** Storing `2:5` without mushaf context.
**Right:** Always pair with a mushaf or counting system identifier.

### Building Arabic search from scratch
**Wrong:** Writing custom diacritics-stripping, hamza-normalization, and fuzzy matching.
**Right:** Use a production-ready Arabic search service (Kalimat.dev, Quran Foundation API).

### Static translation bundles
**Wrong:** Shipping translations as static JSON files that never update.
**Right:** Use API-driven sources or implement an update mechanism for static bundles.

### Mixing Quran text with user-generated content in the same table
**Wrong:** Storing Quran text and user notes in the same table/collection.
**Right:** Quran text is sacred, read-only data — isolate it architecturally.

### Ignoring font loading
**Wrong:** Rendering Quranic text immediately with a system font, then swapping.
**Right:** Block rendering of Quranic text until the proper font is loaded (`font-display: block`).

### Single-reciter assumption
**Wrong:** Building audio playback tied to one reciter's file structure.
**Right:** Abstract the audio source so reciters are interchangeable.

## Pre-built Packages

**Always check for existing packages before building from scratch:**

| Package | Platform | What It Provides |
|---------|----------|-----------------|
| **MushafImad** | Swift/iOS | Mushaf display, audio players, core Quran features — use as a starting point for iOS apps |
| **mushaf-imad-android** | Kotlin/Android | Android equivalent — mushaf rendering, bookmarks, audio |
| **mushaf-imad-flutter** | Flutter | Cross-platform: display, bookmarks, search, offline storage |
| **Quran MCP** | AI integration | Semantic search & retrieval for Claude/ChatGPT apps via mcp.quran.ai |

These packages handle the hardest parts of Quran app development (mushaf rendering, audio synchronization, offline storage). Evaluate them before writing custom implementations.

See `data-sources.md` for the full list of available tools and libraries.

## Real-World Architecture Examples

These patterns come from production Quran apps in the Itqan community:

### OpenMushaf (Offline-First Mushaf)
**Stack:** React Native (Expo). 604 mushaf images bundled. Lazy loads ±3 pages. Div overlays for tap-to-ayah tafsir popups. PWA + Service Worker for web.

### OpenTarteel (Audio Streaming)
**Stack:** Next.js + TailwindCSS + GunDB (decentralized). MP3Quran API for audio. Jotai for state. PWA → Android via PWABuilder. Shows that reading and listening apps benefit from separate architectures.

### BAHETH (Hybrid Search Engine)
**Stack:** Python + FastAPI + Elasticsearch + FAISS + Sentence Transformers. Hybrid scoring: `0.6 * cosine + 0.4 * bm25`. See `search.md`.

### SyncQuran (Real-Time Learning Circles)
**Stack:** HTML/JS + WebRTC (PeerJS). Teacher creates session, students connect via peer ID. Real-time page sync, ayah highlighting, and audio playback. TURN server (coturn) needed for NAT traversal.

### PWA as Multi-Platform Strategy
Build a Next.js/web app → convert to PWA → use **PWABuilder** (`pwabuilder.com`) to create Android APK and iOS package → publish to stores. "Updating the website is enough."

## Accessibility

**Severely neglected in most Quran apps.** Key guidelines:

- **Screen reader compatibility is essential.** Tarteel AI and many popular apps are not screen-reader compatible.
- **Blind users need the simplest possible interface** — every extra UI element is an obstacle.
- **Reference apps for accessibility:** Dhikr al-Huda (Flutter, by a blind developer), Al-Bayan.
- **Use semantic HTML** for web: proper `<article>`, `<nav>`, ARIA labels for ayah elements.
- **VoiceOver/TalkBack:** Test with actual screen readers, not just automated tools.
- **Text scaling:** Support Dynamic Type (iOS) and user font scaling (Android/web).

## Licensing Guidance

- **MIT/Apache 2.0:** Best for maximum adoption of Quran libraries.
- **Copyleft (GPL/AGPL):** Careful — using a GPL library may require open-sourcing your entire app.
- **Open-Core model:** Recommended — open-source the core, offer premium features commercially.
- **Content licensing:** Audio/visual Quran content often lacks open licenses. Always verify permissions before using recitation recordings.
