# I'rab (Grammatical Analysis)

## Table of Contents
- [Overview](#overview)
- [Three Kinds of I'rab Data](#three-kinds-of-irab-data)
- [Data Sources & Licenses](#data-sources--licenses)
- [The Token Alignment Problem](#the-token-alignment-problem)
- [Morphological Features](#morphological-features)
- [Use Cases in Quran Apps](#use-cases-in-quran-apps)
- [Design Rules](#design-rules)
- [Linguistic Adab](#linguistic-adab)
- [Best Practices](#best-practices)

## Overview

I'rab (إعراب) is the grammatical analysis of Quranic Arabic — identifying each word's syntactic role and case markers. Essential for educational apps and Arabic learning tools, and (via roots/lemmas) a force multiplier for search.

## Three Kinds of I'rab Data

Serious i'rab features combine three distinct layers — don't treat them as one:

| Layer | What it is | Granularity |
|-------|-----------|-------------|
| **Classical i'rab books** (كتب الإعراب) | Scholarly prose analyses | Per ayah, prose |
| **Morphology** (صرف) | Word segmentation: prefix/stem/suffix with POS, root, lemma, features | Per word segment |
| **Syntax** (نحو) | Dependency treebank: grammatical relations between words | Per token/relation |

A good UI offers them as three views: books (prose), morphology (word-by-word table), and tree (dependency arcs). Split trees at the treebank's own sentence boundaries — a long ayah becomes several readable charts.

## Data Sources & Licenses

| Source | Content | License |
|--------|---------|---------|
| **Quranic Arabic Corpus** (corpus.quran.com) | Full morphology of every word — the standard dataset | **GNU GPL, attribution required.** An Arabized fork exists (mustafa0x/quran-morphology) |
| **Quranic Treebank** | Dependency syntax | MIT |
| **QuranPedia API** | All three layers per ayah (`e3rab`, `morphology`, `syntax` services) with license blocks included — preserve them | per source |
| **Quran Foundation API** | Word-level morphology | — |

Never implement Arabic morphological analysis from scratch — the corpus already exists.

## The Token Alignment Problem

The corpus tokenizes **Uthmani** text (يَٰٓأَيُّهَا as one token) while your app's text may spell or split differently, so word sequences differ in both spelling and count — naive position matching mis-assigns grammar to words.

Solve it **once at import time** with a global sequence alignment (Needleman–Wunsch works, allowing 1↔2 merges), store the aligned result, and keep any corrections to upstream data in a separate reviewed file so future upstream fixes are detected instead of silently double-applied.

## Morphological Features

| Feature | Values | Example |
|---------|--------|---------|
| **Part of speech** | noun, verb, particle, pronoun, etc. | كِتَابٌ → noun |
| **Root** | 3–4 letter root | كِتَابٌ → ك-ت-ب |
| **Pattern (wazn)** | فِعَال, فَعِيل, مَفْعُول… | كِتَابٌ → فِعَال |
| **Case** | مرفوع / منصوب / مجرور | |
| **State** | definite, indefinite, construct | |
| **Person / Gender / Number** | 1st–3rd / masc, fem / sg, dual, pl | |
| **Verb form** | I–X | |
| **Voice / Mood** | active, passive / indicative, subjunctive, jussive, imperative | |

## Use Cases in Quran Apps

- **Word-by-word grammar view** — tap a word → segments, POS, root, case. Color-code by POS.
- **Root & lemma exploration** — navigate to all words sharing a root/lemma. Key lemma pages on the unvowelled form but keep homograph lexemes apart.
- **Grammar search** — "all form-II verbs", "all words from ر-ح-م" — index roots/lemmas per ayah ([search.md](search.md)).
- **Dependency tree display** — per treebank sentence.
- **Derived features** — once morphology + syntax are in your DB, root/lemma/proper-noun indexes, collocation views ("which words does this verb take as subject/object"), grammar lessons with real Quranic evidence, and drill questions all fall out of the same tables.

## Design Rules

- **Words, not counts.** Show the actual words filling a grammatical relation, not frequency tables (counts are dominated by repeated formulas like the basmala).
- **Suppress pronoun subjects in aggregates** (they're most subjects and pure noise) — but state how many were suppressed.
- **Drill questions must be answerable from the ayah itself** — a valid answer is another word in the ayah, not a relation label.
- **Evidence, not verdicts** on contested analyses — present the grammar, let scholars rule.
- **Report data gaps honestly** — label missing/cross-ayah relations instead of guessing.

## Linguistic Adab

Grammar terminology for the Quran carries its own adab — apply it as a presentation layer over imported data (never mutate the source), and keep judgment calls in human-reviewed data files, not runtime inference:

- Passive: **«مبنيٌّ لما لم يُسمَّ فاعلُه»**, not «مبني للمجهول».
- لفظ الجلالة as object: **«منصوبٌ على التعظيم»**.
- Imperatives addressed to Allah are **دعاء, not أمر**; لا before them is «حرف دعاء».
- **Never label a Quranic particle «زائد»** — use «صلة» or «تأكيد».
- **Don't display a root for لفظ الجلالة** (derivation is disputed); handle words like «رب» location-aware (divine in most contexts, human master in a few).

See [adab.md](adab.md) for general adab rules.

## Best Practices

- **Use pre-computed data** and honor its licenses (GPL attribution for the corpus).
- **I'rab is an educational feature, not a core reading feature** — keep it optional and non-intrusive.
- **Display terms in Arabic** (فاعل, مفعول به…) with optional English equivalents.
- **Link grammar to meaning**, not just labels.
- **Note scholarly disagreement** where multiple parsings exist.
