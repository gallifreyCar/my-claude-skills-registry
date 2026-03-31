---
name: feishu-chat-mcp
description: 连接飞书机器人，实现飞书私聊/群聊对话能力。用户可通过飞书直接与 Claude 交互。
---

# Feishu Chat MCP

通过飞书机器人实现飞书对话能力，用户可在飞书私聊或群聊中与 Claude 直接交互。

## 功能

- 接收飞书私聊消息
- 接收飞书群聊消息（@机器人触发）
- 自动回复 Claude 生成的回复
- 支持多轮对话记忆

## 环境配置

使用前需配置飞书应用凭证：

```bash
# 设置环境变量（不要写在代码里）
export FEISHU_APP_ID="your_app_id"
export FEISHU_APP_SECRET="your_app_secret"
```

或在本地创建配置文件 `~/.feishu-chat/config.json`：

```json
{
  "app_id": "your_app_id",
  "app_secret": "your_app_secret"
}
```

## 飞书应用配置要求

1. 在飞书开放平台创建应用
2. 开启「机器人」能力
3. 配置事件订阅：
   - 接收消息：`im.message.receive_v1`
4. 配置请求网址（Webhook URL）
5. 发布应用并获取 App ID 和 App Secret

## 使用方式

用户在飞书中：
- 私聊：直接给机器人发消息
- 群聊：@机器人 + 消息内容

机器人会调用 Claude 并返回回复。

## 安全提醒

- **不要将 App ID 和 App Secret 写入代码或提交到 GitHub**
- 使用环境变量或本地配置文件存储凭证
- GitHub 发布时只放模板配置文件

## 相关脚本

- `scripts/feishu_chat_server.py` - 飞书消息接收服务器
- `scripts/feishu_config_template.json` - 配置模板（不含真实密钥）