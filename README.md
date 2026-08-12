# QuranTech — an Agent Skill for Quran App Development

[![Status: Experimental](https://img.shields.io/badge/status-experimental-orange)](#-status-experimental)
[![Agent Skill](https://img.shields.io/badge/type-agent%20skill-blue)](https://code.claude.com/docs/en/skills)
[![PRs Welcome](https://img.shields.io/badge/PRs-welcome-brightgreen)](#-contributing--feedback)

**QuranTech** teaches AI coding agents how to build Quran applications *correctly* — with verified text sources, qira'at-aware data models, proper Arabic rendering, and the etiquette (adab) the Quranic text deserves.

Building Quran apps has hidden domain traps that generic coding knowledge walks straight into: hardcoding 6,236 ayahs (counts differ across riwayat), matching verses across mushafs by number (they merge and split), rendering Uthmani script in system fonts (glyphs break), auto-generating tafsir with an LLM (dangerous), Unicode-normalizing the text (destroys it). This skill encodes the know-how that prevents all of that — gathered from production Quran platforms, open datasets, and the Muslim developer community.

Works with [Claude Code](https://code.claude.com), [Claude.ai](https://claude.ai), the Claude API, and any agent runtime that supports the [Agent Skills](https://code.claude.com/docs/en/skills) format.

## What it covers

The skill uses progressive disclosure: a lean `SKILL.md` entry point, with 19 focused reference files the agent loads only when the task needs them.

| Domain | References |
|--------|-----------|
| **Data & sources** | Data source catalog · API comparison (Quran Foundation, QUL, Tanzil, QuranPedia, MP3Quran…) · QuranPedia API deep-dive (endpoints, bulk dumps, delta sync) · Data models & schemas |
| **Text & display** | Arabic text rendering, fonts & the iOS CoreText problem · Mushaf page display (SVG / image / text) · Tajweed coloring & waqf marks |
| **Qira'at** | All 10 qira'at & 20 riwayat · six ayah-counting systems · cross-mushaf mapping (the Hafs-anchor pattern) · per-riwayah fonts |
| **Audio** | Recitation playback modes · timing data & word-level highlighting · reciter/recitation modeling · verse recognition from audio (offline ONNX models) |
| **Scholarly content** | Translations & tafsir · i'rab (morphology + syntax treebanks) · content types beyond tafsir (asbab, fatwas, topics, gharib, mutashabihat…) |
| **Features** | Search (full-text, root-based, semantic — with a production Arabic search recipe) · memorization/hifz tools · embeddable Quran widgets & oEmbed |
| **Engineering** | Offline-first architecture · delta-sync freshness · testing & text-integrity QA · editorial pipelines |
| **Ethics** | Adab rules for handling sacred text — display, storage, logging, AI safety, linguistic terminology |

Two rules run through everything:

1. **Never compromise the text.** Verified sources only, byte-exact storage, no truncation, no normalization, integrity checks in CI.
2. **Don't reinvent solved problems.** The skill points to existing datasets, fonts, APIs, and open-source packages before recommending custom builds — and it stays **technology-agnostic**: the guidance applies whether you build in Flutter, Swift, Kotlin, React, Laravel, or anything else.

## Installation

### Claude Code

```bash
git clone https://github.com/quranpedia/qurantech-skill.git
mkdir -p ~/.claude/skills
cp -r qurantech-skill ~/.claude/skills/qurantech
```

That's it — Claude Code picks it up automatically. Verify with a prompt like *"add Warsh support to my Quran app"* and watch the skill trigger.

### Claude.ai / Claude API

Upload the packaged skill file (`dist/qurantech.skill`) via **Settings → Capabilities → Skills** on Claude.ai, or attach it through the API's skills support. To rebuild the package yourself, use the [skill-creator](https://github.com/anthropics/skills) tooling:

```bash
python -m scripts.package_skill path/to/qurantech
```

## Example prompts

- *"Build a mushaf reader web app with tajweed coloring"*
- *"My app is Hafs-only — add support for Warsh"*
- *"Identify which ayah is being recited from microphone audio, fully offline"*
- *"Design the database schema for a multi-riwayah Quran app with bookmarks"*
- *"I have a WordPress tafsir blog — let readers tap a verse to see its tafsir"*
- *"Design a hifz app with automatic recitation checking"*
- *"Which Quran API should I use for translations in 50 languages?"*

## Repository layout

```
qurantech-skill/
├── SKILL.md              # Entry point: triggers, workflow, principles, reference index
├── references/           # 19 domain reference files (loaded on demand)
├── evals/                # Test prompts + assertions for benchmarking the skill
├── dist/                 # Packaged .skill file for Claude.ai / API
└── README.md             # You are here
```

## 🧪 Status: Experimental

This skill is **young and actively evolving**. The guidance is drawn from real production systems and verified datasets, but coverage is uneven, some recommendations will age as APIs and datasets evolve, and we are still benchmarking how much it improves agent output (an eval suite lives in `evals/`).

Treat it as a knowledgeable colleague, not a fatwa: **verify religious-content decisions with qualified scholars, and verify technical output like any other code review.** If the skill leads an agent to do something wrong — especially anything touching text integrity or Islamic content — we want to know immediately.

## 🤝 Contributing & Feedback

This project gets better through the community's eyes. All of these help:

- **🐛 Report issues** — wrong facts, dead links, outdated API details, misbehaving guidance, or an agent doing something un-adab with the skill loaded. Open an issue with the prompt you used and what went wrong.
- **📚 Improve references** — deeper qira'at knowledge, more verified data sources, corrections from scholars, new domains (recitation pedagogy, accessibility, Indo-Pak script…).
- **🧪 Add evals** — realistic prompts + objective assertions in `evals/evals.json` are as valuable as content.
- **🌍 Share experience** — built a Quran app using this skill? Tell us what the skill missed; the gaps you hit are the roadmap.

Guidelines for content PRs:

1. **Cite sources** — guidance should trace to verified data, production experience, or scholarly reference.
2. **Stay technology-agnostic** — patterns over frameworks; name a specific stack only as evidence that a pattern works.
3. **Be lean** — this content ships inside an AI context window; every paragraph must earn its tokens.
4. **Respect the adab rules** — including in code examples and test data (`references/adab.md`).

## Acknowledgements

This skill stands on the work of the wider Quran-tech community: the King Fahd Glorious Quran Printing Complex, Tanzil, the Quran Foundation / Quran.com, QUL (Tarteel), QuranPedia, MP3Quran, EveryAyah, the Quranic Arabic Corpus, the offline-tarteel project, and the [Itqan community](https://itqan.dev) of developers serving the Quran. May Allah reward them all.

## License

[MIT](LICENSE) — free to use, modify, and redistribute. Note that the *datasets and content sources the skill points to* each carry their own licenses (some require attribution, e.g. the Quranic Arabic Corpus); the skill's references flag these where they apply.
