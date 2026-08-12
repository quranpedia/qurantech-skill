# Translations & Tafsir

## Table of Contents
- [Translations](#translations)
- [Tafsir](#tafsir)
- [Copyright & Legal](#copyright--legal)
- [Best Practices](#best-practices)

## Translations

### Sources

| Source | Languages | Model | Notes |
|--------|-----------|-------|-------|
| **Quran Foundation API** | 50+ | API (always current) | Recommended — handles updates automatically |
| **Tanzil.net** | 40+ | Static download | Verified, widely used |
| **QUL (Tarteel)** | Multiple | API | Curated translations |
| **QuranEnc** | Multiple | Download/API | Verified translations |
| **Fawaz Ahmed quran-api** | Multiple | GitHub repo | Open dataset |

### Key Considerations

- **Translations are human-authored and may contain errors.** They are not the Quran — they are scholarly interpretations in other languages.
- **Prefer API-driven sources** for translations so users always get the latest corrected version.
- **Multiple translations per language are common.** For example, English has Sahih International, Pickthall, Yusuf Ali, Dr. Mustafa Khattab (The Clear Quran), and many more. Let users choose.
- **Translation quality varies significantly.** Some are literal, some are interpretive. Provide context about each translation's approach if possible.
- **Some translations are copyrighted.** Verify licensing before including any translation.

### Display Patterns

- **Side-by-side:** Arabic text on the right, translation on the left. Most common layout.
- **Inline:** Translation below each ayah. Good for reading flow.
- **Comparison:** Multiple translations shown together per ayah. Useful for study.
- **Always visually distinguish** translation text from Quranic Arabic text (different font, different color/style, labeled).

## Tafsir

### Sources

| Source | Content | Notes |
|--------|---------|-------|
| **Quran Foundation API** | Multiple tafsirs | API-driven, current |
| **QUL (Tarteel)** | Curated tafsirs | Popular tafsirs included |
| **Spa5k tafsir API** | Multiple tafsirs | Open API. `github.com/spa5k/tafsir_api` — self-hostable |
| **Quran Tafseer API** | Multiple editions | `api.quran-tafseer.com/en/docs/` |

### Common Tafsirs

Popular tafsirs that users expect:
- **Tafsir Ibn Kathir** — classical, widely trusted
- **Tafsir al-Tabari** — comprehensive classical tafsir
- **Tafsir al-Sa'di** — accessible modern Arabic tafsir
- **Al-Jalalayn** — concise, popular for quick reference
- **Fi Zilal al-Quran** (Sayyid Qutb) — literary/thematic approach

### Display Patterns

- **Expandable panel:** Tap an ayah to see its tafsir. Keeps the reading view clean.
- **Dedicated tafsir view:** Full-screen tafsir with the ayah shown at the top.
- **Multiple tafsirs:** Let users switch between tafsir sources.
- **Tafsir text is often very long.** Support scrolling, collapsible sections, or pagination within a tafsir entry.

## Copyright & Legal

- **Always verify copyright** before including any translation or tafsir.
- **Some translations are freely distributable**, others require written publisher permission.
- **Seek explicit approval** from the copyright holder or publisher, especially for tafsir works.
- **Attribute the author/translator** — this is both an ethical and legal requirement.
- **Digitized tafsirs may contain OCR errors.** If using a digitized source, note that quality may vary and plan for corrections.

## AI-Generated Translations & Tafsir

**Never let AI generate tafsir freely.** Use RAG (Retrieval-Augmented Generation) with authoritative tafsir sources.

Real example from the Itqan community: an LLM misinterpreted "Fa waylun lil-musalleen" (Al-Ma'un:4) — giving it an opposite meaning by omitting context from the following ayah. Without asbab al-nuzul and scholarly context, AI interpretations are dangerous.

**Guidelines:**
- Use RAG with verified tafsir databases as the knowledge source
- Always show the source tafsir the AI's answer is based on
- Human review is mandatory before publishing any AI-generated Islamic content
- Label AI-generated content clearly — never attribute it to a scholar
- Pre-process and cache AI results rather than generating on-the-fly

## Best Practices

- **Always label translations as "ترجمة معاني" (Translation of meanings)**, not as "Quran" or "translation of the Quran." This is an important Islamic scholarly distinction.
- **Never display a translation without the Arabic original** visible or easily accessible.
- **Translations should be synced to the correct ayah numbering system** for the active mushaf/qira'a.
- **Default translation** should match the user's device language if available.
- **Keep translations updated** via API rather than bundling static files that go stale.
- **Tafsir and translation are different things.** Don't merge them in the UI — users expect them as separate features.
- **RTL considerations:** Some translations are in RTL languages (Urdu, Farsi) and some in LTR (English, French). Handle mixed directionality correctly.
- **Digitized tafsirs may contain OCR errors.** Plan for corrections and periodic updates.
- **Include a transparent changelog** — any project displaying Quranic text should document its source and version.
