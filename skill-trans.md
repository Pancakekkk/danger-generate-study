---
name: baoyu-danger-gemini-web
description: 通过反向工程的 Gemini Web API 生成图像和文本。支持文本生成、根据提示生成图像、用于视觉输入的参考图像以及多轮对话。当其他技能需要图像生成后端时，或用户请求“使用 Gemini 生成图像”、“Gemini 文本生成”或需要具备视觉能力的 AI 生成时使用。
---

# Gemini Web 客户端

通过 Gemini Web API 进行文本/图像生成。支持参考图像和多轮对话。

## 脚本目录

**重要提示**：所有脚本都位于此技能的 `scripts/` 子目录中。

**代理执行指令**：
1. 将此 `SKILL.md` 文件的目录路径确定为 `SKILL_DIR`
2. 脚本路径 = `${SKILL_DIR}/scripts/<script-name>.ts`
3. 将本文档中所有的 `${SKILL_DIR}` 替换为实际路径

**脚本参考**：
| 脚本 | 用途 |
|--------|---------|
| `scripts/main.ts` | 文本/图像生成的 CLI 入口点 |
| `scripts/gemini-webapi/*` | `gemini_webapi` 的 TypeScript 移植版本 (GeminiClient, 类型, 工具函数) |

## 同意检查 (必填)

在首次使用之前，请验证用户是否同意使用反向工程的 API。

**同意文件位置**：
- macOS: `~/Library/Application Support/baoyu-skills/gemini-web/consent.json`
- Linux: `~/.local/share/baoyu-skills/gemini-web/consent.json`
- Windows: `%APPDATA%\baoyu-skills\gemini-web\consent.json`

**流程**：
1. 检查是否存在 `accepted: true` 且 `disclaimerVersion: "1.0"` 的同意文件
2. 如果已存在有效的同意信息 → 打印包含 `acceptedAt` 日期的警告并继续
3. 如果没有同意信息 → 显示免责声明，通过 `AskUserQuestion` 询问用户：
   - "是的，我接受" → 创建带有 ISO 时间戳的同意文件并继续
   - "不，我拒绝" → 输出拒绝信息并停止
4. 同意文件格式：`{"version":1,"accepted":true,"acceptedAt":"<ISO>","disclaimerVersion":"1.0"}`

---

## 偏好设置 (EXTEND.md)

使用 Bash 检查 `EXTEND.md` 是否存在（优先级顺序）：

```bash
# 首先检查项目层级
test -f .baoyu-skills/baoyu-danger-gemini-web/EXTEND.md && echo "project"

# 然后检查用户层级（跨平台：$HOME 适用于 macOS/Linux/WSL）
test -f "$HOME/.baoyu-skills/baoyu-danger-gemini-web/EXTEND.md" && echo "user"
```

| 路径 | 位置 |
| :--- | :--- |
| .baoyu-skills/baoyu-danger-gemini-web/EXTEND.md | 项目目录 |
| $HOME/.baoyu-skills/baoyu-danger-gemini-web/EXTEND.md | 用户主目录 |

| 结果 | 操作 |
| :--- | :--- |
| 已找到 | 读取、解析并应用设置 |
| 未找到 | 使用默认值 |

**EXTEND.md 支持**：默认模型 | 代理设置 | 自定义数据目录

## 使用方法

```bash
# 文本生成
npx -y bun ${SKILL_DIR}/scripts/main.ts "你的提示词"
npx -y bun ${SKILL_DIR}/scripts/main.ts --prompt "你的提示词" --model gemini-3-flash

# 图像生成
npx -y bun ${SKILL_DIR}/scripts/main.ts --prompt "一只可爱的猫" --image cat.png
npx -y bun ${SKILL_DIR}/scripts/main.ts --promptfiles system.md content.md --image out.png

# 视觉输入 (参考图像)
npx -y bun ${SKILL_DIR}/scripts/main.ts --prompt "描述这个" --reference image.png
npx -y bun ${SKILL_DIR}/scripts/main.ts --prompt "创建一个变体" --reference a.png --image out.png

# 多轮对话
npx -y bun ${SKILL_DIR}/scripts/main.ts "记住：42" --sessionId session-abc
npx -y bun ${SKILL_DIR}/scripts/main.ts "数字是多少？" --sessionId session-abc

# JSON 输出
npx -y bun ${SKILL_DIR}/scripts/main.ts "你好" --json
```

## 选项

| 选项 | 描述 |
|--------|-------------|
| `--prompt`, `-p` | 提示词文本 |
| `--promptfiles` | 从文件中读取提示词（串联） |
| `--model`, `-m` | 模型：gemini-3-pro (默认), gemini-3-flash, gemini-3-flash-thinking, gemini-3.1-pro-preview |
| `--image [path]` | 生成图像（默认：generated.png） |
| `--reference`, `--ref` | 视觉输入的参考图像 |
| `--sessionId` | 多轮对话的会话 ID |
| `--list-sessions` | 列出已保存的会话 |
| `--json` | 以 JSON 格式输出 |
| `--login` | 刷新 Cookie，然后退出 |
| `--cookie-path` | 自定义 Cookie 文件路径 |
| `--profile-dir` | Chrome 配置文件目录 |

## 模型

| 模型 | 描述 |
|-------|-------------|
| `gemini-3-pro` | 默认模型，最新的 3.0 Pro |
| `gemini-3-flash` | 快速、轻量级的 3.0 Flash |
| `gemini-3-flash-thinking` | 带有思考过程的 3.0 Flash |
| `gemini-3.1-pro-preview` | 3.1 Pro 预览版（空请求头，自动路由） |

## 身份验证

首次运行会打开浏览器进行 Google 身份验证。Cookie 会自动缓存。

支持的浏览器（自动检测）：Chrome, Chrome Canary/Beta, Chromium, Edge。

强制刷新：使用 `--login` 标志。覆盖浏览器路径：设置 `GEMINI_WEB_CHROME_PATH` 环境变量。

## 环境变量

| 变量 | 描述 |
|----------|-------------|
| `GEMINI_WEB_DATA_DIR` | 数据目录 |
| `GEMINI_WEB_COOKIE_PATH` | Cookie 文件路径 |
| `GEMINI_WEB_CHROME_PROFILE_DIR` | Chrome 配置文件目录 |
| `GEMINI_WEB_CHROME_PATH` | Chrome 可执行文件路径 |
| `HTTP_PROXY`, `HTTPS_PROXY` | 访问 Google 的代理（随命令内联设置） |

## 会话

会话文件存储在数据目录下的 `sessions/<id>.json` 中。

包含：`id`、`metadata`（Gemini 聊天状态）、`messages`数组、时间戳。

## 扩展支持

通过 `EXTEND.md` 进行自定义配置。有关路径和受支持的选项，请参阅 **偏好设置** 部分。
