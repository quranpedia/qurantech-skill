# Quran Data Models & Taxonomy

## Table of Contents
- [Core Entities](#core-entities)
- [Structural Divisions](#structural-divisions)
- [Text Entities](#text-entities)
- [Audio Entities](#audio-entities)
- [User Entities](#user-entities)
- [Relationships](#relationships)
- [Schema Design Principles](#schema-design-principles)

## Core Entities

### Mushaf
The top-level entity. A mushaf is a specific printed/digital edition of the Quran tied to a particular qira'a and rawi.

| Field | Type | Description |
|-------|------|-------------|
| id | string | Unique mushaf identifier |
| name | string | e.g., "Mushaf al-Madinah" |
| qiraa | string | e.g., "Asim" |
| rawi | string | e.g., "Hafs" |
| counting_system | string | e.g., "kufan" |
| total_ayahs | integer | e.g., 6236 |
| total_pages | integer | e.g., 604 |
| language | string | Primary script language |

### Surah (Chapter)
| Field | Type | Description |
|-------|------|-------------|
| number | integer | 1–114 |
| name_arabic | string | e.g., "الفاتحة" |
| name_transliterated | string | e.g., "Al-Fatihah" |
| name_translation | string | e.g., "The Opening" |
| revelation_type | enum | makki / madani |
| ayah_count | integer | Varies by counting system |
| revelation_order | integer | Order of revelation |
| bismillah | boolean | Has basmala (false for Surah 9) |
| start_page | integer | Page in the mushaf |

### Ayah (Verse)
| Field | Type | Description |
|-------|------|-------------|
| surah_number | integer | Parent surah |
| ayah_number | integer | Within the surah (qira'a-specific) |
| text_uthmani | string | Uthmani script text |
| text_simple | string | Simplified Arabic (for search) |
| page | integer | Mushaf page number |
| juz | integer | Juz number (1–30) |
| hizb | integer | Hizb number (1–60) |
| rub_al_hizb | integer | Quarter-hizb (1–240) |
| manzil | integer | Manzil number (1–7) |
| ruku | integer | Ruku number |
| sajda | enum | none / recommended / obligatory |
| equals_hafs_ayah | reference | **Hafs anchor** — for non-Hafs mushafs, the corresponding Hafs ayah (first of the range when merged). Key all shared content to Hafs IDs and resolve through this. See [qiraat.md](qiraat.md) |

## Structural Divisions

| Division | Count | Description | Use Case |
|----------|-------|-------------|----------|
| **Juz** (جزء) | 30 | Equal divisions for monthly reading | "Read one juz per day in Ramadan" |
| **Hizb** (حزب) | 60 | Half-juz divisions | Prayer portions |
| **Rub' al-Hizb** (ربع الحزب) | 240 | Quarter-hizb markers | Finer reading plans |
| **Manzil** (منزل) | 7 | Weekly reading divisions | "Complete Quran in one week" |
| **Ruku** (ركوع) | ~558 | Thematic passage units | Used in Hanafi prayer tradition |
| **Page** (صفحة) | 604 | Mushaf pages | Mushaf display, page-by-page reading |

**All divisions map to specific ayah ranges.** Store the start/end ayah for each division.

## Text Entities

### Word (Kalimah)
| Field | Type | Description |
|-------|------|-------------|
| ayah_id | reference | Parent ayah |
| position | integer | Word position within ayah (1-based) |
| text_uthmani | string | Uthmani rendering |
| text_simple | string | Simplified text |
| transliteration | string | Latin transliteration |
| translation | string | Word meaning |
| root | string | Arabic root (e.g., "ك ت ب") |
| lemma | string | Dictionary form |
| morphology | string | Grammatical analysis |

### Translation
| Field | Type | Description |
|-------|------|-------------|
| ayah_id | reference | Which ayah |
| language | string | ISO language code |
| author | string | Translator name |
| text | string | Translation text |
| source | string | Data source/API |

### Tafsir Entry
| Field | Type | Description |
|-------|------|-------------|
| ayah_range | range | Start–end ayah (tafsir often covers ranges) |
| tafsir_name | string | Which tafsir work |
| author | string | Scholar name |
| text | string | Tafsir content |

## Audio Entities

### Reciter, Recitation, Rawi — three distinct entities

Model these separately (a common conflation bug):
- **Rawi** = the transmission chain (defines the *text*: Hafs, Warsh…)
- **Reciter** = the person/voice
- **Recitation** = one recording set of a reciter in a specific rawi

One reciter can have multiple recitations (different riwayat, murattal vs mujawwad, different studios/bitrates). Audio, timings, and availability attach to the **recitation**, not the reciter.

### Reciter
| Field | Type | Description |
|-------|------|-------------|
| id | string | Unique ID |
| name_arabic | string | Reciter name in Arabic |
| name_english | string | Reciter name transliterated |

### Recitation
| Field | Type | Description |
|-------|------|-------------|
| reciter_id | reference | The person |
| rawi | string | Which riwayah this recording follows |
| classification | enum | by_surah / by_verse |
| style | string | e.g., "murattal", "mujawwad" |
| servers | map | Base URLs per bitrate (128/64/32 kbps) |
| available_surahs | array | Not all recordings are complete |
| has_timings | boolean | Whether ayah/word timing data exists |

### Audio Segment
| Field | Type | Description |
|-------|------|-------------|
| recitation_id | reference | Which recitation (not just reciter) |
| surah | integer | Surah number |
| ayah | integer | Ayah number (null for full-surah files) |
| url | string | Audio file URL |
| duration_ms | integer | Duration in milliseconds |
| word_timestamps | array | [{word_position, start_ms, end_ms}] |

## User Entities

### Bookmark
| Field | Type | Description |
|-------|------|-------------|
| surah | integer | Bookmarked surah |
| ayah | integer | Bookmarked ayah |
| mushaf_id | string | Which mushaf context |
| label | string | User-defined label |
| created_at | datetime | When created |

### Reading Progress
| Field | Type | Description |
|-------|------|-------------|
| mushaf_id | string | Which mushaf |
| last_page | integer | Last viewed page |
| last_surah | integer | Last surah |
| last_ayah | integer | Last ayah |
| updated_at | datetime | When last read |

### Memorization Progress
| Field | Type | Description |
|-------|------|-------------|
| surah | integer | Which surah |
| ayah_from | integer | Range start |
| ayah_to | integer | Range end |
| status | enum | not_started / in_progress / memorized / review |
| last_reviewed | datetime | Last review date |
| confidence | float | Self-assessed confidence (0–1) |

## Relationships

```
Mushaf ─── has many ──→ Surah
Surah  ─── has many ──→ Ayah
Ayah   ─── has many ──→ Word
Ayah   ─── has many ──→ Translation
Ayah   ─── belongs to ──→ Juz, Hizb, Rub, Manzil, Ruku, Page
Reciter ─── has many ──→ Audio Segment
Audio Segment ─── linked to ──→ Ayah
```

## Schema Design Principles

- **Always include mushaf/qira'a context** in ayah references. `(surah:2, ayah:5)` is ambiguous. `(mushaf: hafs_madinah, surah:2, ayah:5)` is unambiguous.
- **Never hardcode total ayah counts.** Derive from the active mushaf.
- **Use the counting system as part of composite keys** when storing cross-qira'a data.
- **Normalize sparingly for Quranic text.** Quranic text should be stored exactly as received from the source — never transform or normalize the Uthmani text itself.
- **Store simplified text separately** for search purposes (stripped diacritics, normalized alef/hamza).
- **Translations and tafsir reference ayah ranges**, not just single ayahs — a tafsir entry often covers multiple consecutive ayahs.
- **Ranges are the norm for scholarly attachments generally.** In production datasets, most topic-to-ayah links are ranges, not single verses. Use `(ayah_from, ayah_to)` on the attachment pivot rather than one-row-per-ayah. See [content-types.md](content-types.md).
- **People attach with roles.** Author, muhaqqiq (editor), mufti, reviewer, translator, narrator — one polymorphic people-attachment with a role type beats separate columns per role.
- **Audio segments should reference the recitation** — the same ayah has different audio per recitation (reciter × rawi × style).
- **User data (bookmarks, progress) must reference the mushaf** — a bookmark at "page 300" means different things in different mushafs.
