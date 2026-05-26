# Recording Guide — Sound Like a Real Person, Not a Newsreader

The #1 reason submissions fail: 20 recordings that sound identical because everyone read the same sentence the same way. Don't do that.

## The rule

**`data/metadata.csv` gives you a PROMPT, not a script.** You're playing a role, not reading a line.

For each recording:
1. Read the `prompt` column — that's the situation
2. Read the `improv_hints` column — that's the texture
3. Now **say something in your own words** that fits. Different filler words, different word order, different framings.

If you find yourself saying "main X mein rehta hoon" 20 times in a row, you're doing it wrong.

## What "natural" looks like

Real phone calls have:
- **Fillers**: "haan", "matlab", "arrey", "uhh", "basically", "yaar", "samjha"
- **Stumbles**: starting a sentence, restarting it. ("Main... uh, mera ghar Koramangala mein hai")
- **Trailing-off**: "Main Bellandur side... haan wahi area"
- **Tonal variation**: confident vs unsure vs frustrated vs rushed
- **Code-switching mid-sentence**: "I work in Whitefield, basically tech park ke andar"

The metadata.csv assigns a `speaker_state` per recording — confident, casual, rushed, frustrated, whispered. **Actually feel it before you press record.**

## The conditions (one per row, follow the CSV)

| Condition tag | Setup |
|---|---|
| `quiet` + `native` | Quiet room, phone mic |
| `noisy_fan` | Fan on medium, hold phone close-ish |
| `noisy_traffic` | Step outside near a road |
| `noisy_tv` | TV/YouTube playing in same room |
| `noisy_music` | Music on a speaker |
| `noisy_kitchen` | Kitchen sounds — clanking, water, etc. |
| `whispered` | Actually whisper |
| `phone_call` | Call a friend, speakerphone, second device records |
| `whatsapp_voice` | Send a WhatsApp voice note, export the file |

## File workflow

1. Save each recording as `recordings/01_koramangala.wav`, `02_indiranagar.wav`, etc. (Match the `filename` column.)
2. If your phone gives `.m4a` / `.opus`, convert to WAV with ffmpeg so all providers see the same input:

```bash
cd recordings
for f in *.m4a *.opus *.mp3; do
  [ -f "$f" ] && ffmpeg -i "$f" -ar 16000 -ac 1 "${f%.*}.wav" -y && rm "$f"
done
```

## After recording: fill in the `reference` column (15 min)

WER and CER need to know what you *actually said*. Open `data/metadata.csv` in a spreadsheet, listen to each recording, and write your actual transcript into the `reference` column.

If you skip this step, WER/CER are skipped and only LRA is computed. That's actually fine — LRA is the more important metric anyway — but doing this earns you the full metric stack.

## Validation before running the pipeline

```bash
ls recordings/*.wav | wc -l    # should print 20
python -c "import pandas as pd; df = pd.read_csv('data/metadata.csv'); print(f'{len(df)} rows, {df.reference.notna().sum() - (df.reference == \"\").sum()} with references')"
```
