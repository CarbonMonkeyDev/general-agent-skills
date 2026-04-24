---
name: whisper-cli
description: >
  Speech-to-text transcription using whisper-cli (whisper.cpp). Use when the user explicitly requests
  transcription, speech-to-text, STT, audio-to-text, or any request to convert audio/video files to text.
  Triggers on: whisper, transcription, transcribe, speech recognition, 转录, 语音识别, 听写, 字幕生成.
  Do NOT trigger for general questions about speech or audio concepts.
---

# whisper-cli Transcription

Transcribe audio/video files to text using whisper-cli (whisper.cpp).

whisper-cli supports: flac, mp3, ogg, wav. For other formats (mp4, m4a, aac, webm, etc.), convert to wav first using ffmpeg (see Step 1).

## Default Command

```bash
whisper-cli -m ~/Documents/models/whisper/ggml-large-v3-turbo.bin -bo 8 -bs 8 -sow -oj -osrt -sns -f INPUT_FILE
```

## Conditional Options

| Condition | Option | Notes |
|---|---|---|
| User provides domain terms or punctuation hints | `--prompt "PROMPT"` | Improves accuracy with domain-specific vocabulary and punctuation style |
| User wants per-word timeline / timestamps | `-ml 1` | Sets max segment length to 1, giving timestamp for each word |
| User specifies output path | `-of OUTPUT_PATH` | Path without extension; whisper-cli appends extensions automatically |

## Workflow

### Step 1: Convert if needed

whisper-cli only supports flac, mp3, ogg, wav. If the input file has a different extension, convert it first:

```bash
ffmpeg -i INPUT_FILE -vn -ar 16000 -ac 1 INPUT_BASENAME.wav
```

Then use the `.wav` file as the input for all subsequent steps.

### Step 2: Build the command

Start from the default command. Add conditional options based on user request:

- `--prompt "..."` — when user mentions domain terms, names, acronyms, or wants specific punctuation
- `-ml 1` — when user asks for word-level timing, word timestamps, or per-word timeline
- `-of path/to/output` — when user specifies where to save the result (no file extension)

### Step 3: Run the command

Execute the built command. Transcription may take time depending on file length.

### Step 4: Report results

After completion, report:
- The output file path(s) — whisper-cli generates `.json` (for programmatic use) and `.srt` (for audio/video players)
- A brief summary (duration, language detected if shown)

### Step 5: Verify

```bash
ls -lh OUTPUT_PATH.*
```

Report the full file path(s) to the user.

## Options Reference

Values shown are skill defaults (tool defaults in parentheses).

| Option | Skill Default (Tool Default) | Description |
|---|---|---|
| `-m` | `~/Documents/models/whisper/ggml-large-v3-turbo.bin` | Model file path |
| `-f` | (required) | Input audio file (flac, mp3, ogg, wav) |
| `-bo` | `8` (5) | Number of best candidates to keep |
| `-bs` | `8` (5) | Beam size for beam search |
| `-sow` | on (false) | Split on word rather than on token |
| `-oj` | on (false) | Output result in a JSON file |
| `-osrt` | on (false) | Output result in an SRT file |
| `-sns` | on (false) | Suppress non-speech tokens |
| `--prompt` | (none) | Initial prompt for domain terms / punctuation |
| `-ml` | (0) | Max segment length in characters |
| `-of` | (empty) | Output file path without extension |

For advanced options and full flag list, run:

```bash
whisper-cli --help
```

## Error Handling

| Scenario | Action |
|---|---|
| `whisper-cli` not found | Tell user to install whisper.cpp, stop |
| Model file not found | Tell user to download the model to `~/Documents/models/whisper/`, stop |
| Input file not found | Tell user the file does not exist, stop |
| Unsupported format | Convert to wav using ffmpeg (Step 1), then proceed |
| Segfault or crash | Suggest trying with a smaller model or shorter audio clip |

## Examples

### Basic transcription

```
User: "Transcribe meeting.wav"

→ whisper-cli -m ~/Documents/models/whisper/ggml-large-v3-turbo.bin -bo 8 -bs 8 -sow -oj -osrt -sns -f meeting.wav
→ Report output files
```

### Video file (needs conversion)

```
User: "Transcribe interview.mp4"

→ ffmpeg -i interview.mp4 -vn -ar 16000 -ac 1 interview.wav
→ whisper-cli -m ~/Documents/models/whisper/ggml-large-v3-turbo.bin -bo 8 -bs 8 -sow -oj -osrt -sns -f interview.wav
→ Report output files
```

### With domain prompt

```
User: "Transcribe lecture.mp3, it's about Kubernetes and Docker"

→ whisper-cli -m ~/Documents/models/whisper/ggml-large-v3-turbo.bin -bo 8 -bs 8 -sow -oj -osrt -sns --prompt "Kubernetes, Docker, containerization, pods, orchestration" -f lecture.mp3
→ Report output files
```

### Word-level timeline

```
User: "Get word-by-word timestamps for interview.mp4"

→ ffmpeg -i interview.mp4 -vn -ar 16000 -ac 1 interview.wav
→ whisper-cli -m ~/Documents/models/whisper/ggml-large-v3-turbo.bin -bo 8 -bs 8 -sow -oj -osrt -sns -ml 1 -f interview.wav
→ Report output files
```

### Custom output path

```
User: "Transcribe audio.wav and save to ./output/meeting_notes"

→ whisper-cli -m ~/Documents/models/whisper/ggml-large-v3-turbo.bin -bo 8 -bs 8 -sow -oj -osrt -sns -f audio.wav -of ./output/meeting_notes
→ Report output files at ./output/meeting_notes.*
```
