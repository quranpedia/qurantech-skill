# Quran Search

## Table of Contents
- [Search Types](#search-types)
- [Full-Text Search](#full-text-search)
- [Root-Based Search](#root-based-search)
- [Semantic Search](#semantic-search)
- [Transliteration Search](#transliteration-search)
- [Pattern Search](#pattern-search)
- [Hybrid Search (BAHETH Approach)](#hybrid-search-baheth-approach)
- [Building It Yourself: A Production Elasticsearch Recipe](#building-it-yourself-a-production-elasticsearch-recipe)
- [Arabic NLP Tools](#arabic-nlp-tools)
- [Services & APIs](#services--apis)
- [Best Practices](#best-practices)

## Search Types

| Type | Description | Complexity | User Need |
|------|-------------|------------|-----------|
| **Full-text** | Find exact or fuzzy word/phrase matches | Medium | "Find all ayahs containing كتاب" |
| **Root-based** | Find words sharing an Arabic root | High | "Find all forms of ك-ت-ب (writing)" |
| **Semantic** | Find ayahs by meaning, not exact words | High | "Find ayahs about patience" |
| **Transliteration** | Search Arabic text using Latin input | Medium | "Search for 'bismillah'" |
| **Reference** | Jump to a specific surah:ayah | Low | "Go to 2:255" |

## Full-Text Search

Arabic full-text search is harder than Latin-script search due to:

- **Diacritics (tashkeel):** Users type without diacritics but Uthmani text has them. Search must normalize: strip diacritics during indexing and querying.
- **Hamza variants:** أ إ آ ا should all match. Normalize alef forms.
- **Ta marbuta / Ha:** ة and ه should match in search context.
- **Waw/Ya variants:** Consider matching ؤ with و, and ئ with ي.
- **Shadda handling:** Don't require users to type shaddah.

**Recommendation:** Use a production-ready Arabic search service (Kalimat.dev, Quran Foundation API) rather than building Arabic text normalization from scratch.

## Root-Based Search

Arabic words derive from 3-letter (sometimes 4-letter) roots. Root-based search finds all words in the Quran sharing a root, regardless of form.

Example: Root ع-ل-م (knowledge) → عَلِمَ, يَعْلَمُ, عِلْمٌ, عَالِمٌ, عَلِيمٌ, etc.

**Data sources:**
- Quran Foundation API provides morphological data with roots.
- Corpus.quran.com (Quranic Arabic Corpus) has full morphological analysis.

**Approach:** Use pre-computed root mappings rather than implementing an Arabic morphological analyzer.

## Semantic Search

Finding ayahs by topic or meaning:

- **Alfanous.org** provides semantic search capabilities.
- **Quran MCP** (mcp.quran.ai) offers semantic search for AI applications.
- **Topic-based indices:** Pre-built topic mappings (e.g., "ayahs about prayer," "ayahs about patience") can supplement text search.

## Transliteration Search

Allow users who can't type Arabic to search using Latin characters:

- Map common transliteration schemes (e.g., "bismillah" → "بسم الله").
- Consider multiple romanization standards (ISO 233, simplified, informal).
- This is best used as a secondary search mode, not the primary one.

## Pattern Search

Search by diacritical pattern (fatha + damma + sukun + fatha + tanwin) regardless of actual letters. Returns words sharing the same vowel pattern (e.g., "مَثُوبَةً" and "حَمُولَةً"). Available on QuranMorphology.com.

## Hybrid Search (BAHETH Approach)

The BAHETH project from the Itqan community demonstrates a production hybrid search:

**Formula:** `hybrid_score = 0.6 * cosine_score + 0.4 * bm25_score`

**Stack:**
- **BM25** (`rank-bm25` library) — lexical matching based on word frequency/rarity
- **FAISS** (Facebook AI Similarity Search) — vector similarity search
- **intfloat/multilingual-e5-base** — converts Arabic text to 768-dim vectors
- **Elasticsearch** — stores text with `dense_vector` fields alongside keyword fields

**Improvements to consider:**
- Arabic stop words filtering for BM25 quality
- Dynamic weight adjustment between semantic and lexical scores based on query length
- Show why each result is relevant, not just the result
- Integrate tafsirs/translations to expand search across languages

**Repo:** `github.com/engsaleh/quran_semantic_search`

## Building It Yourself: A Production Elasticsearch Recipe

If a hosted service won't do (offline requirements, custom corpus), this architecture is battle-tested in production Quran search — copy it rather than inventing your own. The concepts (multi-variant indexing, a boost ladder, proclitic handling, fallback tiers, position re-ranking) port to any engine — OpenSearch, Meilisearch, Typesense, SQLite FTS, or an in-app index; the terminology below is Elasticsearch's because that's where it was proven:

**Index design (per ayah):**
- Multi-field mapping: raw text, Arabic-analyzed, Arabic-*exact* (normalized but unstemmed), plus a diacritics-stripped `clean_text` in the same three variants.
- Custom analyzer chain: standard tokenizer + `lowercase`, `decimal_digit`, `arabic_normalization`, `arabic_stop`, `arabic_stemmer`; a second analyzer without stemming for exact matching; an `edge_ngram(3,20)` filter for as-you-type autocomplete.
- **Index roots and lemmas per ayah** (from morphological data — see [irab.md](irab.md)), so searching a root surfaces all its derivations without query-time analysis.

**Query design — a boost ladder, not one query.** Use `dis_max` (take the best clause, `tie_breaker` ~0.1) over, in descending boost: exact phrase on raw text → phrase on normalized-unstemmed → phrase on stemmed → phrase-prefix for mid-typing.

**The stopword trap:** Arabic stopwords («من», «لا», single letters while typing) get *dropped by the analyzer*, silently breaking phrase queries. Detect when analysis would drop query terms and switch to a whitespace-analyzed field with `span_near` (in order, slop 0). Let the first term match **و/ف proclitic variants** via `span_or` — a search for "من يعمل" must also find "وَمَن يَعمَل".

**Fallback ladder:** strict ordered phrase → loose stemmed match → normalized-query retry (e.g., inserting a space after «لا»). Only show "no results" after all three.

**Post-ranking:** re-rank the top hits by match *position* (starts-with > contains > truncated match) — Elasticsearch relevance alone ranks Quran results unintuitively. Then present in mushaf order per the adab rules.

**Batching:** when one screen needs several queries (result counts per category + top hits per category + autocomplete), send them as a single `_msearch` round trip.

## Arabic NLP Tools

| Tool | Purpose |
|------|---------|
| **CAMeL Tools** (`github.com/CAMeL-Lab/camel_tools`) | Morphological analysis, dialect ID, sentiment |
| **AraBERT** (`github.com/aub-mind/arabert`) | Arabic BERT for text classification |
| **Arabic-Phonetiser** (`github.com/nawarhalabi/Arabic-Phonetiser`) | 68-phoneme analyzer for tajweed sounds |

## Services & APIs

| Service | Type | Notes |
|---------|------|-------|
| **Kalimat.dev** | Full-text Arabic | Production-ready, handles normalization |
| **Alfanous.org** | Semantic/root | Meaning-based search |
| **Quran Foundation API** | Full-text + morphology | API-integrated, includes root data |
| **Quran MCP** | Semantic (AI) | For AI app integration via `mcp.quran.ai` |
| **tafsir.app** | Root-based | Root search for Quran |
| **QuranMorphology.com** | Morphological | Pattern and grammatical form search |

**Use production-ready search services.** Building a correct Arabic search engine from scratch is a large, error-prone project. The services above handle normalization, diacritics, and edge cases.

## Best Practices

- **Always show complete ayahs in results**, not snippets. Highlight the matched portion within the full ayah.
- **Sort results by mushaf order** (surah:ayah) by default, not by "relevance score."
- **Show the surah name and ayah number** prominently in results.
- **Support both with-tashkeel and without-tashkeel input.**
- **Handle zero results gracefully** — suggest alternative searches or offer root-based search as a fallback.
- **Reference search (surah:ayah lookup)** should be instant — detect patterns like "2:255", "البقرة 255", or "Al-Baqarah 255" and navigate directly.
- **Search history** is a valuable feature — users often return to ayahs they searched before.
- **Performance:** For offline search, pre-build a normalized index at install/download time.
