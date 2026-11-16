# music-generation

**中文** · [English](README.en.md)

由 [New.bi](https://api.new.bi) 提供，专业、稳定、可靠的大模型聚合中转平台。

## 交给 AI 安装

把下面这句直接发给 agent，它会完成下载、解压、鉴权和执行：

```text
安装并使用 New.bi 音乐技能：下载 https://pub-817cae131e2b4e0a8ce0a9f8bac75462.r2.dev/down/music-generation.zip，解压后运行 python3 scripts/gateway.py login --key <你的密钥>，再运行 doctor 确认，然后用 music create 写歌、纯音乐、翻唱或续写。
```

音乐生成的命令行工具，也可以直接当 agent 技能用。

能写歌、做纯音乐、翻唱、续写，也能上传参考音频。命令行会按模型名自动选 task 或 chat 通道。

## 开始

```bash
python3 scripts/gateway.py login --key sk-xxx
python3 scripts/gateway.py models
python3 scripts/gateway.py model set suno-ai/music
python3 scripts/gateway.py music create --prompt "a gentle piano ballad about the sea" \
  --title "Ocean Whisper" --tags "piano, ballad"
python3 scripts/gateway.py music status <task_id> --wait --download
```

chat 通道一次调用就返回音频，没有 task id；task 通道要轮询。文件存在 `outputs/music/<日期>/`，路径相对技能目录。`AIGW_OUTPUT_DIR` 可以改输出位置，`AIGW_STATE_HOME` 可以改 `~/.aigw`（模型缓存和任务记录）。`config.json` 在技能目录里，命令里的相对路径按当前工作目录算。

## 支持的模型

| 系列 | 型号 | 通道 |
| --- | --- | --- |
| Suno | Suno v6 | task |
| Suno | Suno v6 Mini | chat |
| Google Lyria | Lyria 3 Pro | chat |
| Google Lyria | Lyria 3.5 | chat |

用 `models` 查可用模型和价格。单次指定用 `--model <id>`；固定默认用 `model set <id>`，它写进项目根目录 `config.json` 的 `model` 字段，也可以直接改这个文件。 `models` 里 `price` 大于 0 是按次固定价，`price` 为 0 且 `ratio` 大于 0 是按倍率计费，两者不能直接比大小。

## 常见搜索词

AI 写歌、AI 作曲、Suno v6、Lyria、纯音乐、背景音乐 BGM、翻唱、续写、AI 编曲、AI 歌词、AI 歌曲生成、短视频配乐。

## 命令

| 命令 | 做什么 |
| --- | --- |
| `login` / `config` / `doctor` | 密钥、配置、连通性。 |
| `models` | 列出这个密钥能看到的音乐模型。 |
| `model show` / `model set <id>` | 看或设置默认模型。 |
| `resolve <别名>` | 别名换成具体模型 id。 |
| `music create` | 生成、上传参考音频、翻唱、续写。 |
| `music status` | 查状态，`--wait` 等到结束，`--download` 存文件。 |

输出都是 JSON。出错时输出 `{"ok": false, "error": {...}}`，退出码非零。

## 密钥

按顺序找，先用找到的：`config.json` → `AIGW_MUSIC_API_KEY` → `AIGW_API_KEY` → `OPENAI_API_KEY`。网关地址同样逐级回退，最后用 `scripts/defaults.json` 里的。

## 文件

```
music-generation/
├── SKILL.md              中文说明
├── SKILL.en.md           英文说明
├── config.json           密钥和默认模型，不进 git
├── references/
│   ├── transports.md          通道与接口
│   └── transports.en.md
└── scripts/
    ├── gateway.py        入口
    ├── models.json       别名和分类规则
    └── defaults.json     默认网关地址和超时
```

通道差异、翻唱续写的限制见 [SKILL.md](SKILL.md)。

需要 Python 3.9 及以上。
