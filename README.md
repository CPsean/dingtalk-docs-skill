# dingtalk-docs-skill

[English](#english) | [中文](#中文)

---

## 中文

**dingtalk-docs-skill** 是一个兼容多平台的 Agent Skill，通过钉钉文档官方 MCP Server 全面操作钉钉云端文档，支持读取、推送、同步、导出、权限管理等。

### 功能

| 类别 | 能力 |
|------|------|
| **读** | 拉取云端文档正文为 markdown 或 JSONML（支持按标题/区间/标签局部读取、读取指定历史版本）、列出知识库/文件夹文档、按关键词搜索文档与文档模板、查看最近访问/编辑、获取文档元信息与封面/背景样式、下载钉盘文件与文档附件 |
| **写** | 将 Markdown/纯文本推送为可编辑钉钉文档（` ```mermaid ` 自动转原生文本绘图）、按原格式上传 HTML/图片/PDF/Office 等文件、将本地图片内联嵌入文档正文、覆盖/追加更新文档内容、块级精确编辑（段落/标题/表格等，支持建议模式改稿）、保存/回滚历史版本、设置封面与背景色、按模板创建文档、将 docx/xlsx/md/txt/xmind 等导入为在线文档 |
| **管理** | 创建文件夹、重命名/移动/复制/删除文档和文件夹、导出文档为 docx/markdown/PDF、增删改节点成员权限、转移文档或知识库所有权 |

所有操作通过[钉钉官方 MCP Server](https://aihub.dingtalk.com/#/detail?mcpId=9629&detailType=marketMcpDetail) 完成，无需本地脚本，API Key 不会写入代码。

### 快速开始

**1. 安装 Skill**

Skill 就是一个包含 `SKILL.md` 的文件夹，把它放进所用客户端的技能目录即可。`~` 表示用户主目录（Windows 为 `%USERPROFILE%`）。

| 客户端 | 用户级（推荐，所有项目可用） | 项目级（仅当前项目） |
|---|---|---|
| Claude Code | `~/.claude/skills/` | `.claude/skills/` |
| Codex CLI | `~/.codex/skills/`、`~/.agents/skills/` | `.agents/skills/` |
| Cursor | `~/.cursor/skills/`、`~/.agents/skills/` | `.cursor/skills/`、`.agents/skills/` |
| Gemini CLI | `~/.gemini/skills/`、`~/.agents/skills/` | `.gemini/skills/`、`.agents/skills/` |
| VS Code（Copilot） | `~/.copilot/skills/` | `.github/skills/` |
| Qoder | `~/.qoder/skills/` | `.qoder/skills/` |
| Trae / Trae CN | `~/.trae/skills/`（Trae CN 为 `~/.trae-cn/skills/`） | `.trae/skills/` |
| WorkBuddy | `~/.workbuddy/skills/` | `.workbuddy/skills/` |
| Pi（pi-coding-agent） | `~/.pi/agent/skills/` | `.pi/skills/` |
| Claude 桌面版 | 自定义 → 技能 → 上传 ZIP（ZIP 内保留 `dingtalk-docs-skill/` 目录；需在设置中开启代码执行） | — |

> `.agents/skills/` 是跨客户端通用的标准目录，Cursor、Codex、Gemini CLI 都会读取；Cursor 还会自动读取 Claude Code、Codex 的技能目录。只装一处想全端可用时，优先选它。

示例（以 Claude Code 用户级安装为例）：

macOS / Linux：
```bash
git clone https://github.com/CPsean/dingtalk-docs-skill \
  ~/.claude/skills/dingtalk-docs-skill
```

Windows（PowerShell）：
```powershell
git clone https://github.com/CPsean/dingtalk-docs-skill `
  "$env:USERPROFILE\.claude\skills\dingtalk-docs-skill"
```

**项目级安装**（仅当前项目可用，以 Claude Code 为例）：

```bash
git clone https://github.com/CPsean/dingtalk-docs-skill \
  .claude/skills/dingtalk-docs-skill
```

安装后确认 `<技能目录>/dingtalk-docs-skill/SKILL.md` 存在，再重启客户端或新开会话加载 Skill。目录名必须与 `SKILL.md` 中的 `name` 一致（`dingtalk-docs-skill`），否则部分客户端会静默跳过该 Skill。

**2. 开通钉钉文档 MCP 服务**

首次使用时，Skill 会引导你完成配置：

1. 打开[钉钉 AI Hub MCP 页面](https://aihub.dingtalk.com/#/detail?mcpId=9629&detailType=marketMcpDetail)，登录钉钉账号，点击【开通服务】
2. 复制页面右侧的 **StreamableHttp URL**（不是 JSON Config）
3. 把 URL 提供给客户端

Claude Code：

```bash
claude mcp add --transport http --scope user dingtalk-doc "<你的 URL>"
```

Codex CLI：

```bash
codex mcp add dingtalk-doc --url "<你的 URL>"
```

Pi：

```bash
pi mcp add dingtalk-doc --url "<你的 URL>"
```

其他客户端在 MCP 设置中新增一个 StreamableHTTP / HTTP 类型的远程服务，服务名统一用 `dingtalk-doc`，URL 填入上面复制的地址。常见配置文件：

| 客户端 | 配置文件 | 关键字段 |
|---|---|---|
| Cursor | `~/.cursor/mcp.json` 或项目 `.cursor/mcp.json` | `mcpServers` |
| VS Code（Copilot） | 项目 `.vscode/mcp.json` | `servers`（注意不是 `mcpServers`） |
| Codex CLI | `~/.codex/config.toml` | `[mcp_servers.dingtalk-doc]` |
| Pi | `~/.pi/agent/mcp.json` 或项目 `.pi/mcp.json` | `mcpServers`（`url`） |
| WorkBuddy | `~/.workbuddy/mcp.json` | `mcpServers`（`type` + `url`） |
| Qoder / Trae / Claude 桌面版 | 各自的 MCP / 连接器设置中手动添加 | 服务名 + StreamableHTTP URL |

> 服务名必须为 ASCII（`dingtalk-doc`）。URL 含个人 API Key，写入配置后不要在对话或提交中重复暴露。

**3.（可选）设置默认知识库**

提供你常用的知识库 URL，后续操作无需每次指定：

```
https://alidocs.dingtalk.com/i/spaces/xxxxx/overview
```

### 使用示例

直接用自然语言和客户端对话即可（下以 Claude Code 为例）：

- *"把 `docs/design.md` 推送到我的钉钉知识库"*
- *"用本地最新版本更新钉钉上的 PRD 文档"*
- *"把这篇带本地截图的 markdown 推到钉钉，图片要能正常显示"*
- *"列出我知识库里的所有文档"*
- *"在钉钉文档里搜索「认证」"*
- *"把会议纪要从钉钉拉取到本地"*
- *"删除钉钉上的测试文档"*

### 工作流程

```
用户请求
  → Skill 激活（通过 description 触发词匹配）
  → MCP 工具映射（create / update / list / search / delete ...）
  → 执行前确认（所有写操作需用户确认）
  → 钉钉文档 MCP Server
  → 返回结果：文档标��� + 链接
```

### 已知限制

| 限制项 | 详情 |
|---|---|
| **Mermaid 流程图** | 已支持：` ```mermaid ` 代码块在 `create_document` / `update_document` 导入时会**自动转换为钉钉原生「文本绘图」卡片**（2026-11 实测，含追加路径）。但该卡片在块级 API 中显示为 `unknown`（`rawType=card`），**无法用块级工具编辑**；回读 markdown 仍为 ` ```mermaid ` 代码块。需要改图请在钉钉编辑器中操作。 |
| **文本绘图卡片** | 除 Mermaid 外，钉钉编辑器内的其它卡片（如内嵌插件）在 `list_document_blocks` 中同样为 `unknown`/`card`，不能块级增删改；`update_document(overwrite)` 会连同卡片一起清空。 |
| **对话上传文件编码** | 非 ASCII 文件（中文等）通过对话上传时可能出现不可逆乱码。请始终提供**本地文件路径**。 |
| **文档类型** | `create_document` 和 `update_document` 仅支持 **adoc**（文字类型）文档。 |
| **非 adoc 读取** | `get_document_content` 只可读取 adoc。`axls` 表格需使用当前会话实际可用的表格工具；工具不可用时请直接提供所需内容或导出文件。`dlink` 快捷方式需先解析目标原始 nodeId，并按该 ID 去重。 |
| **推送类型选择** | Markdown/纯文本默认创建可编辑 adoc；HTML、图片、PDF、Office 等默认按原格式上传到知识库，不会擅自转换。 |
| **Markdown 大文件** | `.md` / `.markdown` 优先创建 adoc；正文超过 9,500 字符时，先由用户选择分段 adoc 或原样上传 MD 文件。 |
| **方案不明确** | 文件格式、保留原格式或在线编辑需求不明确时，先确认推送方案；方案确认不替代云端写入前的确认。 |
| **删除** | 移入回收站（30 天内可恢复），非永久删除。 |
| **本地图片内联** | 需内联进正文的图片走 `get_doc_attachment_upload_info` → PUT → `download_doc_attachment` 取签名 URL → 写入 `![](签名URL)` 自动转存链路。**统一用签名 URL，勿依赖 `resourceUrl`**（资源不可达时会得到「暂无权限访问」占位插画）。所有写操作串行，推送后做字节级校验。 |

### 安全性

- MCP 服务 URL 包含个人 API Key，仅写入本地配置，**不会提交到版本控制**
- 所有写操作执行前都需要用户明确确认

### 文件结构

```
<技能目录>/dingtalk-docs-skill/   # 技能目录见上方安装表，如 ~/.claude/skills/
├── SKILL.md                  # Skill 入口（触发器 + 指令）
├── README.md                 # 本文件
├── evals/
│   ├── evals.json            # 任务执行测评
│   └── trigger-evals.json    # 触发边界测评
└── references/
    └── mcp-tools.md          # MCP 工具速查表
```

---

## English

**dingtalk-docs-skill** is a cross-platform Agent Skill for reading, pushing, syncing, exporting, and managing documents on [DingTalk Docs](https://alidocs.dingtalk.com) via the official DingTalk Document MCP Server.

### Features

| Category | Capabilities |
|----------|-------------|
| **Read** | Pull cloud document content as markdown or JSONML (partial reads by outline/range/tags, read any historical version), list knowledge base / folder nodes, search documents and document templates, view recently accessed/edited docs, get document metadata and cover/background style, download DingTalk Drive files and attachments |
| **Write** | Push Markdown/plain text as editable DingTalk documents (` ```mermaid ` blocks auto-convert to native text diagrams), upload HTML/images/PDF/Office files in their original formats, inline local images into the document body, overwrite / append document content, make block-level edits (including suggestion mode), save / revert document versions, set cover and background, create documents from templates, and import docx/xlsx/md/txt/xmind as online documents |
| **Manage** | Create folders, rename / move / copy / delete documents and folders, export documents to docx/markdown/PDF, add / update / remove node permissions, and transfer node or knowledge base ownership |

All operations go through [DingTalk's official MCP Server](https://aihub.dingtalk.com/#/detail?mcpId=9629&detailType=marketMcpDetail) — no local scripts or API keys in code.

### Quick Start

**1. Install the Skill**

A skill is just a folder containing `SKILL.md`. Drop it into your client's skills directory. `~` is your home directory (`%USERPROFILE%` on Windows).

| Client | User-level (recommended — all projects) | Project-level (current project only) |
|---|---|---|
| Claude Code | `~/.claude/skills/` | `.claude/skills/` |
| Codex CLI | `~/.codex/skills/`, `~/.agents/skills/` | `.agents/skills/` |
| Cursor | `~/.cursor/skills/`, `~/.agents/skills/` | `.cursor/skills/`, `.agents/skills/` |
| Gemini CLI | `~/.gemini/skills/`, `~/.agents/skills/` | `.gemini/skills/`, `.agents/skills/` |
| VS Code (Copilot) | `~/.copilot/skills/` | `.github/skills/` |
| Qoder | `~/.qoder/skills/` | `.qoder/skills/` |
| Trae / Trae CN | `~/.trae/skills/` (`~/.trae-cn/skills/` for Trae CN) | `.trae/skills/` |
| WorkBuddy | `~/.workbuddy/skills/` | `.workbuddy/skills/` |
| Pi (pi-coding-agent) | `~/.pi/agent/skills/` | `.pi/skills/` |
| Claude Desktop | Customize → Skills → upload a ZIP (keep the `dingtalk-docs-skill/` folder inside the ZIP; requires code execution enabled) | — |

> `.agents/skills/` is the cross-client standard directory read by Cursor, Codex, and Gemini CLI; Cursor also auto-loads Claude Code and Codex skill directories. If you want one install to work everywhere, prefer it.

Example (Claude Code, user-level):

macOS / Linux:
```bash
git clone https://github.com/CPsean/dingtalk-docs-skill \
  ~/.claude/skills/dingtalk-docs-skill
```

Windows (PowerShell):
```powershell
git clone https://github.com/CPsean/dingtalk-docs-skill `
  "$env:USERPROFILE\.claude\skills\dingtalk-docs-skill"
```

**Project-level install** (current project only, Claude Code shown):

```bash
git clone https://github.com/CPsean/dingtalk-docs-skill \
  .claude/skills/dingtalk-docs-skill
```

Verify `<skills-dir>/dingtalk-docs-skill/SKILL.md` exists, then restart the client or start a new session. The folder name must match the `name` field in `SKILL.md` (`dingtalk-docs-skill`), otherwise some clients silently skip the skill.

**2. Enable the DingTalk Document MCP Server**

On first use, the skill will guide you through setup:

1. Open the [DingTalk AI Hub MCP page](https://aihub.dingtalk.com/#/detail?mcpId=9629&detailType=marketMcpDetail) and activate the service
2. Copy the **StreamableHttp URL** from the page
3. Provide the URL to your client

Claude Code:

```bash
claude mcp add --transport http --scope user dingtalk-doc "<your-url>"
```

Codex CLI:

```bash
codex mcp add dingtalk-doc --url "<your-url>"
```

Pi:

```bash
pi mcp add dingtalk-doc --url "<your-url>"
```

For other clients, add a StreamableHTTP / HTTP remote MCP server with the name `dingtalk-doc` and the URL above. Common config files:

| Client | Config file | Key |
|---|---|---|
| Cursor | `~/.cursor/mcp.json` or project `.cursor/mcp.json` | `mcpServers` |
| VS Code (Copilot) | project `.vscode/mcp.json` | `servers` (not `mcpServers`) |
| Codex CLI | `~/.codex/config.toml` | `[mcp_servers.dingtalk-doc]` |
| Pi | `~/.pi/agent/mcp.json` or project `.pi/mcp.json` | `mcpServers` (`url`) |
| WorkBuddy | `~/.workbuddy/mcp.json` | `mcpServers` (`type` + `url`) |
| Qoder / Trae / Claude Desktop | add it manually in the client's MCP / connector settings | name + StreamableHTTP URL |

> The server name must be ASCII (`dingtalk-doc`). The URL contains a personal API key — do not repeat it in conversations or commits.

**3. (Optional) Set a Default Knowledge Base**

Provide your knowledge base URL so you don't have to specify it every time:

```
https://alidocs.dingtalk.com/i/spaces/xxxxx/overview
```

### Usage Examples

Just talk to your client naturally (Claude Code shown below):

- *"Push `docs/design.md` to my DingTalk knowledge base"*
- *"Update the PRD document on DingTalk with the latest local version"*
- *"Push this markdown with local screenshots to DingTalk so the images render inline"*
- *"List all documents in my knowledge base"*
- *"Search DingTalk docs for 'authentication'"*
- *"Pull the meeting notes from DingTalk to local"*
- *"Delete the test document from DingTalk"*

### How It Works

```
User request
  → Skill activation (matched by description trigger)
  → MCP tool mapping (create / update / list / search / delete ...)
  → Pre-execution confirmation (for all write operations)
  → DingTalk Document MCP Server
  → Result: document title + link
```

### Known Limitations

| Limitation | Detail |
|---|---|
| **Mermaid diagrams** | Supported: ` ```mermaid ` blocks are **automatically converted into DingTalk's native "text drawing" cards** when imported via `create_document` / `update_document` (verified 2026-11, append path included). The card appears as `unknown` (`rawType=card`) in the block API and **cannot be edited with block-level tools**; reading back markdown still shows the ` ```mermaid ` fence. Edit the diagram in the DingTalk editor. |
| **Text-drawing cards** | Besides Mermaid, other editor cards (embedded plugins, etc.) also appear as `unknown`/`card` in `list_document_blocks` and cannot be inserted, updated, or deleted at block level; `update_document(overwrite)` clears them along with the rest of the body. |
| **File encoding on chat upload** | Non-ASCII files (Chinese, etc.) uploaded through the chat may suffer irreversible garbled text. Always provide a **local file path**. |
| **Document types** | `create_document` and `update_document` only support **adoc** (text document) type. |
| **Non-adoc reads** | `get_document_content` only reads adoc. `axls` needs a spreadsheet tool that is actually available in the current session; otherwise provide the content or an export. Resolve `dlink` shortcuts to their original node IDs and deduplicate by that ID. |
| **Push routing** | Markdown/plain text creates editable adoc by default. HTML, images, PDF, and Office files upload in their original formats without implicit conversion. |
| **Large Markdown** | `.md` / `.markdown` prefers adoc. Above 9,500 characters, the user chooses between segmented adoc and uploading the original MD file. |
| **Ambiguous routing** | When the format, original-format preservation, or online-editing intent is unclear, the Skill confirms the route before creating or uploading. Route confirmation is separate from write confirmation. |
| **Deletion** | Moves to trash (recoverable within 30 days), not permanent deletion. |
| **Inline local images** | Images meant for the body go through `get_doc_attachment_upload_info` → PUT → `download_doc_attachment` (signed URL) → write `![](signedURL)` so DingTalk auto-fetches and re-hosts them. **Always use the signed URL; `resourceUrl` is unreliable** (it yields a "no access" placeholder illustration). Keep writes serial and verify byte-for-byte after pushing. |

### Security

- The MCP server URL contains a personal API key — written to local config only, **never committed to version control**
- All write operations require explicit user confirmation before execution

### File Structure

```
<skills-dir>/dingtalk-docs-skill/   # see the install table above, e.g. ~/.claude/skills/
├── SKILL.md                  # Skill entry point (trigger + instructions)
├── README.md                 # This file
├── evals/
│   ├── evals.json            # Task execution evals
│   └── trigger-evals.json    # Trigger boundary evals
└── references/
    └── mcp-tools.md          # MCP tool reference
```

### License

MIT
