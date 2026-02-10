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

## 使用场景速查（什么时候用哪个命令）

> 这节是“导航地图”。遇到问题先按场景选命令，再去下面的清单里找精确语法。

### 基础 / 导航

- `/help`：忘了语法或想看入口说明时。
- `/commands`：只想快速查看当前支持哪些斜杠命令时。
- `/status`：排查“为什么没回复/没连上/配额或限额问题”时；也可用于日常确认运行是否正常。
- `/whoami`（`/id`）：确认当前身份、是否被识别为授权发送者（allowlisted sender）时。
- `/context [list|detail|json]`：想理解“上下文是什么/为什么会满/压缩（compaction）怎么影响上下文”时。

### 会话管理 / 清空上下文

- `/reset`（或 `/new`）：当前会话变得太乱、上下文太长、或者你要切换到新任务时。
- `/new [model]`：开新会话并顺手切模型（例如这个任务要更强/更省钱的模型）。

### Skills / 功能调用

- `/skill <name> [input]`：你明确要运行某个 skill（如 weather/notion/reddit 等）并传入参数时；比自然语言更可控、更可复现。
- `/stop`：让 agent 立即停止当前输出（例如开始刷屏/方向不对时）。

### 权限 / 安全 / 审批

- `/allowlist`：新增/移除允许使用 directives/敏感操作的人；新设备/新账号接入时常用。
- `/approve <id> ...`：处理 exec 审批（allow-once/allow-always/deny）；典型场景是启用了执行但需要人工确认。

### 子 agent（后台长任务）

- `/subagents list`：查看有哪些子 agent 在跑、各自状态。
- `/subagents log`：长任务没出结果或报错，查看中间输出/错误。
- `/subagents stop`：某个子任务跑偏、卡住或不该继续时终止。
- `/subagents send`：给子 agent 追加说明，纠正方向。

### 配置 / 运行期调试（高风险，通常 owner 才开）

- `/config ...`：修改“落盘配置”（重启后仍生效）；用于确定要改 openclaw.json 的场景。
- `/debug ...`：临时覆盖运行期行为做排障（不落盘）；定位完建议 reset。
- `/restart`：修改了需要重启才能生效的配置、或网关状态异常需要重启时。

### 用量/展示控制（让信息更透明或更安静）

- `/usage ...`：控制是否展示 tokens/cost 等用量信息。
- `/compact [instructions]`：上下文太长，需要“总结压缩并保留关键事实”时。
- `/verbose ...`：需要更多解释/细节时开；只要结论时关。
- `/reasoning ...`：需要观察推理路径做调试/教学时开；群聊慎用（有泄露内部信息风险）。
- `/think ...`：控制“思考强度/花多少算力”。复杂任务调高，简单任务调低。
- `/elevated ...`：涉及执行、敏感资源或高权限动作时（建议默认 `ask` 更安全）。

### 执行 / Shell（强风险）

- `/exec ...`：明确指定执行发生位置（sandbox/gateway/node）与安全策略；用于严肃排障和可控执行。
- `/bash <command>` / `! <command>`：需要在 host 侧跑命令取信息、排障或做运维时。
- `!poll`：你启动了后台长任务，想轮询输出。
- `!stop`：终止后台长任务。

### 模型 / 队列 / 投递策略

- `/model <name>`：切换模型或模型别名（更强/更快/更便宜）。
- `/queue ...`：控制消息队列/合并/顺序等行为；排查“回复顺序怪/任务串台”也会用。
- `/send on|off|inherit`：owner-only；控制是否真实发到外部通道（灰度测试/避免误发）。

### 通道 Dock（把当前会话绑定到指定通道）

- `/dock-telegram` / `/dock-discord` / `/dock-slack`：多通道入口时，把当前会话固定绑定到某个通道继续收发。
- `/activation mention|always`：群聊里控制“必须 @ 才响应”还是“总是响应”。

### TTS

- `/tts ...`：开启/关闭语音输出，或只在特定条件触发；排查 TTS 配置时也用（Discord 原生命令通常是 `/voice`）。

## 高风险命令：推荐用法与示例

> 这些命令一旦开启/误用，可能导致**执行系统命令**、**写入配置**、或**把内部信息发到群聊**。建议默认采用 `ask`/allowlist，并尽量在私聊环境使用。

### `/elevated`

典型场景：你要临时允许更敏感的操作（例如执行、配置修改），但希望“每次都问我”。

```text
/elevated: ask
```

做完排障后，建议关掉：

```text
/elevated: off
```

### `/exec`

典型场景：你希望强制后续执行发生在 sandbox（更安全），并且未命中 allowlist 时询问。

```text
/exec host=sandbox security=allowlist ask=on-miss
```

如果你明确知道自己在做什么、并且允许完全执行（不推荐默认）：

```text
/exec host=gateway security=full ask=off
```

### `/bash` 与 `!`

典型场景：取信息/排障（查看版本、看日志、确认某个文件存在）。

```text
/bash uname -a
```

等价的 `!` 形式（更短，一次只跑一个）：

```text
! uname -a
```

长任务常用组合：

```text
! <some long command>
!poll
!stop
```

### `/config`

典型场景：你需要“落盘写配置”（重启后保持）。通常用于修复配置或开关某些能力。

查看当前配置：

```text
/config show
```

读取某个 key：

```text
/config get commands.restart
```

设置某个 key（示例：允许 /restart；注意这属于高风险开关）：

```text
/config set commands.restart true
```

### `/debug`

典型场景：你只想临时改行为复现问题（不落盘），定位完就恢复。

查看当前 debug overrides：

```text
/debug show
```

设置/取消某个 override（示例语法，具体 key 以实际实现为准）：

```text
/debug set <key> <value>
/debug unset <key>
```

恢复默认：

```text
/debug reset
```

### `/reasoning` 与 `/verbose`

典型场景：私聊排障/教学时临时打开，观察中间过程；群聊谨慎。

```text
/reasoning: on
/verbose: full
```

结束后关掉：

```text
/reasoning: off
/verbose: off
```

### `/restart`

典型场景：你刚改了需要重启才能生效的配置、或网关处于异常状态需要重启。

```text
/restart
```

> 该命令默认建议关闭，仅 owner 在明确需要时开启（见 `commands.restart`）。

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
