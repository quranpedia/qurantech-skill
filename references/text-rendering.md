# Arabic Text Rendering & Fonts

## Table of Contents
- [RTL Layout](#rtl-layout)
- [Quranic Fonts](#quranic-fonts)
- [Font Selection Guide](#font-selection-guide)
- [Unicode & Encoding](#unicode--encoding)
- [Line Breaking](#line-breaking)
- [Common Pitfalls](#common-pitfalls)

## RTL Layout

- Set `dir="rtl"` on the root container of any Quranic text.
- Use CSS `direction: rtl` and `text-align: right` for Quranic content areas.
- In mixed-language UIs (Arabic + English), use CSS `unicode-bidi: isolate` to prevent bidi algorithm issues.
- Flexbox and Grid: use `flex-direction: row-reverse` or logical properties (`margin-inline-start` instead of `margin-left`).
- Mobile: set `android:layoutDirection="rtl"` or use SwiftUI's automatic RTL support with `Environment(\.layoutDirection)`.
- **Test with actual Quranic text**, not generic Arabic — Quranic text has more diacritics and special characters.

## Quranic Fonts

| Font | Style | Qira'a | License | Best For |
|------|-------|--------|---------|----------|
| **KFGQPC Uthmanic Script Hafs** | Uthmani Naskh | Hafs | Free (King Fahd Complex) | Authentic mushaf appearance |
| **Amiri Quran** | Traditional Naskh | Hafs | OFL (Open Font License) | Web apps, good browser support |
| **Me Quran** | Modern digital | Hafs | Varies | Modern app aesthetics |
| **Digital Khatt** | Variable font | Hafs | Varies | Responsive typography |
| **KFGQPC Warsh** | Uthmani | Warsh | Free | North/West African apps |

### Font Loading Strategy

1. **Primary:** Load the Quranic font specific to the active mushaf/qira'a.
2. **Fallback:** Another Quranic font (e.g., Amiri Quran as fallback for KFGQPC).
3. **Never fall back to system Arabic fonts** for Quranic text — they lack proper glyph support for Uthmani script.
4. **Preload fonts** for Quranic text to avoid FOUT (Flash of Unstyled Text) showing broken glyphs.

### Web Font Setup

```css
@font-face {
  font-family: 'UthmanicHafs';
  src: url('/fonts/UthmanicHafs.woff2') format('woff2');
  font-display: block; /* block, not swap — avoid showing broken text */
  unicode-range: U+0600-06FF, U+FB50-FDFF, U+FE70-FEFF;
}
```

Use `font-display: block` (not `swap`) for Quranic fonts — showing Quranic text in the wrong font briefly is worse than a short blank.

## Font Selection Guide

**Deciding which font to use:**

1. **Which qira'a?** — Each qira'a may require a different font. Hafs fonts will not correctly render Warsh-specific letterforms.
2. **Mushaf fidelity vs. readability?** — Uthmani fonts (KFGQPC) match the printed mushaf exactly. Naskh fonts (Amiri) are more readable on screen.
3. **Platform constraints?** — Variable fonts work well on web. System-bundled fonts work better on mobile for offline support.
4. **Performance?** — Arabic fonts with full glyph tables are large. Subset if only Quranic text is needed (no need for modern Arabic glyphs).

## Unicode & Encoding

- **Always use UTF-8** encoding for Quranic text.
- Quranic Arabic uses characters from multiple Unicode blocks:
  - `U+0600–U+06FF` — Arabic (base letters, harakat)
  - `U+FB50–U+FDFF` — Arabic Presentation Forms-A (ligatures)
  - `U+FE70–U+FEFF` — Arabic Presentation Forms-B
  - `U+0610–U+061A` — Small superscript Quranic annotations
  - `U+06D6–U+06ED` — Quranic annotation marks (sajda, hizb, etc.)
- **Uthmani text uses special Unicode codepoints** not found in standard Arabic (e.g., small alef, superscript noon). Ensure your font and rendering pipeline supports them.
- **Normalization caution:** Never apply Unicode normalization (NFC/NFD) to Quranic text — it can destroy diacritical marks and special Uthmani characters.

## Line Breaking

- **Never break in the middle of a word.** Arabic cursive script connects letters; breaking mid-word is visually broken.
- **Prefer breaking at ayah boundaries** for Quranic text.
- **Use `word-break: keep-all`** in CSS to prevent mid-word breaks.
- **Justify carefully:** Arabic text justification uses kashida (elongation) not inter-word spacing. Use `text-align: justify` with a font/engine that supports kashida, or avoid justification.

## iOS CoreText Problem (Critical)

**Problem:** iOS CoreText does not automatically enable advanced OpenType features (contextual alternates, ligatures, mark positioning) that Quranic fonts depend on. Android uses HarfBuzz which enables these by default.

**Symptoms:** Broken madd signs, missing alif khanjariyya, incorrect ligatures in React Native / Expo / SwiftUI.

**Solutions (in order of reliability):**
1. **Manually activate OpenType features** in iOS — see `mickf.net/tech/activating-open-type-features-ios`
2. **Use QPC fonts from King Fahd Complex** designed for iOS — `github.com/mohamedshawky982/react-native-quran-hafs`
3. **Render as SVG** using `react-native-skia` or `react-native-svg` to bypass CoreText entirely — `github.com/batoulapps/quran-svg`
4. **Use per-page woff2 fonts** where each word is a single glyph — pixel-perfect but not searchable/copyable

**Font bugs:** KFGQPC Hafs font has known issues with madd signs in word middles and alif khanjariyya on mobile. Use **FontForge** (`fontforge.org`) to manually fix glyph issues if needed.

## Word-as-Glyph Rendering

Some approaches represent each Quran **word** as a single vector glyph. This produces pixel-perfect mushaf rendering but:
- Massive file sizes (one glyph per word across 77,430+ words)
- Cannot be searched or copy-pasted
- Each page becomes its own font file

Quran Foundation provides per-page woff2 files: `verses.quran.foundation/fonts/quran/hafs/v2/woff2/p{PAGE}.woff2`

Use this approach only when exact mushaf fidelity is the top priority and search/copy are handled separately.

## The Uthmani Script Unicode Problem

Standard Unicode cannot perfectly represent Uthmani script. Some mushaf apps use characters from "Arabic Presentation Forms-A" — these are decorative glyphs, not readable text. Implications:
- Copy-paste from the app may produce garbage text
- Screen readers may not read correctly
- Search requires mapping presentation glyphs back to standard text
- Standard text shaping engines (HarfBuzz, CoreText) cannot produce pixel-perfect Madinah mushaf layout from Unicode alone

**Recommendation:** Use standard Unicode Arabic for searchable/accessible text, and SVG or per-page fonts for visual fidelity. Maintain both representations.

## Common Pitfalls

1. **String length ≠ character count** in Arabic. Diacritics are separate Unicode characters. `"بِسْمِ"` is 4 visible letters but 7 Unicode code points.
2. **String reversal breaks Arabic.** Never reverse a string containing Arabic text.
3. **Substring operations are dangerous.** Cutting at byte/codepoint boundaries can split a letter from its diacritics.
4. **Copy-paste introduces invisible characters.** Always sanitize pasted Quranic text against verified sources.
5. **Emoji and Arabic in the same line** can cause bidi rendering chaos. Isolate them with `U+2068` (FSI) and `U+2069` (PDI) or CSS `unicode-bidi: isolate`.
6. **HTML tags in API responses.** Quran.com API translations sometimes contain `<sup>` tags. Clean with regex before display.
