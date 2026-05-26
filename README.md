# ASR — Assignment 

Benchmarks Deepgram (baseline) against Sarvam AI and Whisper-large-v3 (via Groq) on 20 self-recorded utterances of Bangalore locality names.

# Setup

pip install -r requirements.txt
cp .env.example .env # add your keys

# Audio: record per recording_guide.md OR generate TTS placeholders:

python scripts/generate_recordings.py

# Record audio (see recording_guide.md), drop files in recordings/

# Verify setup before burning API credits

python scripts/00_smoke_test.py

## Main pipeline

- python scripts/01_transcribe.py # ~5-10 min, ~60 API calls
- python scripts/02_metrics.py # headline tables
- python scripts/03_failures.py # every locality miss
- python scripts/05_substitutions.py # what specifically went wrong
- python scripts/04_cost_analysis.py # production cost model

# Optional (adds ~15 min) — generalization check on FLEURS Hindi

-pip install datasets scipy
-python scripts/06_fleurs_benchmark.py
-python scripts/07_realworld_delta.py # computes the "real-world penalty"

## API keys (15 min, free tier on all three)

- **Deepgram** — https://console.deepgram.com — $200 free credit, way more than enough
- **Sarvam** — https://dashboard.sarvam.ai — free credits on signup
- **Groq** (Whisper) — https://console.groq.com — free tier with generous rate limits

## Files you'll actually open

-`data/metadata.csv` The designed experiment — 20 prompts × conditions 
-`recording_guide.md` How to record so the samples don't all sound the same 
-`HYPOTHESES.md` Pre-registered predictions — falsifying one is where your "surprise" comes from
-`REPORT_TEMPLATE.md` he opinionated report shape. Fill in numbers, submit this.

## What we measure and why

WER / CER , Standard for context; weights every word equally — not ideal for entity extraction 
**LRA-strict**
Did the exact locality name appear? Binary. What the product cares about.  
**LRA-fuzzy**
Same with Levenshtein tolerance — what a phonetic post-processor could recover
Latency  
Approximates production latency for batch APIs  
Cost at scale  
Modeled at MVP / Growth / Scale volumes 

The headline insight typically lives in the gap between WER and LRA-fuzzy: standard ASR benchmarks can recommend the wrong model for an entity-extraction use case.


