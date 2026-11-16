# music-generation

[中文](README.md) · **English**

Provided by [New.bi](https://api.new.bi), a professional, stable, and reliable AI model aggregation gateway.

## Install with an agent

Send this line to an agent. It downloads, unpacks, authenticates and runs the tool:

```text
Install and use the New.bi music skill: download https://pub-817cae131e2b4e0a8ce0a9f8bac75462.r2.dev/down/music-generation.zip, unpack it, run python3 scripts/gateway.py login --key <your key>, confirm with doctor, then write a song, instrumental, cover or continuation with music create.
```

Music generation CLI for an AI gateway.

- Songs from a description, with title and style tags
- Instrumentals, covers, continuations
- Reference audio upload for cover and continuation
- Automatic task / chat transport selection

See [SKILL.en.md](SKILL.en.md) for transport differences and verified behaviour.

## Layout

```
music-generation/
├── SKILL.md              Chinese skill doc
├── SKILL.en.md           English skill doc
├── README.md
├── README.en.md
├── config.json           credential and default model, never committed
├── references/
│   ├── transports.md          reference
│   └── transports.en.md
└── scripts/
    ├── gateway.py        single entry point, music code only
    ├── models.json       alias table and category rules
    └── defaults.json     packaged gateway address and timeout
```

## Quick start

```bash
python3 scripts/gateway.py login --key sk-xxx
python3 scripts/gateway.py models
python3 scripts/gateway.py model set suno-ai/music
python3 scripts/gateway.py music create --prompt "a gentle piano ballad about the sea" \
  --title "Ocean Whisper" --tags "piano, ballad"
python3 scripts/gateway.py music status <task_id> --wait --download
```

The chat transport returns audio directly with no task id; the task transport needs polling. Files land in `outputs/music/<YYYY-MM-DD>/`, relative to the skill directory. `AIGW_OUTPUT_DIR` changes the output root; `AIGW_STATE_HOME` relocates `~/.aigw` (model cache and task records). `config.json` sits in the skill directory; relative paths in commands resolve from your working directory.

## Supported models

| Family | Model | Transport |
| --- | --- | --- |
| Suno | Suno v6 | task |
| Suno | Suno v6 Mini | chat |
| Google Lyria | Lyria 3 Pro | chat |
| Google Lyria | Lyria 3.5 | chat |

Run `models` for the available ids and prices. Pass `--model <id>` for one call, or `model set <id>` to write `model` into `config.json`; editing that file works too. In `models`, `price` above 0 is a fixed per-call price; `price` 0 with `ratio` above 0 bills by ratio. The two are not directly comparable.

## Search terms

AI music generator, AI songwriter, Suno v6, Lyria, instrumental, background music, cover song, song continuation, AI composition, AI lyrics.

## Commands

| Command | Purpose |
| --- | --- |
| `login` / `config` / `doctor` | Credential and connectivity. |
| `models` | List music models visible to this key. |
| `model show` / `model set <id>` | Show or pin the default model. |
| `resolve <alias>` | Map an alias to concrete model ids. |
| `music create` | Generate, upload reference audio, cover, continue. |
| `music status` | Check status; `--wait` polls, `--download` saves files. |

## Credentials

Resolution order, first match wins: `config.json` → `AIGW_MUSIC_API_KEY` → `AIGW_API_KEY` → `OPENAI_API_KEY`. `base_url` follows the same chain and falls back to `scripts/defaults.json`.

Every command prints JSON; failures return `{"ok": false, "error": {...}}` and exit non-zero.

## Requirements

Python 3.9 or newer. No third-party packages.
