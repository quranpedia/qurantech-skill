# QuranPedia API (api.quranpedia.net)

## Table of Contents
- [Overview](#overview)
- [Base URLs, Auth, Rate Limits](#base-urls-auth-rate-limits)
- [Caching & CORS](#caching--cors)
- [Core Endpoints](#core-endpoints)
- [Per-Ayah Services](#per-ayah-services)
- [Translations Endpoints](#translations-endpoints)
- [Other Endpoints](#other-endpoints)
- [Delta Sync & Bulk Dumps](#delta-sync--bulk-dumps)
- [Response-Format Gotchas](#response-format-gotchas)
- [Licensing Obligations](#licensing-obligations)
- [Use Cases](#use-cases)

## Overview

The QuranPedia API is a free, no-auth REST API backing quranpedia.net — an Arabic-first Quranic encyclopedia. Its unique strengths: **16 riwayat mushafs with downloadable SVG/PNG page packs and fonts**, the **largest tafsir/translation/book library** of any Quran API, **i'rab as data** (both classical books and computed morphology + syntax treebank), fatwas, tadabbur notes, topics, mutashabihat, reciters with timing files, **print-book page images** for text verification, and a **delta-sync feed + bulk dumps** for keeping offline apps fresh.

Documentation: `api.quranpedia.net` (docs page) · For the embeddable widget see [quranpedia-embed.md](quranpedia-embed.md).

## Base URLs, Auth, Rate Limits

| Concern | Value |
|---------|-------|
| Canonical base | `https://api.quranpedia.net/v1` |
| Mirror base | `https://quranpedia.net/api/v1` (identical endpoints) |
| Version | v1 only |
| Auth | None — no API key, no OAuth |
| Rate limits | **120 requests/min and 10,000/day per IP** (429 response includes usage policy and points to `/dumps`) |
| Method | All endpoints are GET, all return JSON |

For bulk needs, don't crawl the API — use the [dumps](#delta-sync--bulk-dumps).

## Caching & CORS

- Responses send `ETag` + `Cache-Control: public, max-age=86400` (24h). Use **conditional requests** (`If-None-Match`) — a 304 doesn't re-transfer the body. `/v1/changes` caches for 1h.
- Server-side content rarely changes; cache aggressively client-side.
- **CORS caution:** the v1 JSON endpoints do not send CORS headers from the application layer (only `/embed` and `/oembed` do). Verify browser `fetch()` works from your origin before relying on it — otherwise call the API server-side or bundle the dumps.

## Core Endpoints

### Mushafs (multi-riwayah text + assets)

| Endpoint | Returns |
|----------|---------|
| `GET /v1/mushafs` | All **16 riwayat mushafs**: name, description, bismillah, `font_file` URL, `images` (SVG page-pack `.zip`), `images_png` (PNG pack `.zip`), nested rawi → qiraa → counting system. Asset URLs are `null` when the file doesn't exist — check per record. |
| `GET /v1/mushafs/{mushaf_id}` | **The full text of one riwayah in a single response**: surahs[] → ayahs[] with `id, number, text, marker, page_number, juz, hizb, ruku, manzil, options[], number_in_hafs[]` |
| `GET /v1/mushafs/{mushaf_id}/{surah_id}/{ayah_number?}` | Ayah(s) in a riwayah. `number_in_hafs` gives the ayah's Hafs anchor number(s). Don't infer merges from the array length — in practice merges surface as **gaps** (and splits as repeats) in the anchor sequence; resolve equivalence with the greatest-anchor-≤ lookup from [qiraat.md](qiraat.md). |

Mushaf id 1 = Hafs (SVG-rendered), 2 = Hafs plain text. Fetch `/v1/mushafs` to discover the rest — never hardcode ids beyond these.

### Ayah services discovery

| Endpoint | Returns |
|----------|---------|
| `GET /v1/ayah/{surah}/{ayah}/options` | Which per-ayah services have content for this verse (Hafs numbering) |
| `GET /v1/ayah/{surah}/{ayah}/book/{book_id}` | `{book, content: [{text, part, page, image?, ayahs}]}` — a specific book's commentary on the ayah; `image` is the **scanned print page** when the book has one (see use cases) |
| `GET /v1/ayah/{surah}/{ayah}/{service}` | Service payload — see next section. Unknown service → 400 with an `available` list. |

## Per-Ayah Services

`GET /v1/ayah/{surah}/{ayah}/{service}` — 14 services:

| Service | Content |
|---------|---------|
| `tafsir` | Tafsir books covering this ayah (fetch content via `/book/{id}`) |
| `e3rab` | Classical i'rab books (كتب الإعراب) |
| `morphology` | **Computed word-by-word morphology** (Quranic Arabic Corpus data — carries a GPL license block, see [Licensing](#licensing-obligations)) |
| `syntax` | **Dependency-treebank syntax** (sentences with grammatical relations; MIT-licensed source) |
| `asbab` | Asbab al-nuzul (occasions of revelation) books |
| `nasekh` | Nasikh/mansukh (abrogation) books |
| `meanings` | Word-by-word gharib meanings: `[{book_info, words: [{text, meaning}]}]` |
| `mutshabeh` | Mutashabihat (similar-verses) books |
| `similar` | Similar-verse scholarly notes with related ayahs and attachments |
| `topics` | Topics linked to the ayah (`related_ayahs` as `"2:255,3:1"` strings) |
| `fatwa` | Fatwas citing the ayah (Arabic question/answer + summaries) |
| `notes` | Tadabbur notes with image/YouTube/website attachments |
| `qiraat` | **Word-level qira'at variants** grouped per word: `[{ayah_word, qiraat: [{qiraa_text, rewayat: {rawi, audio}}]}]` — with per-rawi audio URLs |
| `translations` | Translation books available for this ayah |

## Translations Endpoints

| Endpoint | Returns |
|----------|---------|
| `GET /v1/translations/available-languages/{surah}/{ayah}` | Languages with `code`, `direction`, names |
| `GET /v1/translations/{surah}/{ayah}/{language?}` | Without language: object keyed by language code. With one: array of `{book, "translation-content"}` |
| `GET /v1/translation-books/{language_code?}` | Translation books; `content` is a URL to a **pre-generated whole-book JSON** — download one file instead of 6,236 calls |
| `GET /v1/translation/{book_id}/{surah}/{ayah?}` | One translation for an ayah or whole surah. **Quirk: errors return `{"error": ...}` with HTTP 200** — check the body, not just the status. |

## Other Endpoints

| Endpoint | Returns |
|----------|---------|
| `GET /v1/surah/information/{surah}` | Structured surah info (revelation, virtues, purposes, names…) |
| `GET /v1/surah/books/{surah}`, `/v1/surah/tafsirs/{surah}`, `/v1/surah/fatwas/{surah}` | Surah-level content lists |
| `GET /v1/topics` | Full topic hierarchy (`parent_id`, ayahs as `"s:a"` comma strings) |
| `GET /v1/reciters` | Reciters grouped per person; each recitation has `rawi`, type, `surahs_list`, and `timing_url` → downloadable **timing ZIP** for audio-text sync |
| `GET /v1/categories/{type?}` | Category tree; `type` ∈ `books|fatwas|notes` adds counts |
| `GET /v1/book/{book_id}` | Full book metadata, people roles (author/muhaqqiq/translator…), attachments, and `contents_url` → whole-book JSON when available |
| `GET /v1/related-books/{book_id}` | Related works grouped by relation type |
| `GET /v1/fatwa/{id}`, `GET /v1/note/{id}` | Single records (**note returns a one-element array**, not an object) |
| `GET /v1/search/{query}/{type?}` | Search across `notes|fatwas|topics|books` (omit type = all four). Query is a **path segment — URL-encode it**. Fixed 15/page. |

Legacy unversioned `/api/*` endpoints also exist (per-rawi ayah counts at `/api/qiraat/ayah-count/{rawi}`, per-surah recitation timings JSON, brotli-compressed mushaf page SVGs at `/api/page/{path}/{page}`). They're publicly reachable but undocumented — prefer v1 where an equivalent exists.

## Delta Sync & Bulk Dumps

The standout feature for **offline-first apps** (see [architecture.md](architecture.md)):

- **`GET /v1/changes?since=YYYY-MM-DD`** — everything that changed since a date (must be within the last year). Returns per-type change sets (`ayahs`, `books`, `fatwas`, `notes`, `topics`, `ayah_book_contents`), capped at 1,000 rows each with a `truncated` flag, and **each row carries a ready `refetch` path**. Poll it periodically → refetch only what changed.
- **`/dumps` + `/dumps/manifest.json`** — versioned, gzipped JSON dumps **in the exact v1 API schema** (each with `license`, `schema`, `data`), SHA-256 checksums in the manifest, plus `complete-all.zip` / `core-all.zip` bundles. Seed your local DB from dumps, then stay current via `/v1/changes`.

This dump-seed + delta-sync pattern is the recommended way to build an offline app on QuranPedia data — it avoids rate limits entirely and gives you checksummed, versioned inputs for [text-integrity testing](testing-qa.md).

## Response-Format Gotchas

Shapes are hand-built per endpoint — clients must branch:

1. **No envelope** — bare arrays/objects, no `data`/`meta` wrapper.
2. **Shape polymorphism** — `/v1/mushafs/{m}/{s}/{a?}` returns an *object* for a single match, an *array* otherwise. `/v1/note/{id}` always returns a one-element array. `/v1/translations/{s}/{a}` returns an object keyed by language without a code, an array with one.
3. **Ayah lists as strings** — topics/notes reference ayahs as `"2:255,3:1"` comma strings, not arrays. Parse them.
4. **HTML in payloads** — book/tafsir `content.text` is raw HTML; search results contain Elasticsearch highlight markup. Sanitize before display.
5. **Arabic-first** — v1 has no locale parameter; translatable fields resolve in Arabic (`ar_title`, `ar_answer`, …). Translations endpoints are the multilingual surface.
6. **One endpoint errors with HTTP 200** — `/v1/translation/{book}/{s}/{a?}`. Everywhere else: 404/400 with `{"error": ...}`.

## Licensing Obligations

- The `morphology` service embeds a **source block: Quranic Arabic Corpus, GNU GPL, attribution to corpus.quran.com required**. The `syntax` service's treebank source is MIT. **Preserve and display these attributions** in your app — don't strip the `source` object.
- Book/tafsir/translation content carries its own copyright; the dumps include per-file `license` fields. Verify before redistribution ([translations-tafsir.md](translations-tafsir.md)).

## Use Cases

| You're building… | Use |
|------------------|-----|
| **Multi-riwayah mushaf app** | `/v1/mushafs` for the catalog + fonts + SVG/PNG page packs; `/v1/mushafs/{id}` for full text; `number_in_hafs` for cross-riwayah mapping |
| **Tafsir reader** | `/options` → `tafsir` service → `/book/{id}` content; `image` field gives the scanned print page for **"مطابق للمطبوع" verification UIs** |
| **I'rab / Arabic-learning app** | `e3rab` (classical books) + `morphology` (word segments) + `syntax` (dependency trees) — the only public API with all three. See [irab.md](irab.md). |
| **Translation app** | `/v1/translation-books` whole-book JSONs — one download per translation |
| **Audio player with highlighting** | `/v1/reciters` → `timing_url` ZIPs for ayah-level sync ([audio.md](audio.md)) |
| **Qira'at education tool** | `qiraat` service: word-level variants with per-rawi audio ([qiraat.md](qiraat.md)) |
| **Hifz app** | `mutshabeh`/`similar` services for confusion warnings ([memorization.md](memorization.md)) |
| **Offline-first anything** | Seed from `/dumps`, stay fresh via `/v1/changes` |
| **Zero-backend website widget** | Skip the API — use the embed service ([quranpedia-embed.md](quranpedia-embed.md)) |
