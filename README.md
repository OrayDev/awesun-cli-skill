# awesun-cli-skill

> 面向 AI Agent（Claude Code / OpenCode 等）的 AweSun（向日葵）远程控制技能包。
> 让 AI 助手通过 `awesun-cli` 完成设备管理、远程桌面控制、文件传输、远程命令执行与端口转发等自动化操作。

[![License](https://img.shields.io/badge/license-MIT-blue.svg)](LICENSE)
[![Skill Version](https://img.shields.io/badge/SKILL-1.0.6-brightgreen.svg)](SKILL.md)
[![CLI Version](https://img.shields.io/badge/awesun--cli-1.0.6-orange.svg)](https://www.npmjs.com/package/@aweray/awesun-cli)

---

## 这是什么

本仓库是一个标准的 **Agent Skill**（技能包）：AI Agent 加载 `SKILL.md` 后，即可获得操作向日葵远程设备的完整能力，包括：

- **远程桌面自动化** —— 截图、鼠标点击/移动/滚动/拖拽、键盘输入、粘贴板，支持归一化坐标（0.0–1.0）
- **远程命令执行** —— Windows 执行 `cmd`、Linux/Mac 执行 `ssh`，支持获取本地 SSH 代理地址
- **远程文件管理** —— 浏览、创建、重命名、删除目录文件，异步上传/下载并支持状态轮询
- **端口转发** —— 将远程服务端口映射到本机，本地端口动态分配
- **设备管理** —— 设备发现、状态查询、密码管理、远程关机/重启/唤醒
- **多场景连接** —— 已绑定设备（`--remote-id`）、临时协助识别码（`--fastcode`）、免密授权（`--interactive`）三种连接方式的智能路由

技能内置 8 份按场景组织的深度参考文档，Agent 按需加载，避免一次性占用过多上下文。

---

## 目录结构

```text
awesun-cli-skill/
├── SKILL.md                        # 技能入口：生命周期模型、场景导航、关键执行规则
├── scripts/
│   └── coordinates.py              # 像素坐标 → 归一化坐标转换工具
├── references/                     # 按场景拆分的深度参考文档
│   ├── account-guide.md            # 账号与认证：登录、无头 URL 授权、License、免费试用
│   ├── device-guide.md             # 设备管理：列表、搜索、授权、远程电源操作
│   ├── connection-guide.md         # 连接策略：remote-id vs fastcode 决策树、凭证规则
│   ├── desktop-guide.md            # 桌面自动化：坐标系、截图、鼠标键盘操作
│   ├── file-guide.md               # 文件管理：目录操作、上传下载、传输状态轮询
│   ├── remote-exec-guide.md        # 命令执行：cmd / ssh、SSH 代理
│   ├── port-forwarding-guide.md    # 端口转发：通道配置、动态本地端口
│   ├── ui-locator.md               # UI 元素定位与归一化坐标换算
│   └── ui-patterns.md              # 常见 UI 视觉模式目录
├── LICENSE                         # MIT License
└── README.md
```

---

## 安装

### 1. 安装 awesun-cli（前置依赖）

```bash
npm install -g @aweray/awesun-cli
```

### 2. 安装技能

将本仓库克隆到 Agent 的技能目录：

**Claude Code（用户级）**

```bash
git clone https://github.com/<your-org>/awesun-cli-skill.git \
  ~/.claude/skills/awesun-cli
```

**Claude Code（项目级）**

```bash
git clone https://github.com/<your-org>/awesun-cli-skill.git \
  .claude/skills/awesun-cli
```

**其他支持 SKILL.md 标准的 Agent**

放入对应技能目录即可，要求 `SKILL.md` 位于技能根目录且包含 `name` / `description` frontmatter。

### 3. 首次登录

```bash
# 交互式登录
awesun-cli login

# 无头 / 云端环境使用 URL 授权
awesun-cli login --url

# 确认登录状态
awesun-cli login status
```

---

## 快速开始

每个远程任务遵循统一的 5 步生命周期：

```text
[1. 验证登录]  →  [2. 解析目标]  →  [3. 建立会话]  →  [4. 执行操作]  →  [5. 断开会话]
login status      remote-id /        session connect      desktop / file /    session
                  fastcode           --type               exec / forward      disconnect
```

示例：远程点击已绑定设备的桌面中心

```bash
# 建立桌面会话
awesun-cli session connect --type desktop --remote-id 1738789907
# => session_id: abc123xyz

# 截图确认当前画面
awesun-cli session screenshot abc123xyz --output /tmp/desktop.png

# 点击屏幕中心（归一化坐标 0.5, 0.5）
awesun-cli desktop mouse click abc123xyz --x 0.5 --y 0.5

# 断开
awesun-cli session disconnect abc123xyz
```

---

## 场景导航

Agent 根据用户任务按需加载对应参考文档：

| 场景 | 适用情况 | 主要命令 | 参考文档 |
| --- | --- | --- | --- |
| **账号与认证** | 登录、状态检查、无头 URL 授权、License、免费试用 | `login`、`login --url`、`logout`、`service info`、`service get-trial` | `references/account-guide.md` |
| **设备管理** | 发现设备、查询状态、更新元数据、密码管理、远程电源 | `device ls`、`device search`、`device info`、`device grant`、`device shutdown/restart/wakeup` | `references/device-guide.md` |
| **会话与连接** | 连接目标、区分 `--remote-id` 与 `--fastcode`、会话管理 | `session connect`、`session ls`、`session status`、`session disconnect` | `references/connection-guide.md` |
| **桌面自动化** | 鼠标点击/拖拽/滚动、键盘输入、归一化坐标 | `session screenshot`、`desktop mouse click/move/scroll/drag`、`desktop type/paste/key` | `references/desktop-guide.md` |
| **文件管理** | 浏览远程文件、新建目录、重命名、删除、异步传输 | `file ls`、`file mkdir`、`file mv`、`file rm`、`file transfer`、`file transfer status/cancel` | `references/file-guide.md` |
| **命令执行与 SSH** | Windows `cmd` / Linux `ssh` 执行命令、获取 SSH 代理地址 | `session exec`、`ssh address` | `references/remote-exec-guide.md` |
| **端口转发** | 通过转发隧道将远程服务端口暴露到本机 | `forward config` | `references/port-forwarding-guide.md` |
| **UI 坐标定位** | 视觉元素定位与归一化（0.0–1.0）坐标换算 | `scripts/coordinates.py` | `references/ui-locator.md`、`references/ui-patterns.md` |

---

## 关键规则速览

完整规则见 `SKILL.md`，核心要点：

1. **认证预检** —— 任何操作前先执行 `awesun-cli login status`；无头环境使用 `login --url`。
2. **目标解析** —— 账号内已绑定设备用 `--remote-id`（访问密码）；未绑定的临时识别码用 `--fastcode`（验证码或 `--interactive` 免密授权）。二者不可混用，失败后禁止盲目重试另一种方式。
3. **会话类型匹配** —— 桌面操作需 `type=desktop`，文件操作需 `type=file`，命令执行需 `type=cmd`/`type=ssh`，端口转发需 `type=forward`。
4. **文件传输** —— `--local` 必须是已存在的本地**目录**（保留原文件名）；传输后轮询 `file transfer status` 直到 `finished`。
5. **端口转发** —— 通道 JSON 只接受 `channel_id`、`address`、`port`（无 `local_port`）；本地端口动态分配，从返回结果读取。
6. **上下文复用** —— 优先复用对话中已有的 `remote_id` / `session_id`，避免重复查询。

---

## 辅助脚本

`scripts/coordinates.py` 用于像素坐标与归一化坐标互转（无需第三方依赖，Python 3 即可运行）：

```bash
# 像素坐标 → 归一化坐标（1920x1080 屏幕的中心点）
python3 scripts/coordinates.py norm 960 540 1920 1080
# Normalized coordinates: (0.5, 0.5)

# 校验归一化坐标是否在合法范围
python3 scripts/coordinates.py validate 0.5 0.5
# Coordinates (0.5, 0.5) valid: YES
```

---

## 全局约定

| 项目 | 说明 |
| --- | --- |
| 输出格式 | `-o json` / `table` / `yaml` / `wide`，自动化场景建议 `json` |
| 调试日志 | `-v` / `--verbose` 输出详细请求/响应日志 |
| 日志位置 | `~/awesun-cli/log/` |
| 守护进程 | 按需自动启动，`awesun-cli stop-daemon` 手动停止 |
| 缓存清理 | `awesun-cli config clean-cache` |

---

## 常见问题

**Q: `--remote-id` 和 `--fastcode` 都是 9~10 位数字，怎么区分？**

长度无法区分，必须根据**来源和上下文**判断：来自 `device ls` / `device search` 的是 `remote-id`；来自「我的识别码」分享模板或明确表述为远程协助的是 `fastcode`。无法确定时执行 `device info <ID>`，返回 `Remote not exists` 即为 `fastcode`。详见 `references/connection-guide.md`。

**Q: 连接失败后换个参数重试？**

禁止。未验证标识来源前，禁止把失败的 `--remote-id` 连接改用 `--fastcode` 重试（反之亦然）。

---

## 许可证

[MIT](LICENSE) © Shanghai Best Oray Information Technology Co., Ltd
