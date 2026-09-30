# Source: https://github.com/frankzh0330/query-agent/blob/master/docs/TELEGRAM_TEST.md

[frankzh0330](https://github.com/frankzh0330) / **[query-agent](https://github.com/frankzh0330/query-agent)** Public

- [Notifications](https://github.com/login?return_to=%2Ffrankzh0330%2Fquery-agent) You must be signed in to change notification settings
- [Fork 0](https://github.com/login?return_to=%2Ffrankzh0330%2Fquery-agent)
- [Star 0](https://github.com/login?return_to=%2Ffrankzh0330%2Fquery-agent)

 ## FilesExpand file tree

master

/

# TELEGRAM\_TEST.md

Copy path

Blame

More file actions

Blame

More file actions

## Latest commit

[![frankzh0330](https://avatars.githubusercontent.com/u/205485759?v=4&size=40)](https://github.com/frankzh0330) [frankzh0330](https://github.com/frankzh0330/query-agent/commits?author=frankzh0330)

[Organize bilingual project docs](https://github.com/frankzh0330/query-agent/commit/87f9b94ae461e433bbe587f849ca65062108ef90)

Jul 28, 2026

[87f9b94](https://github.com/frankzh0330/query-agent/commit/87f9b94ae461e433bbe587f849ca65062108ef90) · Jul 28, 2026

## History

[History](https://github.com/frankzh0330/query-agent/commits/master/docs/TELEGRAM_TEST.md)

Open commit details

History

100 lines (68 loc) · 1.89 KB

## FilesExpand file tree

master

/

# TELEGRAM\_TEST.md

Copy path

Top

## File metadata and controls

- Preview

- Code

- Blame

100 lines (68 loc) · 1.89 KB

[Raw](https://github.com/frankzh0330/query-agent/raw/refs/heads/master/docs/TELEGRAM_TEST.md)

Copy raw file

Download raw file

Outline

Edit and raw actions

# Telegram Testing Guide

[English](https://github.com/frankzh0330/query-agent/blob/master/docs/TELEGRAM_TEST.md) | [Chinese](https://github.com/frankzh0330/query-agent/blob/master/docs/TELEGRAM_TEST.zh-CN.md)

## Start The Service

### 1\. Set Environment Variables

```shell
# Required: Telegram Bot token
export TELEGRAM_BOT_TOKEN="your_bot_token_here"

# Enable mock mode when the Bearer API is not available
export BEARER_MOCK="true"

# LLM configuration
export LLM_BACKEND="zhipu"  # or "ollama"
export ZHIPU_API_KEY="your_zhipu_key"  # when using zhipu
export OLLAMA_BASE_URL="http://localhost:11434"  # when using ollama

# Optional: API service URL
export API_BASE_URL="http://localhost:8000"
```

### 2\. Run

```shell
python server.py
```

## Test Flow

1. Send a message to the Telegram bot, for example:

- `PV in Germany`
- `UV in France`
- `app_launch in California over the last 7 days`

2. The bot should return a formatted query result:

```
Query Result
Region: EUTTP
Metric: pv
Event: app_launch
Time: last 7 days

Data Result
2024-03-24: pv=12345, region=EUTTP
2024-03-23: pv=11234, region=EUTTP
2024-03-22: pv=10234, region=EUTTP
...
```

## Mock Data

In mock mode, the service:

- extracts `region / metric / event` from `exec_dsl`
- returns three example rows for recent days
- includes date and metric value in each row

## Debugging

### API Docs

```
http://localhost:8000/docs
```

### List Sessions

```shell
curl http://localhost:8000/sessions
```

### Test The NL2DSL API Directly

```shell
curl -X POST http://localhost:8000/nl2dsl \
  -H "Content-Type: application/json" \
  -d '{"text": "PV in Germany", "project_id": 55}'
```

## Troubleshooting

1. Telegram bot does not respond.

- Check whether `TELEGRAM_BOT_TOKEN` is correct.
- Check service logs for errors.

2. LLM calls fail.

- Check `LLM_BACKEND`.
- If using zhipu, check `ZHIPU_API_KEY`.

3. Memory files are not visible.

- Runtime memory is stored under the configured `data/` directory.
- Check `data/memory/`, `data/session/`, and `data/user_preferences/`.