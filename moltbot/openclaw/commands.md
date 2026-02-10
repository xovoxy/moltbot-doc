# OpenClaw chat commands（斜杠命令）速查

这份文档整理 OpenClaw 在聊天界面里可用的 **`/` 命令**（例如 `/status`）。

> 说明：命令由 **Gateway** 处理。多数命令需要作为**单独一条消息**发送（整条消息以 `/` 开头）。

## 两类东西：Commands vs Directives

- **Commands（命令）**：形如 `/help`、`/status`、`/restart`。
- **Directives（指令/提示）**：形如 `/think`、`/model`、`/reasoning`、`/elevated`、`/exec`、`/queue`。
  - Directives 可以作为“单独一条消息”持久化到会话，也可以在普通消息中作为“内联提示”（不持久）。

## 开关配置（是否启用命令）

在 `openclaw.json` 里：

```json5
{
  "commands": {
    "text": true,
    "native": "auto",
    "nativeSkills": "auto",
    "bash": false,
    "bashForegroundMs": 2000,
    "config": false,
    "debug": false,
    "restart": false,
    "useAccessGroups": true
  }
}
```

- `commands.text`：是否解析文本 `/...` 命令（默认 true）。
- `commands.native`：是否注册“原生命令”（Discord/Telegram 更常用；Slack 默认关闭，需要你在 Slack App 里创建 slash command）。
- `commands.config`：允许 `/config` 写入 `openclaw.json`（默认关闭）。
- `commands.debug`：允许 `/debug`（运行期覆盖，不落盘；默认关闭）。
- `commands.restart`：允许 `/restart`（默认关闭）。

## 常用命令速查

### 查询类

- `/status`：查看当前 OpenClaw 状态（包含当前 provider 的 usage/quota 信息：如果可用）。
- `/context [list|detail|json]`：解释“上下文”包含什么；`detail` 会给出按文件/工具/skill/system prompt 的占用。
- `/whoami`：查看你在当前渠道的 sender id（别名：`/id`）。
- `/commands`：命令列表（简表）。
- `/help`：帮助。

### 会话控制

- `/stop`：停止当前进行中的一次运行。
- `/reset` 或 `/new [model]`：开启新会话；可选传模型提示。

### 模型 / 推理与输出控制

- `/think <off|minimal|low|medium|high|xhigh>`：设置思考强度（别名：`/thinking`、`/t`）。
- `/verbose on|full|off`：控制额外输出（调试用；别名：`/v`）。
- `/reasoning on|off|stream`：控制推理输出（调试/谨慎使用；别名：`/reason`）。
- `/model <name>`：切换模型（也可用 `/models` 或配置的模型 alias）。
- `/queue <mode> ...`：队列/去抖/容量等设置（发 `/queue` 查看当前）。

### 权限与执行

- `/elevated on|off|ask|full`：提升权限/执行策略（别名：`/elev`）。
- `/exec host=<sandbox|gateway|node> security=<deny|allowlist|full> ask=<off|on-miss|always> node=<id>`：查看/设置 exec 执行位置与策略（发 `/exec` 查看当前）。
- `/approve <id> allow-once|allow-always|deny`：处理 exec 审批请求。

### 配置/调试（默认关闭，需显式开启）

- `/config show|get|set|unset`：读写 `openclaw.json`（owner-only；需要 `commands.config: true`）。
- `/debug show|set|unset|reset`：运行期覆盖（owner-only；需要 `commands.debug: true`）。

### 频道/路由（跨渠道“停靠”）

- `/dock-telegram`（别名 `/dock_telegram`）
- `/dock-discord`（别名 `/dock_discord`）
- `/dock-slack`（别名 `/dock_slack`）

### 群聊行为

- `/activation mention|always`：群聊里是否需要 @ 才响应。
- `/send on|off|inherit`：控制是否允许发送（owner-only）。

### 技能（skills）

- `/skill <name> [input]`：按 skill 名称运行。

### Bash（host-only，谨慎）

- `! <command>`：运行 shell 命令（一次只能跑一个）。
- `/bash <command>`：`! <command>` 的别名。
- `!poll` / `!stop`：查看/停止长任务（`/bash poll` / `/bash stop` 也可用）。

### 重启

- `/restart`：重启 Gateway（默认关闭；需要 `commands.restart: true`）。

## 注意事项

- 命令通常需要作为独立消息发送；允许可用 `:` 形式：例如 `/think: high`。
- 群聊里：来自 allowlisted/authorized sender 的“命令-only 消息”通常会绕过 mention 要求。
- `/reasoning`、`/verbose` 在群聊里有泄露风险（可能暴露内部推理/工具输出），建议默认关闭。

## 参考

- OpenClaw 官方文档：Slash commands
  - https://docs.openclaw.ai/tools/slash-commands
