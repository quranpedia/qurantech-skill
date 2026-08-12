# Adab (Etiquette) Rules for Quran Applications

These rules are **mandatory** — not suggestions. Every Quran app must respect the sanctity of the Quranic text.

## Text Integrity

- **Never truncate an ayah mid-text.** Always display complete ayahs. If space is limited, show fewer ayahs rather than cutting one short.
- **Never split a word across lines in a way that breaks meaning.** Use proper Arabic line-breaking rules.
- **Always include the Basmala** (بسم الله الرحمن الرحيم) at the beginning of every surah except At-Tawbah (Surah 9). In An-Naml (27:30), the Basmala appears within the ayah itself.
- **Preserve diacritical marks** (tashkeel/harakat) in Quranic text. Never strip them for convenience.
- **Use verified text sources only.** Never manually type Quranic text — always pull from authenticated sources (King Fahd Complex, Tanzil verified, Quran Foundation). This applies to **every place Quranic text appears**: UI, code snippets, sample data, documentation, and prose explanations alike. When writing *about* an ayah in a plan or answer, cite it by reference (surah:ayah) instead of typing it — hand-typed text in prose carries the same corruption risk (dropped shadda, truncated verse) as in code.

## Display & Presentation

- **Ayah numbers must be accurate** to the specific riwayah/counting system being used. Hafs has 6,236 ayahs; Warsh has 6,214. Never assume a universal count.
- **Surah names and metadata must match the mushaf edition.** Different mushafs may use slightly different surah header styles.
- **Use appropriate Quranic fonts** — never render Quranic text in generic Arabic fonts. Use Uthmani script fonts (KFGQPC, Amiri Quran, etc.).
- **Right-to-left layout is mandatory** for all Quranic text. Ensure proper bidi handling even in mixed-language contexts.
- **Never place Quranic text in UI elements that imply dismissal** (e.g., swipe-to-delete, dismissible toasts, error messages).

## Data Handling

- **Never log Quranic text in error/debug logs.** Use ayah references (surah:ayah) instead of actual text in logs.
- **Variable and database column names should be respectful.** Avoid names like `quran_string`, `verse_blob`, or `raw_text`. Prefer `ayah_text`, `surah_name`, `mushaf_page`.
- **Never use Quranic text as test data, placeholder text, or lorem ipsum.**
- **Cache and store Quranic text with care.** Ensure cached text is not corrupted, partially written, or mixed with non-Quranic content.
- **Quranic text should not be modifiable by users** in the UI. It is read-only content.

## Audio Etiquette

- **Audio recitations must start from the beginning of an ayah**, not from the middle.
- **Provide a way to seek by ayah**, not arbitrary timestamps that land mid-recitation.
- **Attribute the reciter clearly** when playing audio.
- **Do not auto-play Quran recitation** without user intent — always require explicit user action.

## Search & Results

- **Search results must show complete ayahs**, not fragments or snippets with the match highlighted mid-word.
- **Never rank or sort Quranic ayahs by "relevance"** in a way that implies some ayahs are more important than others. Sort by mushaf order (surah:ayah) by default.

## Error States

- **If Quranic text fails to load, show a respectful placeholder** (e.g., "Unable to load ayah" with the reference) — never show broken/partial text.
- **If a font fails to load, fall back to another Quranic font**, not a system font. If no Quranic font is available, show the reference only.

## AI & LLM Integration

- **Never let AI generate tafsir freely.** Use RAG (Retrieval-Augmented Generation) with authoritative tafsir sources only.
- **AI hallucination in Quranic context is unacceptable.** Real example: an LLM explained "Fa waylun lil-musalleen" (Al-Ma'un:4) without the crucial context that the "woe" is for those who are *negligent* of their prayer, not those who pray. Without the following ayah and asbab al-nuzul, the AI gives misleading interpretations.
- **Human review is mandatory** for any AI-generated Islamic content before it reaches users.
- **AI-generated translations must be clearly labeled** as AI-generated and not attributed to any scholar.
- **Include transparent sourcing** — always show which tafsir/translation the AI's answer is based on.

## Accessibility

- **Screen reader compatibility is mandatory**, not optional. Many popular Quran apps are completely inaccessible.
- **Blind users need the simplest possible interface** — every unnecessary UI element is an obstacle.
- **Test with VoiceOver (iOS), TalkBack (Android), and NVDA/JAWS (desktop).**

## Translations & Tafsir

- **Always label translations as "ترجمة معاني القرآن"** (Translation of the meanings of the Quran), never as the Quran itself. Make the distinction visually clear.
- **Always attribute translations** to their author/scholar.
- **Tafsir must be clearly separated** from the Quranic text visually and semantically.

## Linguistic Adab (Grammar Features)

If the app displays grammatical analysis, the *terminology* itself carries adab. Classical scholars phrased grammar about the Quran with deliberate reverence — reproduce their choices:

- Passive verbs: **«مبنيٌّ لما لم يُسمَّ فاعلُه»**, never «مبني للمجهول».
- لفظ الجلالة as object: **«منصوبٌ على التعظيم»**.
- Imperatives addressed to Allah are **دعاء, not أمر**; لا before them is «حرف دعاء», not «ناهية».
- **Never label any Quranic particle «زائد»** (redundant); use «صلة» or «تأكيد».
- **Don't derive or display a root for لفظ الجلالة** — its derivation is disputed among scholars.

Implement these as a presentation layer over imported linguistic data (never edit the data), and keep judgment calls — like which addresses are divine — in a human-reviewed data file rather than inferring at runtime. Full detail and rationale: [irab.md](irab.md).
