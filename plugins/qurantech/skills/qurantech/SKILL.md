---
name: qurantech
license: MIT
description: "Expert guide for building Quran apps and adding Quran features to existing apps. Covers mushaf display, Arabic text rendering and fonts, audio recitation (ayah, continuous, word-level), all 10 qira'at with ayah-count mapping, tajweed, search (full-text, root, semantic), translations, tafsir, i'rab, scholarly content types (asbab, fatwas, topics), memorization/hifz tools, verse recognition from audio (offline-tarteel ONNX), Quran embed widgets (oEmbed), data modeling, offline-first architecture, testing, and adab rules for handling sacred text. Use for ANY task touching Quranic text, recitation, or Quran data — building an app or tool, adding a feature ('add Warsh support', 'identify verse from audio', 'embed an ayah in my site'), choosing APIs (quran.com, QUL, Tanzil, quranpedia, mp3quran), picking Quran fonts, or designing schemas. Triggers on: quran, mushaf, ayah, surah, qira'at, hafs, warsh, tajweed, recitation, hifz, memorization, tafsir, tarteel, verse recognition, quran embed/widget, islamic app."
---

# QuranTech

Build Quran apps and tools with best practices, verified data sources, and respect for the sacred text.

## Interactive Workflow

**Clarify requirements before building or modifying anything** — these decisions change the architecture, and reworking a Quran app after picking the wrong qira'a or data source is expensive. Use AskUserQuestion where available; otherwise ask in plain conversation. Skip questions the user has already answered, and skip the interview entirely when the request is fully specified.

### For new projects

Ask these questions in sequence (1-4 per round):

1. **What are you building?** (Mushaf reader, audio app, learning tool, API, search engine, etc.)
2. **Target platform?** (Web, iOS, Android, Flutter, backend API, etc.)
3. **Which qira'a?** (Hafs only, Warsh, multiple, all 10 — see [references/qiraat.md](references/qiraat.md))
4. **Core features needed?** (Audio, tajweed, search, translations, tafsir, memorization, mushaf pages)
5. **Offline support?** (Bundled data, download-on-demand, online-only)
6. **Data sources?** Guide them through API options (see [references/api-comparison.md](references/api-comparison.md))

After gathering answers, recommend an architecture and data sources, then proceed.

### For existing projects

Ask:
1. **What feature are you adding/changing?**
2. **What's the current tech stack?**
3. **What qira'a does the app currently support?**
4. **What data sources are currently in use?**

Then consult the relevant reference file and guide the implementation.

## Adab Rules (Mandatory)

**Read and enforce [references/adab.md](references/adab.md) at all times.** These rules are non-negotiable:

- Never truncate an ayah mid-text
- Never strip diacritical marks from Quranic text
- Use verified text sources only — never manually type Quranic text
- Use Quranic fonts, not generic Arabic fonts
- Never log Quranic text in error/debug logs — use references (surah:ayah)
- Never use Quranic text as test data or placeholder text
- Label translations as translations, not as the Quran itself
- Never auto-play audio without user intent

## Reference Files

Load the relevant reference file based on the task:

| Task | Reference |
|------|-----------|
| Choosing data sources or APIs | [references/data-sources.md](references/data-sources.md) |
| Comparing Quran APIs | [references/api-comparison.md](references/api-comparison.md) |
| Using the QuranPedia API (endpoints, dumps, delta sync) | [references/quranpedia-api.md](references/quranpedia-api.md) |
| Embedding Quran content in websites (widget, oEmbed) | [references/quranpedia-embed.md](references/quranpedia-embed.md) |
| Scholarly content beyond tafsir (asbab, fatwas, topics, gharib…) | [references/content-types.md](references/content-types.md) |
| Supporting multiple qira'at | [references/qiraat.md](references/qiraat.md) |
| Rendering Arabic text, fonts, RTL | [references/text-rendering.md](references/text-rendering.md) |
| Mushaf page display (SVG/image/text) | [references/mushaf-display.md](references/mushaf-display.md) |
| Tajweed color coding | [references/tajweed.md](references/tajweed.md) |
| Audio recitation features | [references/audio.md](references/audio.md) |
| Verse recognition (audio-to-ayah, offline-tarteel) | [references/verse-recognition.md](references/verse-recognition.md) |
| Quran search implementation | [references/search.md](references/search.md) |
| Translations and tafsir | [references/translations-tafsir.md](references/translations-tafsir.md) |
| Memorization / hifz features | [references/memorization.md](references/memorization.md) |
| Data modeling and schemas | [references/data-models.md](references/data-models.md) |
| App architecture and offline | [references/architecture.md](references/architecture.md) |
| Grammatical analysis (i'rab) | [references/irab.md](references/irab.md) |
| Testing, QA, text integrity | [references/testing-qa.md](references/testing-qa.md) |
| Etiquette rules (adab) | [references/adab.md](references/adab.md) |

## Key Principles

1. **Use existing packages first.** Check MushafImad (iOS), mushaf-imad-android (Android), mushaf-imad-flutter (Flutter) before building from scratch. See [references/architecture.md](references/architecture.md).
2. **Use production-ready search.** Don't build Arabic search from scratch — use Kalimat.dev, Quran Foundation API, or Alfanous. See [references/search.md](references/search.md).
3. **Never hardcode ayah counts.** Different qira'at have different counts (6,204–6,236). Always derive from mushaf metadata.
4. **Prefer API-driven data** for translations and tafsir — they receive corrections over time. Static bundles go stale.
5. **Design for offline.** Quran apps must work without internet. Bundle core content, download extras on demand. See [references/architecture.md](references/architecture.md).
6. **Respect copyright.** Always verify licensing for translations, tafsir, and audio recordings before inclusion.
7. **Ayah references need context.** `2:5` is ambiguous without a counting system. Always include mushaf/qira'a context in data models.
8. **AI safety is critical.** Never let AI generate tafsir freely — use RAG with authoritative sources. Human review is mandatory for all AI-generated Islamic content. See [references/adab.md](references/adab.md).
9. **Accessibility matters.** Screen reader compatibility, simple UI for blind users, Dynamic Type support. Most Quran apps fail here. See [references/architecture.md](references/architecture.md).
10. **Don't reinvent the wheel.** The Itqan community was founded to solve this exact problem — scattered efforts rebuilding the same things. Check existing open-source projects first. See [references/data-sources.md](references/data-sources.md).
11. **Automate text integrity checks.** A dropped diacritic or stripped waqf mark silently corrupts sacred text. Verify stored text byte-exact against the verified source in CI. See [references/testing-qa.md](references/testing-qa.md).

## Quick Start Recommendations

**Simplest Quran app:** Quran Foundation API (text + translations + audio) + Amiri Quran font + any web/mobile framework.

**Full-featured mushaf reader:** quranpedia/quran-svg (mushaf pages) + Quran Foundation API (text + audio + translations) + KFGQPC font + offline DB.

**Multi-qira'a app:** quranpedia/qiraat-ayah-map (ayah mapping) + quranpedia/quran-svg (5 mushafs) + MP3Quran (multi-qira'a reciters) + per-qira'a font selection.

**Memorization (hifz) app:** Quran Foundation API or QUL (text + ayah audio) + QUL mutashabihat data + spaced-repetition review queue + offline-tarteel for recitation checking. See [references/memorization.md](references/memorization.md).

**Quran content in an existing website (zero backend):** QuranPedia embed widget or oEmbed. See [references/quranpedia-embed.md](references/quranpedia-embed.md).

**AI-powered Quran tool:** Quran MCP (mcp.quran.ai) for semantic search + Quran Foundation API for data.
