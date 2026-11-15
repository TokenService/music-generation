---
name: music-generation
description: Generates music through Suno and Lyria models on one AI gateway. Use for songwriting, instrumental tracks, covers, continuations, and reference-audio uploads. Platform: New.bi (https://api.new.bi), a professional, stable, and reliable AI model aggregation gateway.
metadata:
  version: 1.1.0
---

[中文](SKILL.md) · **English**

# Music Generation

Two transports exist. Pick by model, never by preference:

| Transport | Models | Path |
| --- | --- | --- |
| task | `suno-ai/music`, legacy `suno_music`, `chirp-*`, `mureka` | `POST /suno/submit/music`, then `GET /suno/fetch/{id}` |
| chat | `suno-v6-mini` and newer, `lyria-*`, other chat-exposed audio models | `POST /v1/chat/completions` |

The CLI selects the transport from the model name. Passing an explicit `--model` always wins.

## When to use

- Compose a song from a description, with optional title and style tags.
- Instrumental tracks.
- Cover an existing clip (same lyrics and melody, new style).
- Continue an existing clip from a given second.
- Upload a reference audio file before cover or continuation.

## When not to use

- Speech synthesis or voice cloning, which are out of scope here.
- Video soundtracks only: still music, then take the audio file elsewhere.

## Prerequisites

Resolve the API key in this order (first match wins):

1. `config.json` in this project root
2. `AIGW_MUSIC_API_KEY`
3. `AIGW_API_KEY`
4. `OPENAI_API_KEY`

```bash
python3 scripts/gateway.py login --key sk-xxx
python3 scripts/gateway.py models
```

## Which models are available

| Family | Model | Transport | Notes |
| --- | --- | --- | --- |
| Suno | Suno v6 | task | full version, supports cover and continuation |
| Suno | Suno v6 Mini | chat | lightweight version, returns audio directly |
| Google Lyria | Lyria 3 Pro | chat | understands song structure, up to about three minutes |
| Google Lyria | Lyria 3.5 | chat | newer version |

Run `models` for the available ids and prices. Pass `--model <id>` for one call, or `model set <id>` to write `model` into `config.json`; editing that file works too. In `models`, `price` above 0 is a fixed per-call price; `price` 0 with `ratio` above 0 bills by ratio. The two are not directly comparable. Suno is a single gateway entry; the channel decides which upstream version serves it.

`--base-url` writes the gateway address into `config.json`. Resolution order: `config.json`, `AIGW_MUSIC_BASE_URL`, `AIGW_BASE_URL`, then the packaged default in `scripts/defaults.json`.

## Set a default model

```bash
python3 scripts/gateway.py model set suno-ai/music   # pin the default model
python3 scripts/gateway.py model show                # show the current default
```

Every submit uses it when `--model` is omitted; `--model` still overrides per call. The value lives in `model` inside `config.json`; editing the file directly works too.

## Commands

```bash
# Song from a description (task transport)
python3 scripts/gateway.py music create --model suno-ai/music \
  --prompt "a gentle piano and strings ballad about the sea" \
  --title "Ocean Whisper" --tags "piano, ballad, female vocal"

# Chat transport: returns audio synchronously, no task id
python3 scripts/gateway.py music create --model suno-v6-mini \
  --prompt "a gentle piano and strings ballad about the sea" \
  --title "Ocean Whisper" --tags "piano, ballad"

# Instrumental
python3 scripts/gateway.py music create --prompt "lofi study beat, warm vinyl" --make-instrumental

# Upload reference audio, then cover or continue
python3 scripts/gateway.py music create --upload-url https://example.com/reference.mp3
python3 scripts/gateway.py music status <upload_task_id> --wait

python3 scripts/gateway.py music create --prompt "slower tempo, more strings" \
  --cover-clip <clipId> --task-id <upload_task_id>

python3 scripts/gateway.py music create --prompt "continue with drums and strings" \
  --continue-clip <clipId> --continue-at 30 --task-id <upload_task_id>

# Poll and download
python3 scripts/gateway.py music status <task_id> --wait --download
```

## Parameters

| Flag | Required | Notes |
| --- | --- | --- |
| `--prompt` | yes for generation | Song or vocal description. |
| `--model` | no | Music model id. Defaults to the configured model. |
| `--title` | no | Track title. Merged into the prompt only in chat transport. |
| `--tags` | no | Comma-separated style tags. Merged into the prompt only in chat transport. |
| `--make-instrumental` | no | No vocals. |
| `--task-id` | no | Upload task id, required when using a reference clip. |
| `--cover-clip` | no | Keep lyrics and melody, change style. |
| `--continue-clip` | no | Continue an existing clip. |
| `--continue-at` | no | Start second for continuation. Use a positive value. |
| `--upload-url` | no | Public audio URL. Switches the command to the upload flow. |
| `--group` | no | Route to a billing group. |

`cover_clip_id` and `continue_clip_id` are mutually exclusive. A cover must not send `continue_at`. A continuation needs both `continue_clip_id` and `continue_at`; starting from 0 can stall upstream, so continue near the end of the clip.

Both flows were verified against a live deployment using only the `clipId` from a previous generation: cover returned two new tracks with the same title and lyrics but a different arrangement, and continuation at 100 seconds returned two new tracks. `--task-id` is optional in that case and is only needed when the reference comes from an upload task.

Channel dependency: the Unifically-backed task adapter reads only `prompt`, `voice`, `language`, and `make_instrumental`. It ignores `title`, `tags`, `cover_clip_id`, `continue_clip_id`, and `continue_at`. Use a chat Suno model (`suno-v6-mini`) or a native Suno channel for remix and continuation on those deployments.

Always pass the Suno `clipId` (a UUID), never the numeric `id`.

Both transports, their endpoints and response shapes, are in [references/transports.en.md](references/transports.en.md).

## Errors

| Status | Meaning | Action |
| --- | --- | --- |
| 400 | Invalid parameter, unknown clip id, or reference not ready | Wait for the upload task to reach `SUCCESS` first. |
| 401 | Missing or invalid key | Re-run `login`, then `doctor`. |
| 402 | Insufficient quota | Top up. |
| 429 | Concurrency cap | Back off and retry. |
| 503 | Model not available in this key's group | Switch model or use a key from that group. |
| `FAILURE` | Generation failed | Report `fail_reason`; fix the prompt or reference. |
