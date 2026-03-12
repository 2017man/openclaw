# OpenClaw 0→1 部署教程（含避坑指南）

---

## 文档说明

**这份文档是干什么的**  
从零在 Windows 电脑上安装、配置 OpenClaw，直到能通过**个人飞书**在单位指挥**家里常开的笔记本**自动干活（开发、整理数据等）。文档里每一步都写了要执行什么、填什么、常见坑怎么避。

**适合谁用**  
家里有一台常开的 Windows 笔记本、在单位能用个人飞书，想用 OpenClaw 做「远程 AI 助手」的人。环境要求：Node.js 22+、能联网。

**总体需要怎么做（一句话）**  
先**第一步**：装好 OpenClaw，向导里能跳过的都跳过，只把 Web 图形界面跑起来；再**第二步**：按优先级依次配置「AI 模型 → 飞书 Channel」，可选再配 Skills、Hooks。最后在单位用飞书发消息，家里的 OpenClaw 就会在这台笔记本上执行任务。

**两步概览**  
| 阶段 | 目标 | 必做/可选 |
|------|------|------------|
| **第一步** | 安装 + 跑通图形界面（Dashboard） | 必做 |
| **第二步·1** | 配置 AI 模型与厂商（否则无法对话） | 必做 |
| **第二步·2** | 配置飞书 Channel（否则无法从单位指挥） | 必做（你的场景） |
| **第二步·3/4** | Skills、Hooks | 可选 |

**大致耗时**：第一步约 10～20 分钟（含安装与向导）；第二步·1 约 5 分钟；第二步·2 飞书约 15～30 分钟（含开放平台申请与配置）。首次建议预留约 1 小时。

**完成后的效果**：家里笔记本常开并运行 OpenClaw Gateway；在单位用个人飞书给机器人发消息，家里的 OpenClaw 会在这台电脑上执行任务（对话、写代码、整理文件等）；也可在本机浏览器打开 Dashboard 直接使用。

---

## 文档结构（方便跳读）

| 章节 | 内容 |
|------|------|
| **一、环境检查** | 安装前：Node、npm、已有目录处理 |
| **二、安装 OpenClaw** | 一键安装 / npm 安装、验证 |
| **三、第一步** | 运行 onboard 向导并**全部跳过**，手动启动 Gateway，打开 Dashboard |
| **四、第二步** | 按优先级：4.1 模型 → 4.2 飞书 → 4.3 Skills → 4.4 Hooks |
| **五** | 让 AI 能执行工具命令（创建文件等） |
| **六** | 常用命令速查 |
| **七** | 常见坑汇总 |
| **八** | 推荐学习资源 |
| **九** | 建议执行顺序总览（可打印照着做） |

---

> **适用场景**：家里常开笔记本 + 个人飞书，从零到能用。  
> **环境参考**：Windows 10、Node v22+、npm 9+（满足即可）。若本机已有 `~/.openclaw` 目录（之前装过或其它工具残留），安装前建议先备份该目录。

---

## 一、环境检查（安装前先做）

### 1.1 检查 Node 版本

```powershell
node --version
```

**期望输出**：`v22.x.x` 或更高（如 v24.13.0）  
**若版本过低**：到 [nodejs.org](https://nodejs.org/) 下载 LTS 安装，或用 nvm-windows 安装 Node 22+。

### 1.2 检查 npm

```powershell
npm --version
```

能正常输出版本号即可（如 10+）。

### 1.3 处理已有 `~\.openclaw` 目录

若目录 `C:\Users\<你的用户名>\.openclaw` 已存在：

```powershell
# 备份（可选，防止覆盖）
Copy-Item -Path "$env:USERPROFILE\.openclaw" -Destination "$env:USERPROFILE\.openclaw.backup" -Recurse
```

**坑**：若之前装过旧版 OpenClaw 或 Clawdbot，配置可能冲突。安装后若报错，可考虑删除 `~\.openclaw` 重新 `openclaw onboard`。

---

## 二、安装 OpenClaw

### 2.1 推荐：官方一键安装

在 **PowerShell**（建议以管理员身份运行）中执行：

```powershell
iwr -useb https://openclaw.ai/install.ps1 | iex
```

**执行时会做什么**：
- 从官方拉取安装脚本并执行
- 会安装 `openclaw` 到全局 npm，并可能配置 PATH

**可能需要填写的**：
- 一般无需人工输入，等待完成即可

**坑 1：执行策略限制**  
若报错类似 `无法加载文件，因为在此系统上禁止运行脚本`：

```powershell
Set-ExecutionPolicy -ExecutionPolicy RemoteSigned -Scope CurrentUser
```

然后重新执行安装命令。

**坑 2：网络超时**  
若 `openclaw.ai` 访问慢或超时，可改用社区脚本：

```powershell
iwr -useb https://raw.githubusercontent.com/736773174/openclaw-setup-cn/main/install-windows.ps1 | iex
```

### 2.2 备选：npm 全局安装

```powershell
npm install -g openclaw@beta
```

**坑**：若提示权限不足，可加 `--unsafe-perm` 或使用管理员 PowerShell。

### 2.3 验证安装

```powershell
openclaw --version
```

**期望**：输出类似 `0.x.x` 的版本号。  
**若提示「无法将 openclaw 识别为 cmdlet」**：关闭并重新打开 PowerShell，或检查 npm 全局目录是否在 PATH 中。

```powershell
openclaw doctor
```

按提示修复报错项。

---

## 三、首次配置（onboard 向导）— 第一步：先打开图形界面

**目标**：只做最少必选配置，其余**全部跳过**（含模型、渠道、技能、钩子），直到能打开 OpenClaw 的 Web 图形界面（Dashboard）。模型、Channel、Skills、Hooks 均在**第二步**按优先级逐项配置。

### 3.1 运行向导

```powershell
openclaw onboard --install-daemon
```

**执行时会做什么**：
- 创建 `~\.openclaw` 下的配置文件
- 可选安装守护进程（Windows 上可能是计划任务或服务）
- 启动交互式配置向导

### 3.2 向导步骤详解（第一步：能跳过的都选「跳过」）

#### 步骤 1：选择配置模式

```
? 选择配置模式:
  ❯ QuickStart - 快速开始（推荐新手）
    Custom - 自定义配置（高级用户）
```

**第一步建议**：选 **QuickStart** 即可，按回车。

---

#### 步骤 2：选择 AI 模型供应商 / 步骤 3：输入 API Key

若界面出现「选择 AI 模型供应商」或「输入 API Key」：

**第一步建议**：若有 **跳过 / Skip** 选项则选**跳过**，模型与 API Key 留到**第二步一**再配。若没有跳过选项，可任选一个厂商并随便填一个占位 Key，先过关，第二步再改；或选 QuickStart 时留意是否有「最小安装、仅打开界面」之类选项。

---

#### 步骤 4：是否现在配置 Channel（渠道）？

```
? 是否现在配置 Channel?
  ❯ 跳过，稍后配置
    是，现在配置
```

**第一步建议**：选 **跳过，稍后配置**，后面单独用 `openclaw channels add` 配飞书。

---

#### 步骤 5：Configure skills now?（是否现在配置技能？）

界面可能显示：

```
Configure skills now? (recommended)
  ❯ No
    Yes
```

**第一步建议**：选 **No**，先不装 Skills，等图形界面能打开后再按需安装。

---

#### 步骤 6：Enable hooks?（是否启用钩子？）

界面会显示：

```
Enable hooks?
  [•] Skip for now      ← 选这项
  [ ] boot-md
  [ ] bootstrap-extra-files
  [ ] command-logger
  [ ] session-memory

Please select at least one option.
Press `space` to select, `enter` to submit
```

**第一步建议**：用方向键选中 **Skip for now**，然后提交。

**坑：按 Enter 没反应时**  
- 先按一次 **空格 (Space)** 再按 **Enter**（有些 TUI 需要先“勾选”再提交）。  
- 若仍无反应：按 **Ctrl+C** 退出向导，重新执行 `openclaw onboard`，到这一步再试；或跳过向导，稍后需配飞书时用 `openclaw channels add`。

---

#### 步骤 6.5：个人使用确认（Personal-by-default）

可能出现英文提示：

```
I understand this is personal-by-default and shared/multi-user use requires lock-down. Continue?
```

**含义**：确认为「个人使用」；多人/共享使用需额外锁定配置。  
**操作**：选 **Continue** 继续即可（个人飞书 + 家里笔记本就是个人使用）。

---

#### 步骤 7：是否打开 Web Dashboard？

```
? 是否打开 Web Dashboard?
  ❯ 是
```

选 **是**。向导可能会尝试打开浏览器。

---

### 3.3 打开图形界面：手动启动 Gateway（必做）

向导结束后，浏览器可能自动打开 `http://127.0.0.1:18789`，但常会看到 **「无法访问此网站」/ ERR_CONNECTION_REFUSED**，说明本机还没有在跑 Gateway，需要**手动启动**。

**操作（推荐）**：

1. 新开一个 **PowerShell** 窗口（不要关）。
2. 执行（若 18789 被占用则用 18790）：

```powershell
openclaw gateway --port 18790
```

3. **保持该窗口不关**，等终端出现类似 “listening” 或 “ready” 的提示。
4. 在浏览器中访问：**http://127.0.0.1:18790**（若你用 18789 则访问 18789）。

**说明**：
- 默认端口 18789 若被占用或无法访问，改用 **18790** 即可，第一步以能打开界面为准。
- 关掉运行 `openclaw gateway` 的窗口 = 关掉 Gateway，页面会再次无法访问；下次要用 Dashboard 时重新执行上述命令即可。

**第一步完成标志**：浏览器能正常打开 OpenClaw 的 Web 控制台（图形界面），即表示第一步完成。此时可能还不能对话（未配模型）或不能从飞书指挥（未配 Channel），需在**第二步**按优先级逐项配置。

---

## 四、第二步：按优先级配置模型、Channel、Skills、Hooks

第二步在「能打开图形界面」之后进行，建议**按下面优先级**逐项完成，每完成一项可先验证再继续。

| 优先级 | 配置项 | 必选/可选 | 说明 |
|--------|--------|-----------|------|
| 1 | **AI 模型与厂商** | 必选 | 不配则无法与 AI 对话 |
| 2 | **Channel（飞书）** | 必选（你的场景） | 不配则无法从单位用飞书指挥家里 |
| 3 | **Skills** | 可选 | 扩展能力，按需安装 |
| 4 | **Hooks** | 可选 | 自动化钩子，按需配置 |

---

### 4.1 第二步一：配置 AI 模型与厂商（必做）

不配置模型时，Dashboard 里发消息不会有 AI 回复。按以下步骤添加至少一个模型。

#### 4.1.1 进入模型配置

在 PowerShell 中执行：

```powershell
openclaw onboard
```

在向导中选择与「模型 / Model / LLM」相关的选项；或执行（以实际命令为准）：

```powershell
openclaw models add
```

若没有独立命令，则再次运行 `openclaw onboard`，在向导中**不要**跳过「选择 AI 模型供应商」与「输入 API Key」两步。

#### 4.1.2 选择厂商

常见选项示例：

| 选项 | 适用场景 | 说明 |
|------|----------|------|
| **MiniMax M2.5(CN)** | 国内、中文 | 直连，有免费额度 |
| **Kimi** | 国内、长文本 | 月之暗面 |
| **DeepSeek** | 国内、代码/推理 | 代码能力强 |
| **Claude / OpenAI / Gemini** | 需国际网络 | 需可访问对应 API |
| **Ollama** | 本地、离线 | 本机安装 Ollama 后使用 |

**建议**：国内用户优先选 **MiniMax** 或 **Kimi**，无需翻墙。

#### 4.1.3 填写 API Key

- 在向导提示「输入 API Key」时，粘贴从厂商开放平台获取的 Key。
- **不要**在前后加空格或引号，粘贴后回车。

**获取方式**：

| 厂商 | 开放平台地址 |
|------|--------------|
| MiniMax | https://platform.minimax.io |
| Kimi | https://platform.moonshot.cn |
| DeepSeek | https://platform.deepseek.com |
| Claude | https://console.anthropic.com |
| OpenAI | https://platform.openai.com |

**坑**：Key 错误会提示验证失败，需重新复制（含完整字符、无换行）。

#### 4.1.4 验证

配置完成后重启 Gateway（若在运行）：

```powershell
openclaw gateway --port 18790
```

在 Dashboard（http://127.0.0.1:18790）中发一条消息，若有 AI 回复即表示模型配置成功。

---

### 4.2 第二步二：配置 Channel（飞书）（必做，你的场景）

从单位用飞书指挥家里的 OpenClaw，必须配置飞书 Channel。以下逐步骤进行。

#### 4.2.1 创建飞书应用

1. 打开 [飞书开放平台](https://open.feishu.cn/app)
2. 用 **个人飞书账号** 登录
3. 点击「创建企业自建应用」
4. 填写：
   - **应用名称**：如 `OpenClaw 家庭助手`（任意）
   - **应用描述**：如 `个人 AI 助手`（可选）
   - **应用图标**：可上传或跳过
5. 创建后进入应用，在「凭证与基础信息」中记下：
   - **App ID**（形如 `cli_xxxxxx`）
   - **App Secret**（点击「显示」后复制）

**坑**：App Secret 只显示一次，务必及时复制保存。

#### 4.2.2 配置应用权限

1. 左侧进入「权限管理」
2. 点击「批量导入权限」或逐个添加，需包含：
   - `im:message` - 接收与发送消息
   - `im:message:send_as_bot` - 以机器人身份发消息

若希望 OpenClaw 读飞书文档，可额外添加：
- `docs:document.content:read`

3. 添加后点击「批量开通」，对需要的权限全部开通

**坑**：权限开通后需等待几分钟生效，不要立刻去配事件订阅。

#### 4.2.3 启用机器人能力

1. 左侧进入「应用能力」→「机器人」
2. 开启「机器人」能力
3. 设置：
   - **机器人名称**：如 `OpenClaw`
   - **机器人描述**：可选
   - **消息卡片请求地址**：先留空（使用长连接时可不需要）

#### 4.2.4 配置事件订阅（关键：长连接）

1. 左侧进入「事件订阅」
2. 点击「添加事件」
3. 搜索并添加：`im.message.receive_v1`（接收用户发给机器人的消息）
4. **重要**：选择「使用长连接接收事件」，即 WebSocket 模式

**为什么选长连接**：
- OpenClaw 主动连飞书，家里笔记本无需公网 IP
- 不需要内网穿透或暴露端口

5. 若提示「请求网址」或「URL 验证」：
   - 需在 **OpenClaw 网关已启动** 的前提下配置
   - 按 OpenClaw 文档或飞书插件的提示填写（部分版本可能自动处理）

**坑**：若选「HTTP 回调」，需要公网可访问的 URL，家里环境一般不满足，务必选「长连接」。

#### 4.2.5 发布/上线应用

1. 在飞书开放平台中，将应用从「开发版」发布为「可用」状态
2. 若只是个人用，通常选「仅企业内可用」或「对指定用户/群组可用」

**坑**：未发布的应用，机器人可能无法正常收发消息。

#### 4.2.6 在 OpenClaw 中添加飞书

```powershell
openclaw channels add
```

或：

```powershell
openclaw onboard
```

在向导中选择 **飞书 / Feishu**，按提示输入：
- **App ID**：`cli_xxxxxx`
- **App Secret**：之前保存的密钥

**坑**：填写时注意没有多余空格。

#### 4.2.7 把机器人拉进会话

1. 在飞书中搜索你的机器人（应用名称或机器人名称）
2. 创建私聊或群聊，把机器人加进去
3. 发一条消息测试，如：`你好`

**若机器人无响应**：
- 执行 `openclaw gateway restart`（或重新执行 `openclaw gateway --port 18790`）
- 执行 `openclaw logs --follow` 查看报错
- 检查飞书应用是否已发布、权限是否已开通

---

### 4.3 第二步三：安装 Skills（可选）

Skills 用于扩展 OpenClaw 能力（如浏览器、文件操作等）。按需安装，非必选。

**常用方式**：

```powershell
# 若提供 clawhub 等工具，可搜索并安装
clawhub search browser
clawhub install browser
```

或通过 `openclaw onboard` 再次运行向导，在「Configure skills now?」时选 **Yes**，按提示选择要安装的 Skill。

**验证**：`openclaw skills list` 查看已安装技能。

---

### 4.4 第二步四：配置 Hooks（可选）

Hooks 用于在特定时机自动执行动作（如网关启动时运行 BOOT.md、会话重置时写入记忆等）。按需配置，非必选。

**方式**：再次运行 `openclaw onboard`，在「Enable hooks?」中按需勾选，例如：

- **Skip for now**：不启用
- **boot-md**：网关启动时执行 BOOT.md
- **session-memory**：/new 或 /reset 时保存会话到记忆

**坑**：若按 Enter 无反应，先按**空格**再按 **Enter** 提交。

---

## 五、让 AI 能「干活」（执行工具命令）

若发现 AI 只回复文字、不执行创建文件等操作，可尝试（版本 ≥ 2026.3.2）：

```powershell
openclaw config set tools.profile full
openclaw gateway restart
```

再在飞书中发：`在桌面创建一个 test.txt` 测试。

---

## 六、常用命令速查

| 命令 | 说明 |
|------|------|
| `openclaw --version` | 查看版本 |
| `openclaw doctor` | 环境诊断 |
| `openclaw gateway status` | 查看网关状态 |
| `openclaw gateway --port 18790` | 启动网关（指定端口，保持窗口不关） |
| `openclaw onboard` | 再次运行配置向导（第二步配模型、Channel 等） |
| `openclaw models add` | 添加 AI 模型（第二步一） |
| `openclaw channels add` | 添加渠道如飞书（第二步二） |
| `openclaw channels list` | 查看已配置渠道 |
| `openclaw skills list` | 查看已安装技能 |
| `openclaw logs --follow` | 实时查看日志（排错用） |

---

## 七、常见坑汇总

| 问题 | 可能原因 | 处理方式 |
|------|----------|----------|
| `openclaw` 命令找不到 | PATH 未更新或安装失败 | 重启 PowerShell；用 `npx openclaw` 临时测试 |
| 浏览器访问 127.0.0.1:18789 连接被拒绝 | Gateway 未启动 | 先执行 `openclaw gateway --port 18790` 并保持窗口不关，再访问 18790 |
| 飞书机器人不回复 | 未发布应用 / 权限未开通 / 未选长连接 | 检查飞书后台；重启 gateway；看 logs |
| API Key 验证失败 | Key 错误或已过期 | 去对应平台检查、重新复制 |
| 端口 18789 被占用 | 本机其他程序占用 | 改端口：`openclaw gateway --port 18790` |
| Hooks 步骤按 Enter 无反应 | TUI 需先勾选再提交 | 先按**空格**再按 Enter；或 Ctrl+C 退出后重跑 onboard |
| AI 不执行工具命令 | tools.profile 为 safe | 执行 `openclaw config set tools.profile full` |
| PowerShell 禁止脚本 | 执行策略限制 | `Set-ExecutionPolicy RemoteSigned -Scope CurrentUser` |

---

## 八、附录：国内环境安装 Node.js / npm（Windows）

OpenClaw 依赖 **Node.js 22+**。如果你在国内网络下载/安装 Node 不顺，按本节做。

### 8.1 方式 A（最稳）：直接安装官方 Node MSI

1. 打开 Node 官网下载页并选择 **LTS**（Windows x64）：`https://nodejs.org/`
2. 下载 `.msi` 后双击安装，安装选项保持默认即可。
3. 安装后打开 PowerShell 验证：

```powershell
node --version
npm --version
```

**坑**：若公司/家庭网络访问 `nodejs.org` 很慢，可改用浏览器搜索「Node.js LTS Windows x64 MSI」并从可信镜像下载；安装包来源务必可信。

### 8.2 方式 B（多版本管理）：nvm-windows

适合你需要在不同项目间切换 Node 版本。

1. 安装 nvm-windows：`https://github.com/coreybutler/nvm-windows/releases`
2. 安装后在 PowerShell 执行：

```powershell
nvm install 24.13.0
nvm use 24.13.0
node --version
```

**坑**：安装目录尽量用默认，避免路径里有中文或空格导致少数工具兼容问题。

### 8.3 npm 国内镜像（可选但推荐）

若 `npm install` 速度慢，可切到国内 registry：

```powershell
npm config set registry https://registry.npmmirror.com
npm config get registry
```

恢复官方 registry：

```powershell
npm config set registry https://registry.npmjs.org
```

### 8.4 常见问题

- **装完 Node 但 `node`/`npm` 仍提示找不到**：关闭所有终端窗口，重新打开 PowerShell；仍不行就重启电脑（通常是 PATH 未刷新）。
- **权限问题（EACCES/EPERM）**：用「管理员身份运行 PowerShell」再执行安装命令；或确保你有写入 npm 全局目录的权限。
- **代理/抓包软件影响下载**：临时关闭代理或在代理中放行 Node/npm 相关域名后重试。

---

## 八、推荐学习资源

- 官方文档：https://docs.openclaw.ai/getting-started  
- 中文快速开始：https://openclawgithub.cc/guide/start/  
- 飞书接入：https://docs.openclaw.ai/zh-CN/channels/feishu  
- 中文教程合集：https://github.com/xianyu110/awesome-openclaw-tutorial  
- 中文一键安装脚本：https://github.com/736773174/openclaw-setup-cn  

---

## 九、建议执行顺序总览

第一次看本文档时，建议先读开头「文档说明」和「文档结构」，再按下面顺序操作；已熟悉的可直接按本节约步骤执行。

**第一步（先打开图形界面）**

1. 检查环境：`node --version`、`npm --version`、`openclaw doctor`
2. 安装：`iwr -useb https://openclaw.ai/install.ps1 | iex`
3. 验证：`openclaw --version`
4. 向导：`openclaw onboard --install-daemon`，**所有能跳过的都选跳过**（含配置模式后的模型、Channel、Skills、Hooks），直到「打开 Web Dashboard」选「是」
5. 手动启动 Gateway：`openclaw gateway --port 18790`（保持窗口不关）
6. 浏览器访问 `http://127.0.0.1:18790`，能打开即第一步完成

**第二步（按优先级逐项配置）**

7. **优先级 1**：配置 AI 模型与厂商（第四章 4.1），否则无法对话
8. **优先级 2**：配置飞书 Channel（第四章 4.2 第二步二），实现单位用飞书指挥家里
9. **可选**：安装 Skills（第四章 4.3）、配置 Hooks（第四章 4.4）

完成第一步 + 第二步的模型与飞书后，即可实现：**单位用飞书发指令 → 家里笔记本的 OpenClaw 执行任务**。
