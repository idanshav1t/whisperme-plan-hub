# R1 — Soniox L0 deep dive

Technical and contractual diligence for Soniox as the speech-to-text foundation of a clinical Hebrew SLP product.

---

## 1. API surface

- **Async (file):** `POST https://api.soniox.com/v1/transcriptions`, plus `POST /v1/files`, `GET /v1/transcriptions/{id}/transcript`, `DELETE /v1/transcriptions/{id}`. Auth: `Authorization: Bearer {api_key}` header. ([Async API ref](https://soniox.com/docs/stt/async/async-transcription))
- **Streaming:** WebSocket at `wss://stt-rt.soniox.com/transcribe-websocket`. Auth: `"api_key"` field inside the first JSON config frame. End-of-audio = send empty string. ([Real-time API ref](https://soniox.com/docs/stt/rt/real-time-transcription))
- **SDKs:** Official [soniox-python](https://github.com/soniox/soniox-python) and [soniox-js](https://github.com/soniox/soniox-js) (Apache-2.0).

## 2. Hebrew-specific capabilities

- **No Hebrew-dedicated model.** Hebrew is one of 60+ languages on the unified `stt-rt-v4` / `stt-async-v4`. ([Models](https://soniox.com/docs/stt/models))
- Set via `language_hints: ["he", ...]` in the config frame.
- **Code-switching:** mid-utterance Hebrew⇄English supported, no manual switch; example `"אני צריך להזמין קפה לפני ה-meeting"`. ([Hebrew page](https://soniox.com/speech-to-text/hebrew))
- **Punctuation, slang, number normalization:** UNVERIFIED — docs claim alphanumeric handling (phone numbers, license plates) but no explicit Hebrew punctuation/digits-vs-spelled rules.

## 3. Word confidence

- Field name: **`confidence`**, type `number`, range **0.0–1.0**, "model's estimate of how likely the token was recognized correctly". ([Confidence scores](https://soniox.com/docs/stt/concepts/confidence-scores); [Node SDK types](https://soniox.com/docs/sdk/node-SDK/reference/types))
- **Calibration:** UNVERIFIED — docs explicitly do not claim calibration nor recommend thresholds.

## 4. N-best / alternatives

- **Not exposed.** Top-1 hypothesis only; no `alternatives`/lattice field in the documented token schema. ([WebSocket API](https://soniox.com/docs/stt/api-reference/websocket-api))

## 5. Diarization

- Request: `"enable_speaker_diarization": true`. Response: `speaker` string on each token. Up to **15 speakers**. Async > realtime accuracy.
- **Hebrew-specific diarization WER: UNVERIFIED.** ([Diarization docs](https://soniox.com/docs/stt/concepts/speaker-diarization))

## 6. Custom vocabulary / context

- `context` object with `general`, `text`, `terms`, `translation_terms`.
- Hard cap **8,000 tokens (~10K chars)**.
- `general` recommended ≤10 keys.
- Healthcare example in docs lists `Celebrex, Zyrtec, Xanax, Prilosec…`. ([Context](https://soniox.com/docs/stt/concepts/context))

## 7. Limits

- **Async:** max **300 min/file** (hard cap, not raisable), 10 GB storage, 1,000 stored files, 100 pending, 2,000 total. ([Async limits](https://soniox.com/docs/stt/async/limits-and-quotas))
- **Streaming:** **10 concurrent** WebSockets, **300 min/session**, **100 req/min**; raisable except session cap. ([RT limits](https://soniox.com/docs/stt/rt/limits-and-quotas))

## 8. BAA process

- Mechanism documented: "compliance documentation can be obtained through [Soniox Console](https://console.soniox.com/) → Security & compliance". ([Security & Privacy](https://soniox.com/docs/security-and-privacy))
- DPA is explicitly "pre-signed by Soniox and countersigned with one click". ([EU page](https://soniox.com/europe))
- **Whether BAA is self-serve in that same console or sales-gated: behind login — verify on signup.**

## 9. Pricing / discounts

- Async **$1.50/M audio tokens (~$0.10/hr)**.
- Streaming **$2.00/M (~$0.12/hr)**. ([Pricing](https://soniox.com/pricing))
- **Volume/enterprise discounts: not published.** FAQ routes Business/Enterprise to `support@soniox.com`. ([FAQ](https://soniox.com/docs/faq))

## 10. SLA

- **Contractual SLA percentage and service credits: UNVERIFIED** (no public SLA page found).
- Public [status page](https://status.soniox.com/) shows **100.0% trailing 90-day uptime** across US/EU/Japan/Console, no incidents past 7 days. Observational, not contractual.

## 11. EU residency

- EU endpoint: **`api.eu.soniox.com`**.
- Audio + transcripts stay in-region, no US fallback; "system data" (account, usage, billing) is excluded from residency guarantee. ([Data residency](https://soniox.com/docs/stt/data-residency); [Sovereign Cloud blog](https://soniox.com/blog/2025-11-19-sovereign-cloud))
- **Exact EU city/AZ (Frankfurt vs Ireland vs Belgium) and named sub-processors: UNVERIFIED — not in public docs.** Enable via project region toggle in Console.

## 12. Hebrew competitor landscape

| Provider | Hebrew WER | Source |
|---|---|---|
| Soniox | 7.5% | [Hebrew page](https://soniox.com/speech-to-text/hebrew) (vendor benchmark, 2025 YouTube audio) |
| Google STT | 12.1% | Soniox-published bench |
| AWS Transcribe | 11.2% | Soniox-published bench |
| OpenAI Whisper | 16.1% | Soniox-published bench |
| ElevenLabs Scribe v1 | 15.2% | [elevenlabs.io/speech-to-text/hebrew](https://elevenlabs.io/speech-to-text/hebrew) |
| ElevenLabs Scribe v2 Realtime | ">10% to ≤25%" tier | [Scribe v2 RT](https://elevenlabs.io/realtime-speech-to-text) (Hebrew supported but worse tier) |
| Deepgram Nova-3 Hebrew | UNVERIFIED | [Deepgram Hebrew](https://deepgram.com/learn/speech-to-text-for-hebrew-persian-urdu-on-nova-3) (ships 2025, no published WER) |
| ivrit-ai/whisper-large-v3 | UNVERIFIED | [HF leaderboard](https://huggingface.co/spaces/ivrit-ai/hebrew-transcription-leaderboard) didn't render via fetch |
| Azure STT Hebrew | UNVERIFIED | — |

## 13. Real-world reputation

- Multiple HN threads, mostly positive, focused on price and translation: [v3 launch](https://news.ycombinator.com/item?id=45655859), [60-lang RT](https://news.ycombinator.com/item?id=46891922), [pricing](https://news.ycombinator.com/item?id=45408160).
- No outage complaints surfaced. No substantive Reddit production threads.

---

## Gaps that need verification on signup / use

1. Exact EU datacenter region + named sub-processor list.
2. BAA self-serve countersign vs sales-mediated; plan-tier minimum.
3. Contractual SLA %, service credits, force-majeure clauses.
4. Volume pricing below $0.10/$0.12 per hour.
5. Confidence calibration data on Hebrew specifically.
6. Hebrew diarization WER, Hebrew punctuation/number-normalization behavior.
7. Independent (non-Soniox-published) Hebrew WER vs Deepgram Nova-3 / Azure / ivrit-ai.
8. Whether N-best output is on roadmap (currently absent).

## Sources

[Soniox docs root](https://soniox.com/docs) · [Real-time](https://soniox.com/docs/stt/rt/real-time-transcription) · [Async](https://soniox.com/docs/stt/async/async-transcription) · [WebSocket ref](https://soniox.com/docs/stt/api-reference/websocket-api) · [Models](https://soniox.com/docs/stt/models) · [Confidence](https://soniox.com/docs/stt/concepts/confidence-scores) · [Diarization](https://soniox.com/docs/stt/concepts/speaker-diarization) · [Context](https://soniox.com/docs/stt/concepts/context) · [RT limits](https://soniox.com/docs/stt/rt/limits-and-quotas) · [Async limits](https://soniox.com/docs/stt/async/limits-and-quotas) · [Data residency](https://soniox.com/docs/stt/data-residency) · [Security & Privacy](https://soniox.com/docs/security-and-privacy) · [Pricing](https://soniox.com/pricing) · [Hebrew page](https://soniox.com/speech-to-text/hebrew) · [EU page](https://soniox.com/europe) · [Sovereign Cloud](https://soniox.com/blog/2025-11-19-sovereign-cloud) · [Status](https://status.soniox.com/) · [Python SDK](https://github.com/soniox/soniox-python) · [JS SDK](https://github.com/soniox/soniox-js) · [Deepgram Hebrew](https://deepgram.com/learn/speech-to-text-for-hebrew-persian-urdu-on-nova-3) · [Scribe v2 RT](https://elevenlabs.io/realtime-speech-to-text) · [ivrit-ai leaderboard](https://huggingface.co/spaces/ivrit-ai/hebrew-transcription-leaderboard)
