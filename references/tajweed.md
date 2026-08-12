# Tajweed Color Coding

## Table of Contents
- [Overview](#overview)
- [Tajweed Rules & Colors](#tajweed-rules--colors)
- [Waqf (Pause) Marks](#waqf-pause-marks)
- [Implementation Approaches](#implementation-approaches)
- [Data Sources](#data-sources)
- [Best Practices](#best-practices)

## Overview

Tajweed is the set of rules governing pronunciation of Quranic recitation. Color-coded tajweed overlays specific letters/marks with colors to help learners identify where rules apply. This is a common feature in Quran apps.

## Tajweed Rules & Colors

Standard color conventions (based on common tajweed mushaf editions):

| Rule | Arabic | Color | Applies To |
|------|--------|-------|-----------|
| **Ghunnah** | غنّة | Green | Noon/meem with shaddah (nasalization) |
| **Ikhfa** | إخفاء | Orange/light green | Noon sakinah/tanween before specific letters |
| **Idgham with ghunnah** | إدغام بغنّة | Green | Noon sakinah/tanween before ي ن م و |
| **Idgham without ghunnah** | إدغام بلا غنّة | Green (darker) | Noon sakinah/tanween before ل ر |
| **Iqlab** | إقلاب | Pink | Noon sakinah/tanween before ب |
| **Qalqalah** | قلقلة | Blue | Letters ق ط ب ج د when sakin |
| **Madd (elongation)** | مد | Red/dark red | Alef, waw, ya when elongated |
| **Idhhar** | إظهار | No color (normal) | Noon sakinah/tanween before throat letters |
| **Lam shamsiyyah** | لام شمسية | Gray/muted | Lam in ال before sun letters |
| **Silent letters** | حروف لا تُنطق | Gray | Written but not pronounced |

**Note:** Color schemes vary between publishers. The above is the most common convention but not universal. Allow customization if possible.

## Waqf (Pause) Marks

Waqf marks tell the reciter where stopping is required, preferred, permitted, or forbidden. They are part of the mushaf text and must never be stripped or rendered incorrectly.

| Mark | Name | Meaning |
|------|------|---------|
| مـ | Waqf lazim | Stop is required — continuing may distort meaning |
| قلى | Al-waqf awla | Stopping is preferred |
| صلى | Al-wasl awla | Continuing is preferred |
| ج | Waqf ja'iz | Stop or continue — both acceptable |
| لا | La waqfa fih | Do not stop here |
| ∴ ∴ | Mu'anaqah (twin dots) | Stop at one of the two marks, not both |
| س | Saktah | Brief pause without taking a breath |

**Implementation notes:**
- Waqf marks live in the Unicode Quranic annotation range (`U+06D6–U+06ED`). Verified sources (Tanzil, KFGQPC, QUL) include them in the text — never filter them out during processing or normalization.
- They render as small superscript glyphs; a proper Quranic font is required (generic Arabic fonts often render them as boxes or misplace them).
- Waqf placement differs between riwayat — Hafs and Warsh mushafs mark different stop points. Waqf data is part of the mushaf edition, not universal.
- In tajweed-learning apps, waqf marks deserve their own legend entry alongside the color rules.

## Implementation Approaches

### Pre-annotated text
Use Quranic text that already has tajweed annotations embedded:
- Quran Foundation API provides tajweed-annotated text.
- Each character/group is tagged with its tajweed rule.
- Render by mapping rule → color.

### Rule-based detection
Apply tajweed rules programmatically to plain Uthmani text:
- More flexible but significantly more complex.
- Must handle contextual rules (a letter's tajweed depends on what follows it).
- Error-prone — requires deep tajweed knowledge to implement correctly.
- **Not recommended** unless building a tajweed education tool.

### SVG/image overlay
For mushaf-page displays, overlay colored regions on the SVG/image:
- Use coordinate data to position color highlights.
- Works with quranpedia/quran-svg polygon data.

### Tajweed color font
Use a font whose glyphs carry the tajweed colors (e.g., the QPC V4 tajweed font), treating the tajweed edition as its own mushaf in your data model. Pixel-faithful to printed tajweed mushafs with zero annotation logic — but colors aren't customizable and the text needs the same searchability caveats as glyph-based rendering ([text-rendering.md](text-rendering.md)).

### Implementation notes (any approach)
- **Rules overlap — define a priority table.** When two rules apply to the same letters, the more specific wins (madd lazim over madd tabii). Decide this in data, not scattered conditionals.
- **Highlight precisely.** Color only the letters the rule applies to (e.g., just the madd letter), not the whole word.
- **Pattern matching against Uthmani text requires normalization.** Uthmani text is littered with optional marks (small waw/yaa/meem, tatweel, small high seen, three-dot marks…) that break naive string matching. Maintain an explicit, documented list of marks your matcher strips — and never apply that stripping to the *displayed* text ([adab.md](adab.md)).

## Data Sources

- **Quran Foundation API** — Provides tajweed-annotated text with rule tags.
- **Tanzil tajweed text** — Annotated Quranic text with tajweed markers.
- **everyayah.com tajweed images** — Pre-rendered tajweed-colored page images.

## Best Practices

- **Make tajweed colors toggleable.** Not all users want colored text — advanced readers find it distracting.
- **Provide a color legend** accessible from the tajweed view so users can learn what each color means.
- **Use pre-annotated data** rather than implementing rule detection from scratch. The rules have edge cases that are easy to get wrong.
- **Test with a tajweed expert.** Incorrect tajweed coloring is worse than no coloring.
- **Accessibility:** Ensure color choices have sufficient contrast. Offer alternative indicators (underline, bold) for color-blind users.
- **Dark mode:** Tajweed colors must remain distinguishable on dark backgrounds. Test and adjust the palette for both modes.
- **Per-qira'a consideration:** Some tajweed rules differ between qira'at. If supporting multiple qira'at, the tajweed annotations must match the active qira'a.
