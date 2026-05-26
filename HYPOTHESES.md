# Hypotheses — Where to Look for the Surprise

The "insight that surprised you" criterion is the highest-leverage thing in the rubric. The reliable way to find one: **go in with pre-registered predictions, then check which ones broke.** Whichever hypothesis your data falsifies becomes your headline insight.

Before running the pipeline, write down what you expect. After running, see which ones are wrong. That gap is your story.

## Hypotheses to test

**H1 — Indian-specialized models beat generalist models on long Kannada names.**
> *Prediction:* Sarvam > Deepgram > Whisper on `difficulty=hard` localities.
> *How to check:* `scripts/03_failures.py` → "By difficulty" table.
> *If reversed:* Big insight — generalist models have caught up on Indian-specific vocab. Surprising.

**H2 — WER and LRA rank the providers identically.**
> *Prediction:* The provider with the lowest WER also has the highest LRA-fuzzy.
> *How to check:* Compare ranks across columns in the summary table.
> *If reversed:* This is gold. It means standard ASR benchmarks would have led you to ship the wrong model for an entity-extraction use case.

**H3 — Phone-channel audio degrades all providers equally.**
> *Prediction:* Every provider's LRA drops by similar % on `channel=phone_call`.
> *How to check:* `scripts/02_metrics.py` → "By condition" table, look at the phone_call rows.
> *If reversed:* Some provider is doing telephony-specific processing — operationally meaningful for IVR.

**H4 — Whisper hallucinates English on pure Hindi/Kannada input.**
> *Prediction:* Whisper's transcripts of rows 19–20 contain English words that weren't said.
> *How to check:* Read transcripts.json for filenames `19_*` and `20_*` — look for English drift.
> *If confirmed:* Strong signal not to use Whisper for non-Hinglish content.

**H5 — Fuzzy matching rescues 20%+ of strict misses.**
> *Prediction:* `LRA-fuzzy - LRA-strict` ≈ 20 percentage points.
> *How to check:* Top summary table.
> *If confirmed:* A phonetic post-processor is the single highest-ROI fix the team can make — independent of which model they pick.

**H6 — All providers fail on the same localities.**
> *Prediction:* The failure set is heavily overlapping (mostly long compound names).
> *How to check:* `scripts/03_failures.py` → "Hit rate by locality" table.
> *If reversed:* Provider-specific failure modes exist; suggests an ensemble or fallback strategy.

## How to write the surprise in the report

Once you find the broken hypothesis, frame it like this:

> "Going in I expected [X]. The data shows [Y]. The implication for the product is [Z]."

That's the shape. Three sentences. Don't bury it — make it the headline.
