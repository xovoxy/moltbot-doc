# Linux 安装浏览器 & Moltbot 浏览器控制（clawd）排坑记录

## 背景
- 系统：**Alibaba Cloud Linux 3.2104 U12.2 (OpenAnolis Edition)**（ID_LIKE=rhel/fedora/centos）
- 目标：让 Moltbot 的 `moltbot browser --browser-profile clawd ...` 可以启动并控制浏览器（headless）
- 现象：Gateway 正常，但浏览器控制端口访问失败 / 工具侧无法启动受控浏览器

---

## 我们踩到的坑（重点）

### 坑 1：只重启 gateway 没用，因为浏览器控制功能默认没启
表现：
- `curl http://127.0.0.1:18791/` → `Connection refused`
- 说明：**浏览器控制服务（browser control service）没启动/没监听**

原因：
- `~/.clawdbot/moltbot.json` 里**没有 `browser` 配置块**，导致浏览器控制服务未开启。

解决：
- 在 `moltbot.json` 顶层加入 `"browser": { "enabled": true, ... }`，再重启 gateway。

---

### 坑 2：不同端口别“硬猜”，以 CLI 能跑通为准
一开始用“gateway.port + 2”去推端口（例如 18789 → 18791），但实际环境里可能出现：
- 18791 没监听
- 18792 在监听（例如 extension relay / 派生端口变化）

结论：
- **最可靠的验证方式**是直接跑 CLI：
  - `moltbot browser --browser-profile clawd start`
  - `moltbot browser --browser-profile clawd tabs`

只要这俩通了，就说明浏览器控制链路 OK。

---

### 坑 3：Linux 上默认的 `chromium-browser` 经常是“包装器/受限版”
机器上最初的浏览器路径是：
- `/usr/bin/chromium-browser`

这类在不少发行版上可能是 wrapper（Ubuntu 上常见 snap 相关问题），会导致：
- CDP 启动失败、权限/沙箱问题、进程监控异常等

解决思路：
- 推荐安装 **Google Chrome（官方包）**，并在 Moltbot 里显式指定 `executablePath`。

---

### 坑 4：Headless/Root/云服务器环境经常需要 `noSandbox=true`
很多云主机/容器/root 环境下，不加 `--no-sandbox` 会失败或异常。

解决：
- Moltbot 配置里加 `noSandbox: true`

---

## 最终可行方案（我们这次用的）

### 1) 安装 Google Chrome（Alibaba Cloud Linux / RHEL8 系）
两种方式任选其一：

**方式 A：加 repo 安装（推荐）**
```bash
sudo tee /etc/yum.repos.d/google-chrome.repo >/dev/null <<'EOF'
[google-chrome]
name=google-chrome
baseurl=https://dl.google.com/linux/chrome/rpm/stable/x86_64
enabled=1
gpgcheck=1
gpgkey=https://dl.google.com/linux/linux_signing_key.pub
EOF

sudo dnf install -y google-chrome-stable || sudo yum install -y google-chrome-stable
```

**方式 B：下载 rpm 本地安装（repo 受限时）**
```bash
wget -O /tmp/google-chrome-stable.rpm https://dl.google.com/linux/direct/google-chrome-stable_current_x86_64.rpm
sudo dnf install -y /tmp/google-chrome-stable.rpm || sudo yum localinstall -y /tmp/google-chrome-stable.rpm
```

验证：
```bash
which google-chrome-stable
google-chrome-stable --version
```

---

### 2) 开启 Moltbot browser（关键配置）
编辑：`~/.clawdbot/moltbot.json`

在顶层加入（与 `gateway/channels/plugins` 同级）：

```json
"browser": {
  "enabled": true,
  "headless": true,
  "noSandbox": true,
  "executablePath": "/usr/bin/google-chrome-stable"
}
```

然后重启：
```bash
moltbot gateway restart
```

---

### 3) 用 CLI 验证浏览器控制是否真正可用（最权威）
```bash
moltbot browser --browser-profile clawd start
moltbot browser --browser-profile clawd tabs
```

只要能看到 `running: true` 且能列出 tabs（例如 `about:blank`），就代表“浏览器控制链路已通”。

---

## 资源占用（低配 Linux 的建议规则）
- 平时可以保留少量 tab 提升速度（避免重复加载）
- **当感觉变慢/内存紧张时**再清理：
  - 关闭多余 tab
  - 必要时 `moltbot browser --browser-profile clawd stop`（最省内存）

---

## 常用排查命令清单
```bash
# 1) 确认系统
cat /etc/os-release

# 2) 确认 Chrome
which google-chrome-stable
google-chrome-stable --version

# 3) Moltbot browser 自检（最重要）
moltbot browser --browser-profile clawd start
moltbot browser --browser-profile clawd tabs

# 4) 如果还不行，看日志
# 文件日志（按实际日期）
tail -n 200 /tmp/moltbot/moltbot-$(date +%F).log

# 或 systemd 用户服务（如果你用 service 跑）
journalctl --user -u moltbot-gateway -n 200 --no-pager
```
