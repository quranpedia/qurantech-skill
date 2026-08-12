# Testing & QA for Quran Apps

## Table of Contents
- [Why Quran Apps Need Special QA](#why-quran-apps-need-special-qa)
- [Text Integrity Verification](#text-integrity-verification)
- [Rendering QA](#rendering-qa)
- [Multi-Qira'a Test Matrix](#multi-qiraa-test-matrix)
- [Audio QA](#audio-qa)
- [Search QA](#search-qa)
- [Accessibility QA](#accessibility-qa)
- [Editorial Pipelines at Scale](#editorial-pipelines-at-scale)
- [Expert Review](#expert-review)
- [Release Checklist](#release-checklist)

## Why Quran Apps Need Special QA

A rendering bug in a normal app is cosmetic. In a Quran app, a dropped diacritic, a missing waqf mark, or a mis-numbered ayah silently corrupts sacred text for every user. Text integrity bugs are the one category that must reach production probability zero — automate the checks, don't rely on eyeballing Arabic.

## Text Integrity Verification

The single most important test suite in any Quran app.

- **Byte-exact comparison against the source.** After every build/import step, compare each stored ayah byte-for-byte against the verified source (Tanzil, KFGQPC, QUL). Any transformation — trimming, normalization, encoding conversion — is a bug.
- **Checksum the corpus.** Compute a hash of the full text at import time and assert it at app startup / in CI. Catches partial writes, corrupted bundles, and accidental edits.
- **Assert counts from metadata, not constants:** 114 surahs; per-surah ayah counts matching the active mushaf's metadata; total ayahs matching the mushaf (never a hardcoded 6236).
- **Unicode pipeline test:** feed ayahs containing special Uthmani codepoints (small alef `U+0670`, small high seen, waqf marks `U+06D6–U+06ED`) through the full pipeline (DB → API → UI string) and assert the codepoints survive. JSON serializers, ORMs, and "sanitizer" middleware are common silent strippers.
- **Guard against normalization:** add a test asserting that `NFC(stored_text) != stored_text` cases remain unnormalized — i.e., nothing in the stack applies Unicode normalization (see [text-rendering.md](text-rendering.md)).
- **Basmala rules:** present at the start of every surah except At-Tawbah (9); part of the ayah text in Al-Fatiha 1:1 and inside An-Naml 27:30.

## Rendering QA

Automated string tests can't catch shaping bugs — use screenshot tests on real renderers.

- **Golden-glyph pages:** screenshot-test a fixed set of known-problematic passages: madd signs mid-word, alif khanjariyya, stacked diacritics, waqf marks, ayah-end markers with numbers. Compare against approved reference images.
- **Test iOS specifically.** CoreText breaks Uthmani fonts in ways Android/HarfBuzz doesn't (see [text-rendering.md](text-rendering.md) — the iOS CoreText problem). An app that renders perfectly on Android can be broken on iOS with the same font.
- **Font fallback test:** simulate font-load failure and assert the app shows a Quranic fallback font or the ayah reference — never a system Arabic font ([adab.md](adab.md)).
- **Dark mode:** verify tajweed colors remain distinguishable and mushaf images/SVGs remain legible (never naive color inversion).
- **Bidi/mixed content:** ayah + translation + Latin numbers on one screen; assert no reordering artifacts. Test with actual Quranic text, which has more combining marks than generic Arabic.
- **Text scaling:** maximum Dynamic Type / font-scale settings must not truncate ayahs (rewrap or scroll instead — truncation violates adab).

## Multi-Qira'a Test Matrix

Each supported riwayah multiplies the QA surface. Minimum matrix per riwayah:

| Check | Why |
|-------|-----|
| Total ayah count matches the counting system | Counts differ (6,204–6,236) |
| Boundary ayahs where systems split/merge | The classic cross-qira'a bug; get cases from quranpedia/qiraat-ayah-map (`merged`/`split` statuses) |
| Bookmarks/progress are mushaf-scoped | Page 300 in Hafs ≠ page 300 in Warsh |
| Font matches the riwayah | Hafs fonts misrender Warsh letterforms |
| Audio reciter's riwayah matches displayed text | A Hafs recording over Warsh text is a data bug |
| Known word variants render correctly | e.g., مالك vs ملك in Al-Fatiha |

## Audio QA

- **Boundary test:** every ayah audio file starts at the beginning of the ayah — spot-check per reciter, especially first/last ayahs of surahs.
- **Timestamp drift:** for word-level highlighting, verify sync at the start, middle, and end of long surahs (drift accumulates).
- **Completeness:** assert the app knows which surahs/ayahs a reciter is missing and degrades gracefully.
- **Interruption handling:** calls, alarms, and route changes (headphones unplugged) must pause cleanly and resume at an ayah boundary.

## Search QA

Test the Arabic normalization edge cases from [search.md](search.md) as fixtures:

- Query without diacritics matches Uthmani text with diacritics.
- Hamza/alef variants (أ إ آ ا) all match; ة matches ه.
- Reference patterns navigate directly: `2:255`, `البقرة 255`, `Al-Baqarah 255`.
- Results always contain complete ayahs, sorted in mushaf order.
- Zero-result queries show a graceful fallback, not an empty screen.

## Accessibility QA

Run with real assistive tech, not just automated audits (most Quran apps fail here — see [architecture.md](architecture.md)):

- VoiceOver (iOS) / TalkBack (Android) can navigate to and read every ayah, and announces surah:ayah references sensibly.
- All controls (play, repeat, bookmark) have labels; the mushaf page view exposes ayah-level elements, not one giant image.
- Contrast passes WCAG AA for translation/tafsir text and tajweed colors; color is never the only indicator.

## Editorial Pipelines at Scale

When the app carries large digitized corpora (tafsirs, fatwas, notes), spot-checking doesn't scale. Production patterns:

- **Findings → review → apply, never direct edits.** Automated proofreading (diacritics, verse-quote verification against the mushaf) should produce *findings* with pending/approved/rejected/applied states, batch actions ("approve all like this"), and revert — with writes restricted to an explicit whitelist of table/column pairs.
- **Verify quoted ayahs inside content.** Scholarly texts quote the Quran constantly; automatically check quoted verses against the verified mushaf text — quote corruption is the most common and most serious OCR error.
- **Grade content quality explicitly** (complete / usable / thin) and let the grade drive what gets displayed and indexed, rather than shipping everything.
- **Measure your UI choices with users.** For subjective features like tajweed coloring (mode, colors, sizes), a small in-app survey beats team intuition.

## Expert Review

Some things cannot be verified by tests:

- **Tajweed coloring** must be reviewed by someone qualified in tajweed — wrong coloring teaches wrong recitation.
- **Translations/tafsir content** must come from verified sources with licensing confirmed; digitized tafsirs need OCR-error spot checks.
- **Any AI-generated content** requires human scholarly review before release ([adab.md](adab.md)).
- Recruit beta testers from the actual audience (e.g., Warsh readers for a Warsh app, huffaz for a hifz app) — they catch errors developers can't see.

## Release Checklist

Before every release:

- [ ] Text integrity suite green (byte-exact + checksum + counts + Unicode survival)
- [ ] Golden-glyph screenshots approved on iOS and Android
- [ ] Multi-qira'a matrix run for every bundled riwayah
- [ ] Offline mode exercised: fresh install, airplane mode, core reading + audio + search work
- [ ] Audio spot checks per newly added reciter
- [ ] Screen reader pass on the main reading flow
- [ ] Licenses verified for any new translation/tafsir/audio content
- [ ] Data version + changelog updated (source and version of the Quranic text documented)
