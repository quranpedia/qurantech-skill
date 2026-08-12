# Qira'at (Recitation Styles) Support

## Table of Contents
- [Overview](#overview)
- [The 10 Qira'at and 20 Ruwat](#the-10-qiraat-and-20-ruwat)
- [Ayah Counting Systems](#ayah-counting-systems)
- [Ayah Mapping Between Qira'at](#ayah-mapping-between-qiraat)
- [The Hafs Anchor Pattern](#the-hafs-anchor-pattern-cross-mushaf-equivalence)
- [Per-Riwayah Fonts](#per-riwayah-fonts)
- [Best Practices](#best-practices)
- [Data Source](#data-source)

## Overview

The Quran has been transmitted through multiple chains of recitation (qira'at), each with its own rules of pronunciation, pausing, and in some cases, word variants. There are 10 canonical qira'at, each transmitted through 2 primary narrators (ruwat), totaling 20 transmission chains.

**Critical for developers:** Different qira'at follow different ayah counting systems. The same surah:ayah reference can point to different text depending on which system is used.

## The 10 Qira'at and 20 Ruwat

| # | Qari (Reader) | Rawi 1 | Rawi 2 | Counting System |
|---|---------------|--------|--------|-----------------|
| 1 | **Nafi' al-Madani** | Qalun | Warsh | Last Madinan (6,214) |
| 2 | **Ibn Kathir al-Makki** | Al-Buzzi | Qunbul | Makkan (6,219) |
| 3 | **Abu Amr al-Basri** | Al-Duri | Al-Susi | Basran (6,204) |
| 4 | **Ibn Amir al-Shami** | Hisham | Ibn Dhakwan | Damascene (6,226) |
| 5 | **Asim al-Kufi** | Shu'bah | **Hafs** | Kufan (6,236) |
| 6 | **Hamzah al-Kufi** | Khalaf | Khallad | Kufan (6,236) |
| 7 | **Al-Kisa'i** | Abu al-Harith | Al-Duri (al-Kisa'i) | Kufan (6,236) |
| 8 | **Abu Ja'far al-Madani** | Ibn Wardan | Ibn Jammaz | First Madinan (6,214) |
| 9 | **Ya'qub al-Basri** | Ruways | Rawh | Basran (6,204) |
| 10 | **Khalaf al-Bazzar** | Ishaq | Idris | Kufan (6,236) |

**Hafs an Asim** is the most widely used globally (used in Saudi Arabia, most of the Middle East, South/Southeast Asia). **Warsh an Nafi'** is predominant in North Africa and West Africa.

## Ayah Counting Systems

Six counting systems exist, named by scholarly school:

| System | Total Ayahs | Used By |
|--------|-------------|---------|
| **Kufan** | 6,236 | Asim, Hamza, Al-Kisa'i, Khalaf |
| **Last Madinan** | 6,214 | Nafi' |
| **First Madinan** | 6,214 | Abu Ja'far |
| **Makkan** | 6,219 | Ibn Kathir |
| **Basran** | 6,204 | Abu Amr, Ya'qub |
| **Damascene** | 6,226 | Ibn Amir |

The difference is not in the text itself but in where scholars placed ayah boundaries. Some scholars counted the Basmala as ayah 1 of Al-Fatiha; others did not. Some split long passages into two ayahs where others kept them as one.

**Caveat:** Exact totals for the non-Kufan systems vary slightly between scholarly sources (e.g., First Madinan is cited as 6,214 or 6,217 depending on the source). Treat the figures above as orientation only — in code, always derive counts and boundaries from **quranpedia/qiraat-ayah-map**, never from a table like this.

## Ayah Mapping Between Qira'at

Use **quranpedia/qiraat-ayah-map** for cross-qira'a mapping. The dataset provides:

- **Forward mappings:** How each Hafs ayah maps to a target counting system
- **Reverse mappings:** How target system ayahs map back to Hafs
- **Mapping statuses:**
  - `mapped` — 1:1 correspondence
  - `merged` — Two Hafs ayahs combine into one in the target system
  - `split` — One Hafs ayah becomes multiple in the target system
- **Per-rawi metadata** for all 20 narrators
- **Boundary evidence** with scholarly citations

**Data-model insight:** the dataset stores only *disputed* boundaries — the word each contested boundary falls on and which counting madhhabs count it as a ra's ayah. Undisputed Hafs boundaries are implicit. Walking a surah's boundary points in text order reproduces every madhhab's numbering from one compact dataset, with book citations attached. Model your counting data the same way rather than storing six full parallel numberings.

## The Hafs Anchor Pattern (Cross-Mushaf Equivalence)

The production-proven relational design for multi-riwayah apps (battle-tested at 16-mushaf scale):

- **One database**, every ayah row carries `mushaf_id`.
- Every non-Hafs ayah carries an **`equals_ayah_id`** pointing to its corresponding Hafs ayah (its *anchor*). When a mushaf merges two Hafs ayahs into one, its `equals_ayah_id` points to the **first** Hafs ayah of the range.
- **All content** (tafsir, translations, fatwas, topics, qiraat differences) is keyed to **Hafs ayah IDs only** — content is stored once, and any mushaf's ayah reaches it through its anchor ("parent") relationship.

### Why surah + number matching fails

In Hafs, Al-Baqarah opens with two ayahs: (1) "الم" and (2) "ذَٰلِكَ ٱلْكِتَٰبُ...". In Warsh/Qalun these are **one combined ayah**, so their ayah 2 is Hafs's ayah 3. Matching `(surah, number)` across mushafs silently returns the wrong verse.

### The lookup that handles merges

To find the equivalent ayah in a target mushaf, don't require an exact anchor match — take the **greatest anchor ≤ the source's anchor**:

```sql
SELECT * FROM ayahs
WHERE mushaf_id = :target_mushaf
  AND surah_id  = :surah
  AND equals_ayah_id <= :hafs_ayah_id
ORDER BY equals_ayah_id DESC
LIMIT 1
```

When the source ayah was merged into a combined verse (Hafs ayah 2 above has no row with its exact anchor), this still lands on the combined ayah that *contains* it.

### Two different lookups — don't conflate them

- **Displaying text from another mushaf** → the `equals_ayah_id <=` query above.
- **Fetching content (tafsir, differences, fatwas)** → resolve to the Hafs anchor first, then query content by that Hafs ID.

**Word-level differences** are stored the same way: a `(hafs_ayah_id, rawi_id, word, description)` record per variant, always keyed to Hafs. The QuranPedia API exposes this as the `qiraat` service with per-rawi audio per word ([quranpedia-api.md](quranpedia-api.md)).

This single-DB anchor design and the "separate database per riwayah" approach (below) are both valid; prefer the anchor pattern when content (tafsir/translations) must be shared across riwayat, and separate DBs when riwayat are downloaded/deleted independently.

## Per-Riwayah Fonts

Each riwayah needs its own font — Hafs fonts misrender other riwayat's letterforms. KFGQPC-style Uthmani fonts exist for all commonly digitized riwayat: Hafs (+ Nastaleeq variant), Warsh, Qalun, Duri, Susi, Bazzi, Qunbul, Shu'bah, Hisham. The QuranPedia API returns each mushaf's `font_file` URL alongside its text ([quranpedia-api.md](quranpedia-api.md)); King Fahd Complex publishes the originals at `fonts.qurancomplex.gov.sa`. A clean pattern: store the font reference (or `@font-face` CSS) as part of the mushaf record so selecting a mushaf automatically selects its font.

## The Warsh Challenge

Most Quran apps focus on Hafs, but North Africa (Algeria, Morocco, Tunisia, Libya) uses Warsh daily. The Itqan community identified this as a major gap.

**Technical implications of multi-riwayah support:**
1. **Different text** — some words differ (e.g., "مالك" vs "ملك" in Al-Fatiha)
2. **Different fonts needed** — each riwayah may need its own font
3. **Separate databases** per riwayah recommended, with a unified API layer
4. **Testing burden** — each riwayah multiplies QA requirements
5. **Audio metadata** must include riwayah info — different reciters follow different riwayat
6. **Copyright challenges** — some mushaf images require publisher permission

**Available Warsh resources:**
- quranpedia/quran-svg includes Warsh mushaf SVGs
- fawazahmed0/quran-api provides Warsh text (verify accuracy)
- OpenMushaf supports both Hafs and Warsh

**Key community insight:** "We should not teach the machine that all qira'at are the same. Each riwayah must be classified distinctly."

## Mushaf Layouts

The QuranHub API provides multiple mushaf layouts:
- `api.quranhub.com/v1/mushaf-layouts/` — lists available layouts
- IndoPak 13-line layout, Madinah 15-line layout, etc.
- Each layout has different page-to-ayah mappings

## Tahbeer Project

A waqf (endowment) project by Sheikh Saber Abdelhakm (~7 years of work). Professional recordings of all 10 qira'at with their turuq — 20+ reciters for 20 riwayat. Uses al-Tayseer mushafs as reference. iOS + Android apps available.

## Best Practices

- **Never hardcode ayah counts.** Always derive counts from the mushaf/qira'a being used.
- **Store the counting system alongside ayah references** in your database. A reference like "2:5" is ambiguous without knowing which system.
- **Design data models with qira'a awareness.** Use a composite key of (surah, ayah, counting_system) or (surah, ayah, mushaf_id) rather than just (surah, ayah).
- **When supporting multiple qira'at, always show which one is active** in the UI.
- **For cross-qira'a features** (e.g., "show me this ayah in Warsh"), use the mapping data — never assume the same number works across systems.
- **Default to Hafs** if the user hasn't specified a preference, but make it configurable.
- **Test with edge cases:** Surahs where ayah counts differ significantly between systems (e.g., check boundary ayahs in Al-Fatiha, Al-Baqarah, and shorter surahs).
- **Choose an architecture deliberately:** single DB with the [Hafs anchor pattern](#the-hafs-anchor-pattern-cross-mushaf-equivalence) when content is shared across riwayat; separate databases per riwayah with a unified API layer when riwayat are downloaded/deleted independently.

## Data Source

Primary dataset: `github.com/quranpedia/qiraat-ayah-map`

Data format: JSON files in `data/` (source) and `dist/` (generated artifacts).

Structure:
```
data/
  book-boundary-primitives.json   # Disputed boundaries with systems
  book-boundary-evidence.json     # Verification and citations
dist/
  forward/      # Hafs → target system mappings
  reverse/      # Target system → Hafs mappings
  rawi/         # Per-narrator metadata (20 files)
  differences.json      # Word-level compatibility view
  boundary-events.json  # Authoritative deltas
```
