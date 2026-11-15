# Transports and endpoints

## Endpoints and response shapes

Task submit:

```
POST /suno/submit/music
{ "code": 200, "success": true, "data": { "task_id": "task_xxxxxxxxxxxxxxxx", "status": "processing" } }
```

Suno-compatible gateways return `data` as a bare task id string or an object with `task_id`; the CLI accepts both.

Upload:

```
POST /suno/submit/uploads
{ "code": "success", "message": "", "data": "task_xxxxxxxxxxxxxxxx" }
```

Poll:

```
GET /suno/fetch/{task_id}
{
  "code": "success",
  "data": {
    "task_id": "task_xxxxxxxxxxxxxxxx",
    "status": "SUCCESS",
    "progress": "100%",
    "fail_reason": "",
    "data": [{ "clipId": "...", "title": "...", "duration": 194.36, "audio_url": "https://...", "text": "[Verse] ..." }]
  }
}
```

Task status is `QUEUED`, `IN_PROGRESS`, `SUCCESS`, or `FAILURE`. The generated audio URL is `audio_url` on this gateway and `cld2AudioUrl` on Suno-compatible ones; `text` (or `lyric` / `lyrics`) carries the lyrics. Upload tasks return a single clip object, generation tasks return an array.

The CLI extracts audio URLs by key and extension, so a renamed field still works.

## Chat transport

Chat models return audio synchronously from `POST /v1/chat/completions`:

```
POST /v1/chat/completions
{ "model": "suno-v6-mini", "messages": [{ "role": "user", "content": "..." }], "stream": false }
```

- `--title`, `--tags`, and `--make-instrumental` are merged into the prompt for Suno models.
- Top-level `cover_clip_id`, `continue_clip_id`, and `continue_at` are sent for remix and continuation.
- Audio arrives as a URL in `data[]`, a Markdown link in `message.content`, or base64 in `message.audio`. The CLI handles all three and writes files to `outputs/music/`.
- The command finishes in one call: no task id and no polling.
