# Windows 使用 WSL2 安装 OpenClaw 完整教程

> 本文档适合 Windows 用户在 **WSL2 + Ubuntu** 环境中安装和使用 OpenClaw。  
> 目标是让没有 Linux 基础的用户，也能按步骤完成安装、初始化、启动 Gateway、打开 Dashboard，并在需要时配置 API、代理、D 盘迁移等内容。

---

## 目录

- [0. 阅读说明](#0-阅读说明)
- [1. 安装前准备](#1-安装前准备)
- [2. 认识两个终端：PowerShell 与 Ubuntu](#2-认识两个终端powershell-与-ubuntu)
- [3. 安装 WSL2 和 Ubuntu](#3-安装-wsl2-和-ubuntu)
- [4. 首次进入 Ubuntu](#4-首次进入-ubuntu)
- [5. 启用 systemd](#5-启用-systemd)
- [6. 安装 OpenClaw](#6-安装-openclaw)
- [7. 初始化 OpenClaw](#7-初始化-openclaw)
- [8. 检查 Gateway 并打开 Dashboard](#8-检查-gateway-并打开-dashboard)
- [9. 配置官方模型 API](#9-配置官方模型-api)
- [10. 验证安装完成](#10-验证安装完成)
- [11. 日常启动流程](#11-日常启动流程)
- [12. 创建多个 agents](#12-创建多个-agents)
- [附录 A：将 WSL2 从 C 盘迁移到 D 盘](#附录-a将-wsl2-从-c-盘迁移到-d-盘)
- [附录 B：使用 v2rayN 代理时的 WSL 配置](#附录-b使用-v2rayn-代理时的-wsl-配置)
- [附录 C：开启 mirrored 网络模式](#附录-c开启-mirrored-网络模式)
- [附录 D：配置中转 API](#附录-d配置中转-api)
- [附录 E：修改 OpenClaw 配置文件](#附录-e修改-openclaw-配置文件)
- [附录 F：Windows 防火墙规则](#附录-fwindows-防火墙规则)
- [附录 G：备份 WSL](#附录-g备份-wsl)
- [附录 H：常见问题处理](#附录-h常见问题处理)
- [附录 I：命令速查表](#附录-i命令速查表)

---

# 0. 阅读说明

本文档分为两部分：

## 主线安装流程

如果你只是想在 Windows 上安装 OpenClaw，请按顺序阅读：

```text
1. 安装前准备
2. 认识 PowerShell 与 Ubuntu
3. 安装 WSL2 和 Ubuntu
4. 首次进入 Ubuntu
5. 启用 systemd
6. 安装 OpenClaw
7. 初始化 OpenClaw
8. 检查 Gateway 并打开 Dashboard
9. 配置官方模型 API
10. 验证安装完成
```

## 附加配置

以下内容不是每个人都必须操作：

| 附加内容 | 什么时候需要 |
|---|---|
| WSL2 迁移到 D 盘 | C 盘空间不足，担心 WSL 占用 C 盘 |
| v2rayN 代理 | 你的网络需要代理才能访问安装脚本或模型 API |
| mirrored 网络 | Windows 与 WSL 的 localhost / 端口互通异常 |
| 中转 API | 不使用官方 API，而是使用第三方兼容接口 |
| 防火墙规则 | Windows 或局域网访问 Gateway 有问题 |
| WSL 备份 | 想保存当前 OpenClaw 环境，方便恢复 |

---

# 1. 安装前准备

## 1.1 推荐安装方式

Windows 下推荐使用：

```text
WSL2 + Ubuntu + OpenClaw
```

OpenClaw 官方 Windows 文档推荐通过 WSL2 安装，并说明 Gateway 服务需要 systemd。Microsoft 官方文档也说明，使用 `wsl --install` 安装的新 Linux 发行版默认会使用 WSL2。

## 1.2 需要准备的内容

| 项目 | 要求 |
|---|---|
| 操作系统 | Windows 10 新版本或 Windows 11，推荐 Windows 11 |
| Linux 发行版 | Ubuntu 24.04 |
| 网络 | 能访问 OpenClaw 安装脚本和模型 API |
| 模型 API | 官方 API 或兼容中转 API |
| 磁盘空间 | 建议至少预留 30GB，长期使用建议 50GB 以上 |

## 1.3 本教程默认值

本文档默认使用：

```text
Ubuntu 发行版名称：Ubuntu-24.04
Gateway 端口：18789
Gateway 绑定方式：loopback
OpenClaw 配置文件：~/.openclaw/openclaw.json
```

---

# 2. 认识两个终端：PowerShell 与 Ubuntu

安装过程中会出现两个命令行环境。一定要分清楚。

## 2.1 Windows PowerShell

如果你看到：

```powershell
PS C:\WINDOWS\system32>
```

说明你在 **Windows PowerShell** 中。

PowerShell 里执行 Windows / WSL 管理命令，例如：

```powershell
wsl --install
wsl --shutdown
wsl -d Ubuntu-24.04
notepad $env:USERPROFILE\.wslconfig
```

## 2.2 Ubuntu / WSL 终端

如果你看到类似：

```bash
yourname@computer:~$
```

说明你在 **Ubuntu / WSL** 中。

Ubuntu 里执行 Linux / OpenClaw 命令，例如：

```bash
curl -fsSL https://openclaw.ai/install.sh | bash
openclaw --version
openclaw gateway status
nano ~/.openclaw/openclaw.json
```

## 2.3 简单记忆

```text
PS C:\...>              Windows PowerShell
用户名@电脑名:~$         Ubuntu / WSL
```

---

# 3. 安装 WSL2 和 Ubuntu

## 3.1 打开 PowerShell 管理员模式

在 Windows 开始菜单搜索：

```text
PowerShell
```

右键选择：

```text
以管理员身份运行
```

## 3.2 安装 Ubuntu 24.04

在 PowerShell 中执行：

```powershell
wsl --install -d Ubuntu-24.04
```

如果系统提示重启电脑，请重启。

## 3.3 检查 WSL 安装状态

重启后，打开 PowerShell，执行：

```powershell
wsl -l -v
```

正常结果类似：

```text
NAME              STATE           VERSION
Ubuntu-24.04      Stopped         2
```

如果 `VERSION` 是 `2`，说明 Ubuntu 正在使用 WSL2。

---

# 4. 首次进入 Ubuntu

在 PowerShell 中执行：

```powershell
wsl -d Ubuntu-24.04
```

第一次进入 Ubuntu 时，会要求创建 Linux 用户名和密码。

示例：

```text
Enter new UNIX username: yourname
New password:
Retype new password:
```

注意：

```text
输入密码时，屏幕不会显示字符，也不会显示星号。
这是 Linux 终端的正常现象。
```

这个密码后续执行 `sudo` 命令时会用到。

---

# 5. 启用 systemd

OpenClaw Gateway 服务需要 systemd。请在 Ubuntu 中执行：

```bash
sudo tee /etc/wsl.conf >/dev/null <<'EOF'
[boot]
systemd=true
EOF
```

然后回到 PowerShell，关闭 WSL：

```powershell
wsl --shutdown
```

重新进入 Ubuntu：

```powershell
wsl -d Ubuntu-24.04
```

验证 systemd：

```bash
systemctl --user status
```

正常情况下会看到类似：

```text
State: running
Failed: 0 units
```

看到 `State: running` 即表示 systemd 已经正常启用。

---

# 6. 安装 OpenClaw

确认当前在 Ubuntu 中，然后进入用户主目录：

```bash
cd ~
```

执行官方安装脚本：

```bash
curl -fsSL https://openclaw.ai/install.sh | bash
```

安装过程中可能会提示输入 sudo 密码：

```text
[sudo] password for yourname:
```

这里输入的是你刚才创建 Ubuntu 用户时设置的密码。

注意：

```text
这不是 Windows 开机密码。
这不是 Microsoft 账号密码。
这是 Ubuntu 用户密码。
输入时屏幕不会显示任何字符。
```

安装脚本可能会自动安装：

```text
Node.js
npm
Linux build tools
OpenClaw CLI
```

请等待安装完成。

---

# 7. 初始化 OpenClaw

安装完成后，先检查 OpenClaw 是否可用：

```bash
openclaw --version
```

如果能显示版本号，说明 OpenClaw CLI 安装成功。

然后运行初始化：

```bash
openclaw onboard --install-daemon
```

初始化过程中通常会配置：

```text
Gateway
模型提供商
认证信息
默认会话
Dashboard
```

如果后续需要重新配置，可以执行：

```bash
openclaw configure
```

如果 Gateway 没有安装成功，可以执行：

```bash
openclaw gateway install
```

---

# 8. 检查 Gateway 并打开 Dashboard

## 8.1 检查 Gateway 状态

在 Ubuntu 中执行：

```bash
openclaw gateway status
```

如果 Gateway 正常运行，可以继续打开 Dashboard。

## 8.2 重启 Gateway

如果 Gateway 状态异常，可以尝试：

```bash
openclaw gateway restart
```

或者重新安装 Gateway：

```bash
openclaw gateway install
```

## 8.3 打开 Dashboard

执行：

```bash
openclaw dashboard
```

正常情况下，浏览器会打开 OpenClaw Dashboard。

如果浏览器没有自动打开，可以尝试访问：

```text
http://127.0.0.1:18789
```

---

# 9. 配置官方模型 API

> 本节使用官方 API 作为主流程。  
> 如果你使用第三方中转 API，请先完成主线安装，再阅读 [附录 D：配置中转 API](#附录-d配置中转-api)。

## 9.1 推荐使用 onboarding 配置

最适合新手的方法是在初始化过程中选择模型提供商，然后按提示输入 API Key：

```bash
openclaw onboard --install-daemon
```

或重新配置：

```bash
openclaw configure
```

## 9.2 使用配置命令设置 API Key

如果你已经知道要配置哪个 provider，也可以使用配置命令。

示例：配置 Anthropic API Key

```bash
openclaw config set models.providers.anthropic.apiKey "你的_API_KEY"
```

示例：配置默认模型

```bash
openclaw config set agents.defaults.model "anthropic/claude-sonnet-4-5"
```

> 具体可用模型名称请以 OpenClaw 当前版本的模型列表或官方文档为准。

## 9.3 修改后验证配置

每次修改配置后，都执行：

```bash
openclaw config validate
```

如果通过，再重启 Gateway：

```bash
openclaw gateway restart
```

---

# 10. 验证安装完成

当以下命令都正常时，说明 OpenClaw 安装完成。

```bash
openclaw --version
```

```bash
openclaw config validate
```

```bash
openclaw gateway status
```

```bash
openclaw dashboard
```

最终状态应该是：

```text
OpenClaw CLI 可用
配置文件校验通过
Gateway 正常运行
Dashboard 可以打开
```

---

# 11. 日常启动流程

以后每天使用 OpenClaw，只需要按以下步骤操作。

## 11.1 打开 Ubuntu

PowerShell：

```powershell
wsl -d Ubuntu-24.04
```

## 11.2 检查 Gateway

Ubuntu：

```bash
openclaw gateway status
```

## 11.3 打开 Dashboard

```bash
openclaw dashboard
```

如果 Gateway 异常：

```bash
openclaw gateway restart
```

---

# 12. 创建多个 agents

如果你希望建立多个助手，可以使用 `openclaw agents add`。

示例：

```bash
openclaw agents add amazon-ad-master --workspace ~/.openclaw/workspace-amazon-ad-master
```

```bash
openclaw agents add amazon-ops-director --workspace ~/.openclaw/workspace-amazon-ops-director
```

```bash
openclaw agents add chief-assistant --workspace ~/.openclaw/workspace-chief-assistant
```

查看 agents：

```bash
openclaw agents list
```

建议职责：

```text
amazon-ad-master：
用于广告数据分析、关键词、ACOS、ROAS、CTR、CVR、CPC、否词、预算优化。

amazon-ops-director：
用于销售额、毛利、库存周转、目标拆解、运营复盘、工作优先级。

chief-assistant：
用于接收任务、分配给其他 agents、汇总结论、形成行动方案。
```

---

# 附录 A：将 WSL2 从 C 盘迁移到 D 盘

默认情况下，WSL2 的 Ubuntu 会安装在 C 盘。如果你担心 C 盘空间不足，可以将 Ubuntu 迁移到 D 盘。

Microsoft 官方 FAQ 说明，WSL 发行版可以通过 `wsl --export` 导出，再通过 `wsl --import` 导入到指定位置，用于备份或迁移。

## A.1 关闭 WSL

PowerShell：

```powershell
wsl --shutdown
```

## A.2 创建备份目录

```powershell
mkdir D:\WSLBackup
```

## A.3 导出 Ubuntu

```powershell
wsl --export Ubuntu-24.04 D:\WSLBackup\ubuntu.tar
```

## A.4 删除 C 盘中的 Ubuntu

确认上一步没有报错后，执行：

```powershell
wsl --unregister Ubuntu-24.04
```

注意：

```text
这一步会删除 C 盘中的 Ubuntu。
必须确认 ubuntu.tar 已经成功导出。
```

## A.5 导入到 D 盘

```powershell
mkdir D:\WSL\Ubuntu-24.04
wsl --import Ubuntu-24.04 D:\WSL\Ubuntu-24.04 D:\WSLBackup\ubuntu.tar --version 2
```

## A.6 重新进入 Ubuntu

```powershell
wsl -d Ubuntu-24.04
```

---

# 附录 B：使用 v2rayN 代理时的 WSL 配置

如果你不使用代理，可以跳过本节。

如果你使用 v2rayN，且 WSL 中无法访问安装脚本或模型 API，可以按本节配置。

## B.1 查看 v2rayN 端口

v2rayN 左下角可能显示：

```text
mixed: 10808
```

这表示本机混合代理端口是：

```text
10808
```

如果你的端口不是 10808，请把下面命令中的 `10808` 替换成你的实际端口。

## B.2 临时设置代理

Ubuntu：

```bash
export http_proxy=http://127.0.0.1:10808
export https_proxy=http://127.0.0.1:10808
export HTTP_PROXY=http://127.0.0.1:10808
export HTTPS_PROXY=http://127.0.0.1:10808
export ALL_PROXY=socks5://127.0.0.1:10808
```

测试网络：

```bash
curl -I https://openclaw.ai
```

成功时通常会看到：

```text
HTTP/1.1 200 Connection established
HTTP/2 200
```

## B.3 永久保存代理配置

```bash
nano ~/.bashrc
```

在文件最后加入：

```bash
# Proxy for WSL2
export http_proxy=http://127.0.0.1:10808
export https_proxy=http://127.0.0.1:10808
export HTTP_PROXY=http://127.0.0.1:10808
export HTTPS_PROXY=http://127.0.0.1:10808
export ALL_PROXY=socks5://127.0.0.1:10808
```

保存：

```text
Ctrl + O
回车
Ctrl + X
```

立即生效：

```bash
source ~/.bashrc
```

再次测试：

```bash
curl -I https://openclaw.ai
```

---

# 附录 C：开启 mirrored 网络模式

如果你遇到 Windows 和 WSL 的 localhost / IP 不一致问题，可以考虑开启 mirrored 网络模式。

Microsoft 官方文档说明，Windows 11 22H2 及以上可以通过 `.wslconfig` 设置 `networkingMode=mirrored`，让 WSL 使用镜像网络模式，改善 Windows 与 WSL 的 localhost 互通。

## C.1 打开 `.wslconfig`

PowerShell：

```powershell
notepad $env:USERPROFILE\.wslconfig
```

写入：

```ini
[wsl2]
networkingMode=mirrored
dnsTunneling=true
autoProxy=true
firewall=true

[experimental]
autoMemoryReclaim=gradual
hostAddressLoopback=true
```

保存后关闭记事本。

## C.2 重启 WSL

PowerShell：

```powershell
wsl --shutdown
```

重新进入 Ubuntu：

```powershell
wsl -d Ubuntu-24.04
```

## C.3 验证网络模式

Ubuntu：

```bash
wslinfo --networking-mode
```

如果显示：

```text
mirrored
```

说明已经生效。

## C.4 如果 mirrored 导致网络异常

PowerShell：

```powershell
notepad $env:USERPROFILE\.wslconfig
```

改成：

```ini
[wsl2]
dnsTunneling=true
autoProxy=true
firewall=true

[experimental]
autoMemoryReclaim=gradual
```

然后重启 WSL：

```powershell
wsl --shutdown
```

---

# 附录 D：配置中转 API

如果你使用的是官方 API，可以不用看本节。

如果你使用的是第三方中转 API，需要把自定义 provider 放在：

```json
"models": {
  "providers": {}
}
```

不要放在根目录的：

```json
"providers": {}
```

否则可能出现：

```text
Unrecognized key: "providers"
```

> **扩展阅读**：更多中转 API 配置示例和最佳实践，请参考：  
> https://github.com/ai-api-gpt/openclaw-api

## D.1 打开配置文件

Ubuntu：

```bash
nano ~/.openclaw/openclaw.json
```

或使用 Windows 记事本：

```bash
notepad.exe "$(wslpath -w ~/.openclaw/openclaw.json)"
```

## D.2 生成 Gateway token

```bash
openssl rand -hex 32
```

复制输出结果，用作 Gateway token。

## D.3 中转 API 配置模板

以下是通用模板。请自行替换：

```text
YOUR_PROVIDER_NAME
YOUR_BASE_URL
YOUR_API_KEY
YOUR_MODEL_ID
YOUR_MODEL_NAME
YOUR_GATEWAY_TOKEN
```

示例：

```json
{
  "agents": {
    "defaults": {
      "model": "YOUR_PROVIDER_NAME/YOUR_MODEL_ID",
      "models": {
        "YOUR_PROVIDER_NAME/YOUR_MODEL_ID": {}
      }
    }
  },
  "models": {
    "mode": "merge",
    "providers": {
      "YOUR_PROVIDER_NAME": {
        "baseUrl": "YOUR_BASE_URL",
        "authHeader": true,
        "auth": "api-key",
        "apiKey": "YOUR_API_KEY",
        "api": "anthropic-messages",
        "models": [
          {
            "id": "YOUR_MODEL_ID",
            "name": "YOUR_MODEL_NAME",
            "api": "anthropic-messages",
            "reasoning": false,
            "contextWindow": 200000,
            "maxTokens": 8192
          }
        ]
      }
    }
  },
  "gateway": {
    "mode": "local",
    "auth": {
      "mode": "token",
      "token": "YOUR_GATEWAY_TOKEN"
    },
    "port": 18789,
    "bind": "loopback",
    "tailscale": {
      "mode": "off",
      "resetOnExit": false
    },
    "controlUi": {
      "allowInsecureAuth": true
    }
  },
  "session": {
    "dmScope": "per-channel-peer"
  },
  "tools": {
    "profile": "coding"
  }
}
```

## D.4 如果你的中转是 OpenAI 兼容格式

如果中转服务是 OpenAI-compatible API，通常需要把：

```json
"api": "anthropic-messages"
```

改为 OpenClaw 当前版本支持的 OpenAI 兼容 API 类型。具体值请以 OpenClaw 官方模型 provider 文档为准。

## D.5 验证配置

```bash
openclaw config validate
```

如果通过，重启 Gateway：

```bash
openclaw gateway restart
```

---

# 附录 E：修改 OpenClaw 配置文件

## E.1 查看实际配置文件路径

```bash
openclaw config file
```

一般是：

```text
/home/你的用户名/.openclaw/openclaw.json
```

## E.2 用 nano 修改

```bash
nano ~/.openclaw/openclaw.json
```

保存：

```text
Ctrl + O
回车
Ctrl + X
```

## E.3 用 Windows 记事本修改

```bash
notepad.exe "$(wslpath -w ~/.openclaw/openclaw.json)"
```

## E.4 修改后验证

```bash
openclaw config validate
```

通过后：

```bash
openclaw gateway restart
```

---

# 附录 F：Windows 防火墙规则

如果 Windows 浏览器或局域网访问 Gateway 有问题，可以添加防火墙规则。

PowerShell 管理员模式：

```powershell
New-NetFirewallRule -DisplayName "OpenClaw-Service" -Direction Inbound -Action Allow -Protocol TCP -LocalPort 18789
```

查看规则：

```powershell
Get-NetFirewallRule -DisplayName "OpenClaw-Service" | Format-Table
```

测试端口：

```powershell
Test-NetConnection -ComputerName localhost -Port 18789
```

注意：

```text
防火墙规则创建成功，不代表 Gateway 已经启动。
如果 TcpTestSucceeded 是 False，通常说明 18789 端口没有服务监听。
请先在 Ubuntu 中执行 openclaw gateway status。
```

---

# 附录 G：备份 WSL

建议每隔一段时间备份一次 WSL。

PowerShell：

```powershell
wsl --shutdown
wsl --export Ubuntu-24.04 D:\WSLBackup\ubuntu-openclaw-backup.tar
```

恢复时：

```powershell
wsl --import Ubuntu-24.04 D:\WSL\Ubuntu-24.04 D:\WSLBackup\ubuntu-openclaw-backup.tar --version 2
```

---

# 附录 H：常见问题处理

## H.1 `wsl -- install` 没反应

错误写法：

```powershell
wsl -- install
```

正确写法：

```powershell
wsl --install
```

或：

```powershell
wsl --install -d Ubuntu-24.04
```

`--install` 中间不能有空格。

---

## H.2 忘记 Ubuntu 密码

PowerShell：

```powershell
wsl --shutdown
wsl -d Ubuntu-24.04 -u root
```

进入 root 后：

```bash
passwd yourname
```

把 `yourname` 换成你的 Ubuntu 用户名。

退出：

```bash
exit
```

重新进入：

```powershell
wsl -d Ubuntu-24.04
```

测试：

```bash
sudo -v
```

---

## H.3 `sudo: 3 incorrect password attempts`

说明 sudo 密码输错三次。

处理方式同 [H.2 忘记 Ubuntu 密码](#h2-忘记-ubuntu-密码)。

---

## H.4 `SyntaxError: JSON5: invalid character ','`

说明 OpenClaw 配置文件中有多余逗号或括号。

错误示例：

```json
  }
},
  "gateway": {
```

正确写法：

```json
  },
  "gateway": {
```

修复后：

```bash
openclaw config validate
```

---

## H.5 `Unrecognized key: "providers"`

说明你把 provider 写到了根目录。

错误写法：

```json
{
  "providers": {}
}
```

正确写法：

```json
{
  "models": {
    "providers": {}
  }
}
```

---

## H.6 Ubuntu 中提示 `netstat` 或 `findstr` 不存在

如果你在 Ubuntu 中执行：

```bash
netstat -ano | findstr 10808
```

会报错，因为这是 Windows 命令。

应该在 PowerShell 中执行：

```powershell
netstat -ano | findstr 10808
```

---

## H.7 Windows 测试 18789 端口失败

PowerShell：

```powershell
Test-NetConnection -ComputerName localhost -Port 18789
```

如果返回：

```text
TcpTestSucceeded : False
```

先在 Ubuntu 中检查 Gateway：

```bash
openclaw gateway status
```

如果 Gateway 没启动：

```bash
openclaw gateway restart
```

或：

```bash
openclaw gateway install
```

---

# 附录 I：命令速查表

## I.1 PowerShell 常用命令

```powershell
wsl --install -d Ubuntu-24.04
wsl -l -v
wsl --shutdown
wsl -d Ubuntu-24.04
wsl -d Ubuntu-24.04 -u root
notepad $env:USERPROFILE\.wslconfig
```

## I.2 Ubuntu 基础命令

```bash
cd ~
pwd
whoami
sudo -v
systemctl --user status
```

## I.3 OpenClaw 安装

```bash
curl -fsSL https://openclaw.ai/install.sh | bash
```

## I.4 OpenClaw 初始化

```bash
openclaw --version
openclaw onboard --install-daemon
```

## I.5 OpenClaw 配置

```bash
openclaw config file
nano ~/.openclaw/openclaw.json
openclaw config validate
```

## I.6 Gateway

```bash
openclaw gateway status
openclaw gateway restart
openclaw gateway install
```

## I.7 Dashboard

```bash
openclaw dashboard
```

## I.8 Agents

```bash
openclaw agents add amazon-ad-master --workspace ~/.openclaw/workspace-amazon-ad-master
openclaw agents add amazon-ops-director --workspace ~/.openclaw/workspace-amazon-ops-director
openclaw agents add chief-assistant --workspace ~/.openclaw/workspace-chief-assistant
openclaw agents list
```

---

# 参考资料

## 官方文档

- Microsoft WSL 安装文档：`wsl --install` 安装 WSL，新 Linux 发行版默认使用 WSL2。
- Microsoft WSL FAQ：可以使用 `wsl --export` 和 `wsl --import` 备份或迁移 WSL 发行版。
- Microsoft WSL 网络文档：mirrored networking 可改善 Windows 与 WSL2 的 localhost 互通。
- OpenClaw Windows 文档：推荐 WSL2 + Ubuntu，并要求 Gateway 安装前启用 systemd。
- OpenClaw 模型 provider 文档：自定义 provider 应放在 `models.providers` 下。

## 社区教程

- **OpenClaw WSL 安装指南**：https://github.com/spoto-team/openclaw-wsl-guide/tree/main  
- **OpenClaw 接入中转 API 教程**：https://github.com/ai-api-gpt/openclaw-api  

