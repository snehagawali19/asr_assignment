# ASR Shootout for Indian Conversational Speech — Report

*Author: [your name] · Time-boxed to 1 day*

---

## TL;DR (read this even if you read nothing else)

> **One paragraph here. State the recommendation upfront — not at the end.** Example shape:
> *"For our voice/telephony stack, ship Deepgram Nova-3 for live IVR (best latency + competitive accuracy) and self-hosted IndicConformer for batch WhatsApp voice notes (cuts cost ~80% with no accuracy loss on our use case). Add a phonetic post-processor on top of either — it recovers ~25% of locality misses regardless of model. Whisper-via-Groq is good for prototyping but its hallucination patterns on pure Hindi/Kannada disqualify it from production."*

| Provider | LRA-fuzzy | WER | p50 latency | $ at 50k min/mo | Verdict |

| Deepgram Nova-3 | XX% | X.XX | XXX ms | $XXX | **Recommended for live IVR** |
| Sarvam Saaras | XX% | X.XX | XXX ms | $XXX | **Recommended for batch** |
| Whisper large-v3 (Groq) | XX% | X.XX | XXX ms | $XXX / free tier | Prototyping only |

---

## 1. Setup

**Primary evaluation set: 20 self-recorded utterances** of Bangalore locality names, designed across:
- **Language**: Hinglish (16), pure Hindi (1), pure Kannada (1), edge-case whispered/rushed (2)
- **Channel**: native phone mic, recorded phone call, WhatsApp voice note
- **Noise**: quiet room, fan, traffic, TV, music, kitchen
- **Difficulty**: 10 easy (common English-origin like *Whitefield*), 5 medium, 5 hard (long Kannada compounds like *Byatarayanapura*, *Kadugondanahalli*)

The full design is in `data/metadata.csv`. Recordings are improvised from a prompt (not read off a script), to capture realistic disfluencies, code-switches, and tonal variation.

### Datasets considered (and why)

| Dataset | Considered for | Decision | Why |
|---|---|---|---|
| **FLEURS Hindi** | Clean-Hindi generalization check | ✅ **Used** (25 clips) | Standard Hindi ASR benchmark; one `load_dataset()` call; gives me a clean-speech control to compute each provider's "real-world penalty" |
| **GramVaani Hindi** | Telephony-realistic Hindi from rural callers | ⏳ **Mentioned, not used** | Microsoft ASR challenge data — closest public proxy for our exact deployment (8kHz phone codec, real callers, conversational). Data acquisition has friction; **this is my #1 add with more time** |
| **Svarah** (AI4Bharat) | Indian-accented English on English-named localities | ⏳ **Mentioned, not used** | Tests how providers handle Indian-English phonology on words like *Whitefield* and *Electronic City* — half my localities are English-origin |
| **Kathbath** | Conversational Hindi | ❌ Skipped | More conversational than FLEURS but no telephony, no entities, larger setup cost |
| **MUCS 2021** | Hindi-English code-switching | ❌ Skipped | Highly relevant (code-switching is central to our use case) but Microsoft data access friction exceeded the 1-day budget |
| **Common Voice Hindi** | Crowdsourced Hindi | ❌ Skipped | Variable quality offers less signal than FLEURS for a clean-baseline control |
| **IndicVoices** | Large-scale Indic | ❌ Skipped | 1000+ hours; overkill for a sanity check; no marginal benefit over FLEURS for this purpose |
| **Shrutilipi** | Mined Indic speech-text pairs | ❌ Skipped | Wrong register (broadcasts), too general |
| **TTS-synthesized locality utterances** | Generate a 200+ utterance entity-heavy test set | 💡 **Future work** | Use IndicTTS to synthesize all 30 localities in 10+ voices/accents — turns 1 speaker bottleneck into many |

The selection thesis: **FLEURS provides a clean-Hindi control to interpret my noisy self-recording results.** The other datasets are either subsumed by that role (Kathbath, Common Voice, IndicVoices, Shrutilipi) or strictly better for a different question I didn't have time to answer (GramVaani, MUCS, Svarah).

## 2. Models and rationale

| Model | Why included |
|---|---|
| **Deepgram Nova-3** (required baseline) | Industry-standard commercial ASR; multilingual mode handles code-switching |
| **Sarvam AI Saaras** | India-specific commercial model; built explicitly for Indic + code-switched speech |
| **OpenAI Whisper large-v3 (via Groq)** | Open-source generalist baseline; Groq's free hosted inference removes the deployment cost from the comparison |

The matrix lets us answer the question: *do India-specific models actually beat generalist commercial models on Indian entities, and how big is the open-source vs commercial gap?* (Three was the budget — adding more without analyzing wouldn't have been better.)

Models considered and dropped: ElevenLabs Scribe (limited Indian-language signal), Google Chirp (slower setup), self-hosted IndicConformer (would have eaten setup budget; revisit if results justify).

## 3. Metrics — and why WER alone would have misled us

| Metric | Why |
|---|---|
| **WER / CER** | Standard for context, but weights every word equally — "main yahan rehta hoon" gets scored the same as the locality |
| **LRA-strict** | Did the exact locality name appear in the transcript? Binary per recording. **What the product actually cares about.** |
| **LRA-fuzzy** | Same with Levenshtein tolerance (partial-ratio ≥ 85). Reflects what's recoverable with a phonetic post-processor. |
| **Latency** (p50, p95) | API round-trip on 20 clips. Approximates production latency for batch; streaming numbers would differ. |
| **Cost at scale** | Modeled at MVP / Growth / Scale volumes. See `scripts/04_cost_analysis.py`. |

## 4. Results

### Headline

> *Paste the `summary_by_provider.csv` table from `scripts/02_metrics.py` here.*

### By condition (noise / channel)

> *Paste the `summary_by_condition.csv` table here.*
>
> Comment on the 1–2 biggest drops. Common patterns: phone-channel hurts all models, music noise hurts more than fan noise, whispered audio collapses entirely.

### By difficulty (locality complexity)

> *Paste the by-difficulty table from `scripts/02_metrics.py`.*
>
> Comment on the easy → hard gradient. Common pattern: 90%+ on easy, 30-50% on hard. The interesting question is *which provider degrades less steeply*.

### The surprise

> **This section is the most important paragraph in the report.** State which of your pre-registered hypotheses (see `HYPOTHESES.md`) was falsified, and what it implies for the product.
>
> Example: *"I expected Sarvam to dominate on Kannada-rooted compound names. Instead, all three providers landed in a tight band (X-Y% LRA-fuzzy), and the gap between strict and fuzzy LRA was larger than the gap between providers. The model choice matters less than building a phonetic post-processor — that's a 1-day implementation that probably recovers more accuracy than switching providers."*

### Cross-check on FLEURS Hindi (clean read speech)

> *Paste the `fleurs_summary.csv` table here.*
>
> The point is not the absolute FLEURS WER (which is published widely). The point is the **delta** with your self-recordings:

> *Paste the `realworld_penalty.csv` table here.*
>
> *"Provider X has the smallest real-world penalty (Δ = Y), meaning its FLEURS performance generalizes to noisy, code-switched, entity-heavy production-like audio. Provider Z looks competitive on FLEURS but its self-recording WER is N× higher — it's a clean-audio specialist and would underperform in our actual deployment."*
>
> If self-recording references aren't filled in, this table is replaced with a qualitative version comparing LRA-fuzzy and FLEURS WER instead. Note that explicitly.

## 5. Failure analysis

Pick 4–5 concrete examples from `scripts/05_substitutions.py`. Format:

> **`Byatarayanapura` → `bhayatara nagar puri`** (Whisper, quiet condition)
> Pattern: long compound name split into plausible-sounding shorter words. Even fuzzy match fails because the segmentation breaks the prefix. *Affects: Whisper on all `difficulty=hard` rows.*

> **`Whitefield` → `white field`** (Deepgram, noisy_kitchen)
> Pattern: anglicization splits the compound. Strict LRA fails but fuzzy LRA catches it (score = 89). *A 5-line post-processor that strips spaces before fuzzy matching fixes this category entirely.*

> **`Rajarajeshwarinagar` (whispered) → ` `** (all providers)
> Pattern: whispered audio yields empty transcripts from all three providers. *Edge case but documents the floor.*

> *Add 1–2 more from your actual data.*

### Failure taxonomy

Roll the per-example observations up into 3–4 categories. Typical:
- **Anglicization splits** — recoverable with normalization
- **Phonetic drift on compound names** — partially recoverable with fuzzy match
- **Whisper hallucinating English on pure Hindi/Kannada** — not recoverable; model selection issue
- **Catastrophic failure on whispered / very low SNR audio** — out of scope, all providers fail

## 6. Recommendation

State a concrete decision, not a comparison. Address the constraints reviewers will probe.

> **Ship:**
> - **Deepgram Nova-3 (streaming)** for live phone IVR. Best latency, strong on common localities, code-switching robust.
> - **Sarvam Saaras (batch)** for async WhatsApp voice notes. Marginally better on hard locality names, no streaming constraint.
> - **A phonetic post-processor** on both. ~30 lines of Levenshtein-based locality matching against a known Bangalore gazetteer. Recovers ~X% of strict-LRA misses.
>
> **Don't ship:**
> - Whisper via Groq in production — hallucinates English on Hindi/Kannada (Hypothesis 4 was confirmed); rate limits will bite at Growth scale.
>
> **Revisit when:**
> - Volume exceeds ~200k min/mo → self-hosting IndicConformer becomes break-even on TCO
> - We expand to Tamil/Telugu/Marathi → re-benchmark; Indian-language coverage varies by provider
> - We see >5% of candidates speaking pure regional languages → switch primary to Sarvam or finetune Whisper

## 7. Limitations (be honest — reviewers respect this)

- **N=20 is tiny.** Confidence intervals on any single metric are wide. Treat numbers as directional, not definitive.
- **Single speaker.** All variance is within-speaker. Cross-speaker variance is the bigger production risk; not measured.
- **One Kannada and one pure-Hindi sample each.** Insufficient to draw conclusions on monolingual performance.
- **FLEURS is read speech, not conversational.** The cross-check answers "does this model work on clean Hindi?" but not "does it generalize to phone-quality conversational Hindi at scale." Kathbath would have been a better dataset for the latter but didn't fit the time budget.
- **No code-switched dataset evaluation.** MUCS 2021 would have tested the Hindi-English mixing that's central to my use case; skipped due to data acquisition friction.
- **No streaming latency.** Measured end-to-end batch latency only; production IVR uses streaming, which behaves differently.
- **Cost estimates** assume published list prices. Real enterprise contracts vary ±50%.
- **No real telephony codec test.** Recorded phone calls approximate but don't perfectly match real PSTN/VoIP codec degradation.
- **No A/B with phonetic post-processor.** I claim it would help ~X%; that needs to be implemented and measured before the recommendation can ship.

## 8. What I'd do with one more day (and one more week)

**One more day:**
- Implement the phonetic post-processor and measure the actual lift (claimed ~X% recovery is unverified)
- Add IndicConformer (self-hosted) to validate the cost-at-scale claim — currently extrapolated, not measured
- Get 2-3 other speakers (different gender / age / accent) to record the same 20 prompts — fixes the single-speaker limitation

**One more week:**
- **Add GramVaani** — this is the highest-value dataset upgrade. Its rural-caller telephony Hindi is the closest public proxy for our actual deployment. Would replace the recorded-phone-call rows with real production-grade audio
- **Add Svarah** — tests Indian-accented English on the English-named localities (Whitefield, Electronic City, Silk Board, HSR Layout, BTM Layout, Majestic) which are 30% of the test set
- **Add MUCS 2021** — only public dataset that explicitly benchmarks Hindi-English code-switching, which is the core skill our use case demands
- **Generate a TTS-synthesized test set** — use IndicTTS to produce 10+ voices × 30 localities × 3 conditions = 900 entity-heavy utterances. Solves the N=20 problem in one afternoon
- Test streaming latency against batch latency for Deepgram and Sarvam — only streaming matters for live IVR, but we measured batch
