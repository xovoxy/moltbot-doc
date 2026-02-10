# OpenClaw chat commands（斜杠命令）全量清单

这份文档整理 OpenClaw 在聊天界面里可用的 **命令（Commands）**、**指令（Directives）**，以及 host-only 的 `!` 命令。

> 命令由 **Gateway** 处理。多数命令需要作为**单独一条消息**发送（整条消息以 `/` 开头）。

## 术语：Commands vs Directives

- **Commands（命令）**：例如 `/help`、`/status`、`/restart`。
- **Directives（指令/提示）**：例如 `/think`、`/model`、`/reasoning`、`/elevated`、`/exec`、`/queue`。

Directives 的行为：
- 在**普通消息中**出现时，通常只是“内联提示”，不会持久化会话设置。
- 在**directive-only 消息**（整条消息只有 directives）中，会持久化到会话，并返回确认。
- 仅对 **authorized senders** 生效（受 allowlist/pairing + `commands.useAccessGroups` 约束）；未授权时会被当普通文本。

## 启用/配置

在 `openclaw.json`：

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

要点：
- `commands.text`：解析文本 `/...` 命令（默认 true）。
- `commands.native`：注册“原生命令”（Discord/Telegram 默认 auto；Slack 需要你在 Slack App 里创建对应 slash command）。
- `commands.config`：启用 `/config`（落盘写配置，默认关闭）。
- `commands.debug`：启用 `/debug`（运行期覆盖，默认关闭）。
- `commands.restart`：启用 `/restart`（默认关闭）。

## 命令全量清单

下面按官方 Slash commands 文档的命令列表整理（含 text-only / text+native）。

### Text + native（启用 native 时）

- `/help`
- `/commands`
- `/skill <name> [input]`（按 skill 名称运行）
- `/status`（当前状态；当 provider 支持时包含 usage/quota）
- `/allowlist`（list/add/remove allowlist entries）
- `/approve <id> allow-once|allow-always|deny`（处理 exec 审批）
- `/context [list|detail|json]`（解释“上下文”）
- `/whoami`（别名：`/id`）
- `/subagents list|stop|log|info|send`（子 agent 管理）
- `/config show|get|set|unset`（写盘配置；需要 `commands.config: true`）
- `/debug show|set|unset|reset`（运行期覆盖；需要 `commands.debug: true`）
- `/usage off|tokens|full|cost`（用量/成本显示控制）
- `/tts off|always|inbound|tagged|status|provider|limit|summary|audio`（TTS 控制；Discord 原生是 `/voice`）
- `/stop`
- `/restart`（需要 `commands.restart: true`）
- `/dock-telegram`（别名：`/dock_telegram`）
- `/dock-discord`（别名：`/dock_discord`）
- `/dock-slack`（别名：`/dock_slack`）
- `/activation mention|always`（群聊激活策略）
- `/send on|off|inherit`（owner-only）
- `/reset`（或 `/new`）：开启新会话/清空当前会话上下文
- `/new [model]`：开启新会话，并可选指定模型（model alias / provider/model）；其余文本会作为普通消息继续处理
- `/think <off|minimal|low|medium|high|xhigh>`（别名：`/thinking`、`/t`）
- `/verbose on|full|off`（别名：`/v`）
- `/reasoning on|off|stream`（别名：`/reason`；群聊谨慎）
- `/elevated on|off|ask|full`（别名：`/elev`）
- `/exec host=<sandbox|gateway|node> security=<deny|allowlist|full> ask=<off|on-miss|always> node=<id>`
- `/model <name>`（别名：`/models`；也支持配置的模型 alias）
- `/queue <mode> ...`（发 `/queue` 查看当前设置）
- `/bash <command>`（host-only；`! <command>` 的别名；需要 `commands.bash: true` + elevated allowlists）

### Text-only

- `/compact [instructions]`（见 compaction 文档）
- `! <command>`（host-only；一次只能跑一个）
- `!poll`（查看长任务输出；可选 sessionId；`/bash poll` 也可用）
- `!stop`（停止长任务；可选 sessionId；`/bash stop` 也可用）

## 重要注意事项

- 多数命令必须“单独一条消息”发送；允许可写 `:`：例如 `/think: high`。
- 群聊里：allowlisted sender 的“command-only 消息”通常会绕过 mention 要求。
- `/reasoning`、`/verbose` 在群聊里有泄露风险（内部推理/工具输出），建议默认关闭。
- Slack：如果你开启 `commands.native`，需要在 Slack App 里为每个命令创建对应的 slash command（Slack 不会自动生成）。

## 参考

- 官方文档（权威来源）：
  - https://docs.openclaw.ai/tools/slash-commands
