# R5 — Trust Mode UI + Linked Evidence patterns

State-of-the-art UI patterns for a clinical transcript review interface. Soniox returns Hebrew transcript + per-word confidence + word-level timestamps + diarization. The clinician needs to: trust the transcript at a glance, quickly catch + fix STT errors, play back specific audio spans to verify, annotate clinical observations linked to specific words/segments.

---

## 1. Abridge "Linked Evidence" (anchor pattern)

Abridge's core differentiator inside Epic is **Linked Evidence**: every sentence in the AI-generated note maps back to the exact transcript span and audio segment that produced it. The clinician clicks a note phrase to scroll the interactive transcript and play the source audio without replaying the encounter.

**Abridge does NOT publicly expose a per-word confidence heatmap** — trust is delivered via traceability, not color.

Sources: [abridge.com/platform/clinicians](https://www.abridge.com/platform/clinicians) · [abridge.com/ai](https://www.abridge.com/ai)

## 2. Dragon Medical One / Nuance

DMO is a **dictation** model (clinician dictates a finished note), not a review surface. It exposes visual indicators only for **microphone state** and audio capture — no per-word confidence shading. Corrections happen via natural-language voice commands ("correct X to Y") that also train the user's personal vocabulary.

Source: [microsoft.com/health/dragon-medical-one](https://www.microsoft.com/en-us/health-solutions/clinical-workflow/dragon-medical-one)

## 3. DeepScribe / Suki / Augmedix

Ambient-scribe products converge on **edit-anywhere with no confidence cues** in the rendered note. Augmedix uses a human-in-the-loop reviewer before delivery; DeepScribe and Suki present a "finalized" note minutes later that the clinician edits like a Word doc. None publicly surface STT confidence in the UI — they treat confidence as an internal QA signal, not a clinician-facing one.

Source: [deepscribe.ai/resources/best-ai-medical-scribes](https://www.deepscribe.ai/resources/best-ai-medical-scribes)

## 4. Otter / Trint / Sonix / Rev — gold standard for transcript editing

- **Click-any-word seeks audio**; the active word **highlights blue** as audio plays (karaoke pattern). ([Otter: Edit a conversation](https://help.otter.ai/hc/en-us/articles/360047731754-Edit-a-conversation))
- **Trint** adds 5-color highlighters + threaded comments on text ranges + collaborator @-mentions. ([Trint editor](https://support.trint.com/en/articles/3004480-trint-editor))
- **Rev/Sonix** support inline edit + speaker-label reassign + comment threads. ([Rev editor](https://support.rev.com/hc/en-us/articles/29824992702989-Transcription-Editor))

None visualize per-word STT confidence; the "play to verify" loop replaces it.

## 5. Streaming hypothesis revision (Soniox case)

Research on streaming ASR (Google, Microsoft) measures instability with **UPWR/UPSR** (unstable word/segment ratio). The accepted pattern is **prefix-locking**: render unstable suffix in a lighter weight ("tentative") and only allow editing on **finalized** prefix.

Letting users edit live tentative text causes lost edits when the recognizer rewrites.

Sources: [Bruguier — deflickering](https://www.bruguier.com/pub/deflickering.pdf) · [arxiv.org/abs/2006.01416](https://arxiv.org/abs/2006.01416)

## 6. Confidence visualization — academic finding

**The most recent CHI study (Kuhn & Kersken, CHI '25 Extended Abstracts)** tested confidence-based error highlighting in user correction interfaces and found it **neither improved correction efficiency nor was perceived useful** by participants.

Vertanen & Kristensson (CHI '08) had earlier shown modest benefits with subtle underlines.

**Implication: a loud red-yellow-green heatmap is likely worse than a single muted underline on low-confidence words.**

Sources: [dl.acm.org/doi/10.1145/3706599.3720038](https://dl.acm.org/doi/10.1145/3706599.3720038) · [arxiv.org/html/2503.15124v1](https://arxiv.org/html/2503.15124v1)

## 7. "Did you mean..." without N-best

Since Soniox doesn't expose N-best, the field-proven fallbacks are:

- **(a) phonetic-neighbor suggestions** (Soundex/Metaphone-adapted for Hebrew via SimHebrew or Double-Metaphone-Hebrew)
- **(b) LLM rewrite with audio context** (re-prompt a small model with the surrounding text + the audio clip)
- **(c) edit-history corpus**: mine prior clinician corrections for "X → Y" pairs and suggest historically-frequent fixes (Grammarly's GECToR-style approach)

Sources: [Grammarly engineering](https://www.grammarly.com/blog/engineering/efficient-on-device-writing-assistance/) · [GECToR](https://github.com/grammarly/gector)

## 8. Edit-capture as training signal

Grammarly's documented loop: **incorporate accepted user edits into training sets with privacy-preserving measures; flag high-confidence-low-acceptance cases for targeted retraining.** ([grammarly.com/blog/engineering/high-quality-nlp-datasets](https://www.grammarly.com/blog/engineering/high-quality-nlp-datasets/))

Default-on opt-in is standard; for HIPAA/clinical contexts, default must be **opt-in with explicit consent** and PHI-stripping before the edit pair leaves the device.

## 9. Mobile/tablet patterns

- **Google Recorder** shows real-time transcript scrolling with a waveform; tap any word to seek.
- **Apple Voice Memos** added tap-to-seek transcript in iOS 18 but transcription is post-hoc only.
- Tablet pattern: scrolling transcript on top half, large waveform + play controls bottom half (thumb-reachable).

Source: [Voice Memos vs Google Recorder](https://www.pocket-lint.com/apple-voice-memos-google-recorder-comparison/)

## 10. Accessibility (WCAG)

- 4.5:1 contrast minimum (AA); 7:1 for AAA.
- **~8% of male users have red-green CVD** — never encode confidence as red/green alone.
- Safe palette: blue→orange diverging, or single-color opacity ramp + redundant underline.

Sources: [WebAIM contrast](https://webaim.org/resources/contrastchecker/) · [Adobe Color CVD](https://color.adobe.com/create/color-accessibility)

## 11. Hebrew RTL specifics

Sonix documents the known issue: standard subtitle/transcript formats weren't designed for RTL and break on mixed Hebrew+English+numerals. Pattern that works:

- Text container is `dir="rtl"`
- **Audio controls stay LTR** (play buttons, timestamps `mm:ss` left-to-right) — Elfsight/Premiere both confirm "don't mirror media controls"
- Use Unicode bidi isolates (`U+2068`/`U+2069`) around English/number runs to prevent reordering

Source: [Sonix RTL guidance](https://help.sonix.ai/en/articles/12329367-why-doesn-t-right-to-left-text-like-arabic-or-hebrew-align-properly-in-transcripts-or-subtitles)

## 12. Annotation patterns

- **Hypothesis.is** is the canonical model: select range → adder popover → choose "highlight" (private) or "annotate" (threaded comment). Threads support replies, sharing scopes (private/group/public), and tags.
- **Trint** mirrors this with 5 highlighter colors + comment threads.
- **Notion/Roam** use block-level (not range-level) — less suitable for word-precise clinical annotation.

Source: [Hypothesis annotation basics](https://web.hypothes.is/help/annotation-basics/)

---

## Recommended Trust Mode UX recipe

1. **Primary trust mechanism = Linked Evidence (Abridge model), not color.** Click any word → audio seeks + plays 2s pre-roll. Tap-and-hold a sentence → loops it.
2. **Confidence cue: single muted dotted underline** on words below threshold (CHI '25 says heatmaps don't help; a subtle hint does no harm). Opacity ramp on the underline, not the text. No red/green.
3. **Streaming UX:** finalized text rendered solid; tentative suffix rendered at 60% opacity, **read-only until finalized** (prefix-lock). Edits queue if user clicks during stream.
4. **Correction affordance:** double-tap a word → bottom sheet with (a) play-this-word-only, (b) phonetic-neighbor suggestions from a Hebrew Double-Metaphone index, (c) LLM rewrite-in-context, (d) free-text edit.
5. **Annotation:** Hypothesis-style range select → highlighter (4 clinical colors: phonology / syntax / pragmatics / fluency) or threaded note. Annotation anchors to word IDs + timestamps so they survive transcript re-edits.
6. **RTL layout:** transcript `dir="rtl"`, audio rail LTR pinned bottom, timestamps `mm:ss` LTR with bidi isolates. Mobile: scrolling transcript top, persistent waveform + 48pt play button bottom (thumb zone).
7. **Edit-capture loop:** every accepted correction logs `{audio_span, original, corrected, user_id_hashed}` after on-device PHI scrub; opt-in default-off for clinical compliance.
8. **Accessibility:** AAA contrast (7:1), confidence redundantly encoded as underline + opacity (never color alone), full keyboard nav, ARIA live region announces newly-finalized text.

## Sources

[Abridge platform](https://www.abridge.com/platform/clinicians) · [Abridge AI](https://www.abridge.com/ai) · [Otter Edit a conversation](https://help.otter.ai/hc/en-us/articles/360047731754-Edit-a-conversation) · [Trint Editor](https://support.trint.com/en/articles/3004480-trint-editor) · [Rev Editor](https://support.rev.com/hc/en-us/articles/29824992702989-Transcription-Editor) · [Dragon Medical One](https://www.microsoft.com/en-us/health-solutions/clinical-workflow/dragon-medical-one) · [DeepScribe scribes overview](https://www.deepscribe.ai/resources/best-ai-medical-scribes) · [CHI '25 ASR confidence study](https://dl.acm.org/doi/10.1145/3706599.3720038) · [arXiv preprint](https://arxiv.org/html/2503.15124v1) · [Deflickering streaming ASR](https://www.bruguier.com/pub/deflickering.pdf) · [Stability metrics](https://arxiv.org/abs/2006.01416) · [Grammarly engineering](https://www.grammarly.com/blog/engineering/efficient-on-device-writing-assistance/) · [GECToR](https://github.com/grammarly/gector) · [Google Recorder vs Voice Memos](https://www.pocket-lint.com/apple-voice-memos-google-recorder-comparison/) · [Sonix RTL guidance](https://help.sonix.ai/en/articles/12329367-why-doesn-t-right-to-left-text-like-arabic-or-hebrew-align-properly-in-transcripts-or-subtitles) · [WebAIM contrast](https://webaim.org/resources/contrastchecker/) · [Adobe Color CVD palette](https://color.adobe.com/create/color-accessibility) · [Hypothesis annotation basics](https://web.hypothes.is/help/annotation-basics/)
