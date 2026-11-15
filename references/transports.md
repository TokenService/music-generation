# 通道与接口

## 接口

提交：

```
POST /suno/submit/music
{ "code": 200, "success": true, "data": { "task_id": "task_xxxxxxxxxxxxxxxx", "status": "processing" } }
```

`data` 可能是裸的 task id 字符串，也可能是带 `task_id` 的对象，两种都能读。

上传：

```
POST /suno/submit/uploads
{ "code": "success", "message": "", "data": "task_xxxxxxxxxxxxxxxx" }
```

轮询：

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

状态是 `QUEUED`、`IN_PROGRESS`、`SUCCESS`、`FAILURE`。音频地址在这个网关叫 `audio_url`，Suno 兼容实现叫 `cld2AudioUrl`；歌词在 `text`，有的实现用 `lyric` 或 `lyrics`。上传任务返回单个片段，生成任务返回数组。

命令行按字段名和文件后缀找音频地址，字段改名也能认。

## chat 通道

```
POST /v1/chat/completions
{ "model": "suno-v6-mini", "messages": [{ "role": "user", "content": "..." }], "stream": false }
```

- Suno 模型会把 `--title`、`--tags`、`--make-instrumental` 并进 prompt。
- 翻唱和续写用顶层 `cover_clip_id`、`continue_clip_id`、`continue_at`。
- 音频可能是 `data[]` 里的地址、`message.content` 里的 Markdown 链接，或 `message.audio` 的 base64，三种都处理，文件写进 `outputs/music/`。
