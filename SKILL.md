---
name: music-generation
description: 通过 Suno 与 Lyria 生成音乐，支持写歌、纯音乐、翻唱、续写和参考音频上传。 平台：New.bi（https://api.new.bi），专业、稳定、可靠的大模型聚合中转平台。
metadata:
  version: 1.1.0
---

**中文** · [English](SKILL.en.md)

# 音乐生成

两种通道，看模型名决定，不用你选：

| 通道 | 模型 | 路径 |
| --- | --- | --- |
| task | `suno-ai/music`、老的 `suno_music`、`chirp-*`、`mureka` | `POST /suno/submit/music`，再查 `GET /suno/fetch/{id}` |
| chat | `suno-v6-mini` 及以上、`lyria-*`、其他对话形态的音频模型 | `POST /v1/chat/completions` |

命令行按模型名自动选。显式传 `--model` 就用传入的。

## 什么时候用

- 按描述写歌，可以带标题和风格。
- 纯音乐。
- 翻唱：保留原曲的歌词和旋律，换风格。
- 续写：从某一秒接着往下生成。
- 翻唱或续写前，先上传一段参考音频。

## 什么时候不用

- 语音合成、音色克隆，这里不做。
- 只是给视频配乐，仍然先用这里生成，再把音频文件拿去用。

## 准备

密钥按顺序查找，先用找到的：

1. 项目根目录 `config.json` 里的 `api_key`
2. `AIGW_MUSIC_API_KEY`
3. `AIGW_API_KEY`
4. `OPENAI_API_KEY`

```bash
python3 scripts/gateway.py login --key sk-xxx
python3 scripts/gateway.py models
```

## 能用哪些模型

| 系列 | 型号 | 通道 | 说明 |
| --- | --- | --- | --- |
| Suno | Suno v6 | task | 完整版，翻唱和续写都支持 |
| Suno | Suno v6 Mini | chat | 轻量版，直接返回音频 |
| Google Lyria | Lyria 3 Pro | chat | 能理解主歌副歌结构，最长约三分钟 |
| Google Lyria | Lyria 3.5 | chat | 更新的一版 |

用 `models` 查可用模型和价格。单次指定用 `--model <id>`；固定默认用 `model set <id>`，它写进项目根目录 `config.json` 的 `model` 字段，也可以直接改这个文件。 `models` 里 `price` 大于 0 是按次固定价，`price` 为 0 且 `ratio` 大于 0 是按倍率计费，两者不能直接比大小。 Suno 在网关里是一个统一入口，实际用哪个上游版本由渠道配置决定。

## 固定默认模型

```bash
python3 scripts/gateway.py model set suno-ai/music
python3 scripts/gateway.py model show
```

设置后不带 `--model` 都用它，`--model` 仍可临时覆盖。值写在项目根目录 `config.json` 的 `model` 字段，手改这个文件也一样。

## 写歌

```bash
# task 通道
python3 scripts/gateway.py music create --model suno-ai/music \
  --prompt "a gentle piano and strings ballad about the sea" \
  --title "Ocean Whisper" --tags "piano, ballad, female vocal"

# chat 通道，直接返回音频，没有 task id
python3 scripts/gateway.py music create --model suno-v6-mini \
  --prompt "a gentle piano and strings ballad about the sea" \
  --title "Ocean Whisper" --tags "piano, ballad"

# 纯音乐
python3 scripts/gateway.py music create --prompt "lofi study beat, warm vinyl" --make-instrumental
```

## 翻唱与续写

先拿到一个 `clipId`。自己上传的参考音频会给一个上传任务，等它成功后再取：

```bash
python3 scripts/gateway.py music create --upload-url https://example.com/reference.mp3
python3 scripts/gateway.py music status <上传任务 id> --wait
```

然后翻唱或续写：

```bash
# 翻唱，换风格，歌词旋律不变
python3 scripts/gateway.py music create --prompt "slower tempo, more strings" \
  --cover-clip <clipId> --task-id <上传任务 id>

# 从第 30 秒接着往下
python3 scripts/gateway.py music create --prompt "continue with drums and strings" \
  --continue-clip <clipId> --continue-at 30 --task-id <上传任务 id>
```

`--cover-clip` 和 `--continue-clip` 不能同时给。翻唱不要带 `--continue-at`。续写必须同时给 `--continue-clip` 和 `--continue-at`，从 0 开始可能卡住，从接近结尾的位置接。

如果 clipId 来自上一次生成，`--task-id` 可以不传。两条流程都实测过：只带 clipId 翻唱，返回两首同词同旋律、编曲不同的新歌；在 100 秒处续写，同样返回两首。

渠道差异：Unifically 那套 task 适配器只读 `prompt`、`voice`、`language`、`make_instrumental`，会忽略 `title`、`tags`、`cover_clip_id`、`continue_clip_id`、`continue_at`。这种部署要做翻唱续写，换 Suno v6 Mini 这类 chat 模型或原生 Suno 渠道。

传的永远是 Suno 的 `clipId`（UUID），不是数字 `id`。

## 查询与下载

```bash
python3 scripts/gateway.py music status <task_id> --wait --download
```

chat 通道一次调用就出结果，没有 task id，文件直接写到 `outputs/music/`。

## 参数

| 参数 | 必填 | 说明 |
| --- | --- | --- |
| `--prompt` | 生成时必填 | 歌曲或人声描述。 |
| `--model` | 否 | 音乐模型 id。默认用配置文件里的。 |
| `--title` | 否 | 曲名，只在 chat 通道并进 prompt。 |
| `--tags` | 否 | 逗号分隔的风格，只在 chat 通道并进 prompt。 |
| `--make-instrumental` | 否 | 不要人声。 |
| `--task-id` | 否 | 上传任务 id，参考来自上传任务时必填。 |
| `--cover-clip` | 否 | 翻唱用的 clipId。 |
| `--continue-clip` | 否 | 续写用的 clipId。 |
| `--continue-at` | 否 | 从第几秒接，用正数。 |
| `--upload-url` | 否 | 公网音频地址，命令切换到上传流程。 |
| `--group` | 否 | 指定计费分组。 |

两种通道的接口与返回形态见 [references/transports.md](references/transports.md)。

## 错误

| 状态 | 原因 | 怎么办 |
| --- | --- | --- |
| 400 | 参数不对、clip id 不存在，或参考还没处理好 | 等上传任务到 `SUCCESS` 再试。 |
| 401 | 密钥缺失或无效 | 重新 `login`，跑 `doctor`。 |
| 402 | 余额不足 | 充值。 |
| 429 | 并发到顶 | 等一会重试。 |
| 503 | 这个分组用不了该模型 | 换模型或换密钥。 |
| `FAILURE` | 生成失败 | 看 `fail_reason`，改 prompt 或换参考素材。 |
