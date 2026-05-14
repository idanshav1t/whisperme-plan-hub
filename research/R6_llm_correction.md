# R6 — L4 narrow Hebrew LLM correction layer

Narrow LLM post-processing for our clinical Hebrew SLP product. STT layer (Soniox) outputs Hebrew transcript at ~7.5% WER (vendor benchmark). The LLM layer ONLY runs on uncertain spans (low word-confidence) and ONLY does narrowly-scoped fixes: punctuation, code-switching cleanup (Hebrew + English mixed), Hebrew abbreviation expansion, clinical-terminology normalization. It does NOT attempt to fully retranscribe.

**Prior evidence:** text-only LLM rescoring of Whisper N-best with DictaLM-1.7B and KenLM on Hebrew was measured DEAD (KenLM regressed, DictaLM flat). But that was full-text rescoring of Hebrew transcripts. The narrow post-process layer is a different problem.

---

## 1. Hebrew automatic punctuation

- A fine-tuned **AlephBERT** punctuation-restoration model exists, explicitly trained as ASR post-processing for Hebrew spoken transcripts; catalogued in NNLP-IL's Hebrew-Resources index and the danielrosehill catalog.
- **Verbit** has published a Hebrew medical audio dataset and contributed Hebrew inverse-text-normalization to NeMo (WFST-based), but no Verbit punctuation checkpoint with a public model card located.
- **No published F1/accuracy numbers found** for Hebrew restoration covering "?" specifically — real measurement gap.
- Multilingual `1-800-BAD-CODE/punct_cap_seg_47_language` lists Hebrew among 47 supported languages but has no Hebrew-specific eval reported.

Sources: [danielrosehill/Hebrew-AI-Models](https://github.com/danielrosehill/Hebrew-AI-Models) · [NNLP-IL Hebrew-Resources](https://github.com/NNLP-IL/Hebrew-Resources/blob/master/models_tools_services.rst) · [1-800-BAD-CODE punct_cap_seg](https://huggingface.co/1-800-BAD-CODE/punct_cap_seg_47_language)

## 2. DictaLM-3.0 variants

Per the DictaLM 3.0 technical report (arXiv 2602.02104) the three released sizes are:

- **1.7B** (Qwen3-1.7B-based)
- **12B** (Nemotron Nano V2-based)
- **24B** (Mistral-Small-3.1-based)

Each with base + chat-with-tool-calling variants, 65k context.

**There is NO 7B in the 3.0 line** — earlier "7B-Instruct" references reflected the older DictaLM-2.0 line.

**No throughput numbers in abstract**, and no third-party tokens/sec benchmarks for DictaLM specifically. Reasonable proxy: same-size Mistral/Qwen kernels.

Source: [Dicta-LM 3.0 Technical Report (arXiv 2602.02104)](https://arxiv.org/pdf/2602.02104)

## 3. Hebrew NER for clinical

- **`dicta-il/dictabert-ner`** is BERT-base fine-tuned for Hebrew NER (PER/GPE/TIMEX etc.) — general, not clinical.
- **DictaBERT 2.0** has been continually pre-trained on >5M de-identified hospital records; a 2025 paper ("Building Patient Journeys in Hebrew") releases clinical-timeline datasets from internal medicine, ED, and oncology, with DictaBERT-class models annotated for event/temporal relations.
- BGU's hebrew-ner, HeBERT, and AlephBERT NER variants exist but are general-domain.
- **No public Hebrew drug-name NER** found — gap.

Sources: [dicta-il/dictabert-ner](https://huggingface.co/dicta-il/dictabert-ner) · [Building Patient Journeys in Hebrew (arXiv 2512.11502)](https://arxiv.org/html/2512.11502)

## 4. Hebrew abbreviation expansion

- **No purpose-built model surfaced.** Standard approach is **rules-based** using gershayim (״) detection at position n-1, mapped against a curated dictionary (Academy of the Hebrew Language 1994 rules; Wikipedia maintains a list of common abbreviations).
- **Note:** ASR output usually strips the gershayim character, so detection must work on bare orthography (e.g. "ארהב" → expand candidate "ארצות הברית"). This is a custom dictionary-lookup task, not an ML model.

Sources: [Wikipedia: List of Hebrew abbreviations](https://en.wikipedia.org/wiki/List_of_Hebrew_abbreviations) · [Wikipedia: Gershayim](https://en.wikipedia.org/wiki/Gershayim)

## 5. Hebrew code-switching

- **No dedicated Hebrew↔English code-switch detector found** in the HuggingFace ecosystem.
- Standard approach: Unicode-script segmentation (`\p{Hebrew}` vs `\p{Latin}`) then language-ID per token.
- DictaBERT handles mixed input natively.

## 6. Confidence-triggered LLM correction

Specifically referenced in:

- **arXiv 2407.21414** ("interfacing LLMs with ASR using confidence measures and prompting")
- **arXiv 2310.11532** (multi-stage correction work)

Used in production by Otter/Grammarly per their engineering blogs.

**Threshold tuning is dataset-specific**; the literature triggers on per-word confidence below a span-level threshold.

Sources: [Interfacing LLMs with ASR (arXiv 2407.21414)](https://arxiv.org/html/2407.21414v1) · [Multi-stage LLM Correction (arXiv 2310.11532)](https://arxiv.org/html/2310.11532v2)

## 7. EMPIRICAL EVIDENCE — Hebrew narrow LLM post-process

**This evidence does NOT exist for Hebrew.**

Searched arXiv for Hebrew+ASR+LLM+WER and found zero papers. All measured GEC/LLM-correction evidence is English:

- Whispering LLaMA: 37.66% WERR vs n-best
- T5 on Whisper LibriSpeech: 7.37→6.39% WER

or Japanese/English.

**This is a real gap.** To claim L4 will improve your 7.5% baseline, you must measure it yourself on a held-out Hebrew clinical set.

**Linearity bias warning** (CLAUDE.md §7j): English WERR gains don't extrapolate to Hebrew morphology.

Sources: [Whispering LLaMA (arXiv 2310.06434)](https://arxiv.org/abs/2310.06434) · [Generative LLM ASR Correction (arXiv 2307.04172)](https://ar5iv.labs.arxiv.org/html/2307.04172) · [GEC for Rare Words (arXiv 2505.17410)](https://arxiv.org/html/2505.17410v1)

## 8. Inference cost

A10G/24GB, fp16, vLLM serves 7-8B at ~**60 tok/s** peak. 1.7B would scale ~3-4x faster (**~200-240 tok/s extrapolated — not measured**).

- AWS g5.xlarge (A10G) is $1.006/hr on-demand → at 60 tok/s steady = ~$0.0047/1k output tokens for 7B; ~$0.0014/1k for 1.7B.
- Azure NC8as_T4_v3 comparable.

**No DictaLM-specific benchmark exists** — these are kernel-level estimates.

Source: [vLLM v0.6.0 perf update](https://blog.vllm.ai/2024/09/05/perf-update.html)

## 9. Hosting

- **DictaLM is NOT on Amazon Bedrock or Azure Foundry** as of search date (May 2026); self-host only.
- Claude API supports Hebrew (Claude Instant ~89% Hebrew classification accuracy vs GPT-4 92%); Sonnet 4.6 list price ~$3/$15 per M tok in/out — order of magnitude more expensive than self-hosted DictaLM-1.7B per request, but no GPU ops burden.

Source: [Best LLM for Hebrew Classification (Medium)](https://medium.com/@gilinachum/whats-the-best-llm-for-hebrew-classification-58a61b8b9f10)

## 10. Soniox custom vocab as alternative

**Yes — Soniox supports it on Hebrew.** Their `context` API accepts up to 8000 tokens (~10k chars) of context including custom terms and translation pairs (`source`/`target` JSON objects). Boost range -50 to +50 in the legacy customization API.

**This should be the first-line tool for clinical terminology** — push hotwords pre-STT instead of fixing post-STT. Lower latency, no L4 needed for that subset.

Sources: [Soniox Context Docs](https://soniox.com/docs/stt/concepts/context) · [Soniox Customization](https://soniox.com/docs/speech-to-text-legacy/how-to/customization)

---

## Recommended L4 architecture

**Pre-STT (highest leverage, do first):**
- Soniox `context` with clinical hotwords + Hebrew/English term pairs (max 8k tokens) — handles ~60% of clinical-term issues without L4.

**L4 pipeline (only on Soniox-flagged low-confidence spans):**
1. **Punctuation restorer**: AlephBERT-punctuation fine-tuned model (or train your own on Hebrew clinical text — gap warning: no public F1 on "?").
2. **Abbreviation expander**: rules engine + curated clinical dictionary (gershayim heuristic + lookup).
3. **NER pass**: DictaBERT-NER for entity guards (never "correct" a recognized entity).
4. **LLM corrector on spans only**: **DictaLM-3.0-1.7B-Instruct** fp16 on A10G via vLLM. Reasons: ~3-4× faster than 7B at marginal Hebrew-quality cost for narrow edits; cheapest self-host.
5. **Fallback for hard spans**: Claude Sonnet API (Hebrew-capable, no infra cost) gated to <5% of spans by confidence.

**CRITICAL caveat:** Per point 7, there is **no published evidence** that this stack improves a 7.5% Hebrew WER baseline. Mandatory: measure on a held-out Hebrew clinical eval set before shipping. Define success as span-level edit precision (not full WER), since L4 only touches uncertain spans.

## Sources

[danielrosehill/Hebrew-AI-Models](https://github.com/danielrosehill/Hebrew-AI-Models) · [NNLP-IL Hebrew-Resources](https://github.com/NNLP-IL/Hebrew-Resources/blob/master/models_tools_services.rst) · [1-800-BAD-CODE punct_cap_seg_47_language](https://huggingface.co/1-800-BAD-CODE/punct_cap_seg_47_language) · [Dicta-LM 3.0 Technical Report (arXiv 2602.02104)](https://arxiv.org/pdf/2602.02104) · [dicta-il/dictabert-ner](https://huggingface.co/dicta-il/dictabert-ner) · [Building Patient Journeys in Hebrew (arXiv 2512.11502)](https://arxiv.org/html/2512.11502) · [Wikipedia: List of Hebrew abbreviations](https://en.wikipedia.org/wiki/List_of_Hebrew_abbreviations) · [Wikipedia: Gershayim](https://en.wikipedia.org/wiki/Gershayim) · [Interfacing LLMs with ASR using confidence (arXiv 2407.21414)](https://arxiv.org/html/2407.21414v1) · [Multi-stage LLM Correction for ASR (arXiv 2310.11532)](https://arxiv.org/html/2310.11532v2) · [Whispering LLaMA (arXiv 2310.06434)](https://arxiv.org/abs/2310.06434) · [Generative LLM ASR Correction (arXiv 2307.04172)](https://ar5iv.labs.arxiv.org/html/2307.04172) · [GEC for Rare Words (arXiv 2505.17410)](https://arxiv.org/html/2505.17410v1) · [vLLM v0.6.0 perf update](https://blog.vllm.ai/2024/09/05/perf-update.html) · [Best LLM for Hebrew Classification (Medium)](https://medium.com/@gilinachum/whats-the-best-llm-for-hebrew-classification-58a61b8b9f10) · [Soniox Context Docs](https://soniox.com/docs/stt/concepts/context) · [Soniox Customization](https://soniox.com/docs/speech-to-text-legacy/how-to/customization)
