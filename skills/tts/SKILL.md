---
name: tts
description: >
  Text-to-speech skill using an OpenAI-compatible speech API. Use when the user explicitly requests TTS tasks
  such as: text-to-speech, generate audio, read text aloud, convert text to audio, TTS, 语音合成,
  or any request to turn English text into spoken audio.
  Do NOT trigger for general questions about speech or audio concepts.
---

# OpenAI-Compatible TTS

Generate spoken audio from English text using an OpenAI-compatible speech API (Kokoro).

## Prerequisites

Both environment variables **must** be set before proceeding. If either is missing or empty, inform the user and stop:

- `OPENAI_SPEECH_API_BASE_URL` — base URL of the speech API (e.g. `http://localhost:11000`)
- `OPENAI_SPEECH_API_MODEL` — model name (e.g. `kokoro`)

> "OPENAI_SPEECH_API_BASE_URL and OPENAI_SPEECH_API_MODEL must both be set. Please configure them and try again."

Do NOT attempt any API calls without these variables.

## Text Preprocessing

The inline python3 command automatically applies these replacements before sending:

- `CELPIP` → `Celpip` (proper case for correct pronunciation)


## Workflow

### Step 1: Generate and save audio

Run this single command. Replace `USER_TEXT` with the user's text and `OUTPUT_PATH` with the desired output file:

```bash
mkdir -p ./tts-downloads && python3 -c "
import json,sys,os,urllib.request
t=sys.stdin.read().replace('CELPIP','Celpip')
body=json.dumps({'model':os.environ['OPENAI_SPEECH_API_MODEL'],'input':t,'voice':'af_heart+af_bella','response_format':'mp3','stream':False}).encode()
req=urllib.request.Request(os.environ['OPENAI_SPEECH_API_BASE_URL']+'/v1/audio/speech',data=body,headers={'Content-Type':'application/json'})
r=urllib.request.urlopen(req)
with open(sys.argv[1],'wb') as f:f.write(r.read())
print('Saved:',sys.argv[1])
" "./tts-downloads/tts_$(date +%Y%m%d_%H%M%S).mp3" <<< "USER_TEXT"
```

- If the user specifies a different output path or format (e.g. wav), adjust the filename and add `'response_format':'wav'` to the json dict accordingly.
- If the user requests a different speed, add `'speed':0.5` (range 0.25–4.0).

### Step 2: Verify

After download, verify the file exists and is non-empty:

```bash
ls -lh ./tts-downloads/tts_*.mp3 | tail -1
```

Report the full file path to the user.

## API Reference

| Field | Default | Options |
|---|---|---|
| `model` | (from env) | `kokoro`, `tts-1`, `tts-1-hd` |
| `voice` | `af_heart+af_bella` | Fixed — do not change unless user requests |
| `response_format` | `mp3` | `mp3`, `wav`, `flac`, `opus`, `pcm` |
| `speed` | `1.0` | `0.25` – `4.0` |
| `stream` | `false` | Keep `false` for file output |

## Error Handling

| Scenario | Action |
|---|---|
| `OPENAI_SPEECH_API_BASE_URL` not set | Tell user to set it, stop |
| `OPENAI_SPEECH_API_MODEL` not set | Tell user to set it, stop |
| Connection refused / timeout | Tell user the speech API server may not be running, stop |
| Empty or very small output file | Likely an API error — check file contents for error JSON |

## Examples

### Basic TTS

```
User: "Convert this to speech: The quick brown fox jumps over the lazy dog"

→ mkdir -p ./tts-downloads && python3 -c "..." "./tts-downloads/tts_20260424_143100.mp3" <<< "The quick brown fox jumps over the lazy dog"
→ "Saved: ./tts-downloads/tts_20260424_143100.mp3"
```

### Multi-line text with CELPIP

```
User: "Read this aloud: Welcome to the CELPIP test. It has three parts."

→ (same command, TEXT becomes: "Welcome to the Celpip test. It has three parts.")
→ "Saved: ./tts-downloads/tts_20260424_143200.mp3"
```

### Custom format and speed

```
User: "Read this slowly as WAV: Hello world"

→ (add 'response_format':'wav','speed':0.5 to json dict, use .wav extension)
→ "Saved: ./tts-downloads/tts_20260424_143300.wav"
```
