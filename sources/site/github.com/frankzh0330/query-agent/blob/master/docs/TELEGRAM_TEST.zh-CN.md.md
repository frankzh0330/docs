# Source: https://github.com/frankzh0330/query-agent/blob/master/docs/TELEGRAM_TEST.zh-CN.md

[frankzh0330](https://github.com/frankzh0330) / **[query-agent](https://github.com/frankzh0330/query-agent)** Public

- [Notifications](https://github.com/login?return_to=%2Ffrankzh0330%2Fquery-agent) You must be signed in to change notification settings
- [Fork 0](https://github.com/login?return_to=%2Ffrankzh0330%2Fquery-agent)
- [Star 0](https://github.com/login?return_to=%2Ffrankzh0330%2Fquery-agent)

 ## FilesExpand file tree

master

/

# TELEGRAM\_TEST.zh-CN.md

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

[History](https://github.com/frankzh0330/query-agent/commits/master/docs/TELEGRAM_TEST.zh-CN.md)

Open commit details

History

91 lines (68 loc) · 1.88 KB

## FilesExpand file tree

master

/

# TELEGRAM\_TEST.zh-CN.md

Copy path

Top

## File metadata and controls

- Preview

- Code

- Blame

91 lines (68 loc) · 1.88 KB

[Raw](https://github.com/frankzh0330/query-agent/raw/refs/heads/master/docs/TELEGRAM_TEST.zh-CN.md)

Copy raw file

Download raw file

Outline

Edit and raw actions

# Telegram 测试说明

[English](https://github.com/frankzh0330/query-agent/blob/master/docs/TELEGRAM_TEST.md) | [简体中文](https://github.com/frankzh0330/query-agent/blob/master/docs/TELEGRAM_TEST.zh-CN.md)

## 启动服务

### 1\. 设置环境变量

```shell
# 必需：Telegram Bot Token
export TELEGRAM_BOT_TOKEN="your_bot_token_here"

# 启用 Mock 模式（不需要 Bearer API）
export BEARER_MOCK="true"

# LLM 配置
export LLM_BACKEND="zhipu"  # 或 "ollama"
export ZHIPU_API_KEY="your_zhipu_key"  # 如果用 zhipu
export OLLAMA_BASE_URL="http://localhost:11434"  # 如果用 ollama

# 可选：API 服务地址
export API_BASE_URL="http://localhost:8000"
```

### 2\. 启动服务

```shell
python server.py
```

## 测试流程

1. **向 Telegram Bot 发送消息**，例如：

 - `德国的PV`
 - `法国的UV`
 - `加州近7天 app_launch`
2. **Bot 会返回**：

    ```
    📊 查询结果
    地区: `EUTTP`
    指标: `pv`
    事件: `app_launch`
    时间: 近 `7` 天
    
    📈 数据结果
    2024-03-24: pv=12345, region=EUTTP
    2024-03-23: pv=11234, region=EUTTP
    2024-03-22: pv=10234, region=EUTTP
    ...(还有 2 条记录)
    ```

## Mock 数据说明

Mock 模式下会生成以下假数据：

- 自动从 exec\_dsl 提取 region/metric/event
- 返回 3 条示例数据（近3天）
- 每条数据包含日期和指标值

## 调试

### 查看 API 文档

```
http://localhost:8000/docs
```

### 查看所有会话

```shell
curl http://localhost:8000/sessions
```

### 直接测试 nl2dsl API

```shell
curl -X POST http://localhost:8000/nl2dsl \
  -H "Content-Type: application/json" \
  -d '{"text": "德国的PV", "project_id": 55}'
```

## 常见问题

1. **Telegram Bot 无响应**

 - 检查 `TELEGRAM_BOT_TOKEN` 是否正确
 - 检查日志是否有错误
2. **LLM 调用失败**

 - 检查 `LLM_BACKEND` 配置
 - 如果用 zhipu，检查 `ZHIPU_API_KEY`
3. **内存文件目录**

 - 用户记忆会保存在 `~/query-agent/memory/users/` 下
 - 可以查看学习到的用户偏好