# Mushaf Page Display

## Table of Contents
- [Approaches](#approaches)
- [SVG-Based Rendering](#svg-based-rendering)
- [Image-Based Rendering](#image-based-rendering)
- [Text-Based Rendering](#text-based-rendering)
- [Page Navigation](#page-navigation)
- [Best Practices](#best-practices)

## Approaches

Three main approaches for rendering mushaf pages, each with trade-offs:

| Approach | Pros | Cons | Best For |
|----------|------|------|----------|
| **SVG** | Scalable, interactive, clickable ayahs, searchable | Larger initial load, complex rendering | Web apps, interactive features |
| **Image** | Pixel-perfect, simple, fast render | Not scalable, not searchable, large files | Offline-first apps, print fidelity |
| **Text layout** | Fully dynamic, searchable, smallest size | Hard to match mushaf layout exactly | Reading-focused apps without mushaf fidelity |

## SVG-Based Rendering

Use **quranpedia/quran-svg** — provides SVG pages for 5 mushafs (Hafs, Warsh, Qalun, Duri, Shubah).

### Structure per mushaf

```
mushafs/{mushaf}/
  svg/       # Raw SVG files (001.svg – 604.svg)
  svg-br/    # Brotli-compressed (~83% size reduction)
  json/      # Polygon metadata for ayah regions
```

### Interactive ayah regions

Each SVG contains transparent polygon overlays per ayah with attributes:
- `surah` — chapter number
- `ayah` — verse number  
- `number` — 6-digit ID in SSSAAA format (e.g., `002005` = Surah 2, Ayah 5)

### Multi-surah pages

Pages containing multiple surahs have separate SVG variants:
- `106.svg` — full page
- `106-surah4.svg` — only surah 4 content
- `106-surah5.svg` — only surah 5 content

### Styling ayah interactions

```css
polygon { fill: transparent; cursor: pointer; }
polygon:hover { fill: rgba(0, 120, 255, 0.15); }
polygon.selected { fill: rgba(0, 120, 255, 0.25); }
polygon.playing { fill: rgba(255, 200, 0, 0.2); } /* audio highlight */
```

### Performance

- Serve brotli-compressed SVGs (`svg-br/`) with `Content-Encoding: br` header.
- Lazy-load pages — only load the current page and ±1 adjacent pages.
- Cache aggressively — mushaf pages never change.

## Image-Based Rendering

Sources: King Fahd Complex, Shamarly (Hafs), EveryAyah PNG/JPG.

- Use high-DPI images (2x or 3x) for retina displays.
- Overlay transparent hit regions using the JSON metadata from quran-svg for interactivity.
- Consider WebP format for 25-35% size reduction over PNG.

## Text-Based Rendering

For apps that don't need exact mushaf layout:

- Render ayahs sequentially with proper Quranic font.
- Add surah headers (bismillah, surah name, ayah count, revelation type).
- Group by page, juz, or hizb as needed.
- Simpler to implement but won't match the mushaf look.

## Page Navigation

Mushaf pages map to known divisions:

| Division | Count | Navigation Pattern |
|----------|-------|-------------------|
| Pages | 604 | Direct page number |
| Juz | 30 | Each juz starts on a known page |
| Hizb | 60 | 2 per juz |
| Rub' al-Hizb | 240 | 4 per hizb |
| Surah | 114 | Each starts on a known page |

Provide multiple navigation methods: page number, surah selector, juz selector, and last-read position.

## Best Practices

- **Always show the mushaf edition/qira'a** somewhere in the UI so users know which mushaf they're viewing.
- **Support pinch-to-zoom** on mobile for mushaf pages.
- **Landscape mode:** Consider showing two-page spread (even/odd pages side by side) like an open mushaf.
- **Dark mode:** For SVG, invert colors or use CSS filters. For images, apply a carefully designed overlay — never just invert (it can make text illegible).
- **Orientation:** The standard mushaf opens right-to-left. In a swipeable view, swipe LEFT to go to the NEXT page (higher page number).
- **Loading states:** Show a subtle placeholder matching the page dimensions — never show a spinner over where Quranic text will appear.
