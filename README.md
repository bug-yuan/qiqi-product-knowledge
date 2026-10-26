# qiqi-product-knowledge

企企管理云**产品知识库检索路由与引用规范**（乐享「企企产品知识库」重构版）的 Agent Skill。

装上之后，Agent 回答产品问题时会按固定路由去乐享知识库取正文，并给出可追溯的引用
（`条目码` + 乐享 `entry_id`），而不是凭记忆编产品行为。

## 这个 Skill 管什么

- **该查什么**：产品功能、单据、字段、枚举、业务规则、预算与报表口径、审批流、权限、模块归属、某版本新增/变更/下线。
- **怎么查**：主通道 `entry_list_children(模块文件夹 entry_id)` → 取正文 `block_fetch_page` / `entry_describe_ai_parse_content`，
  辅助 `search_kb_search`（按功能名）、`[IDX-00.2-*]` 模块索引页（按轴筛选）。
- **怎么引**：`[EX-002] 员工借款单 › 主要操作 —— 乐享 entry_id 1366a25d...`。
- **边界**：知识库只给**产品定义**，不证明当前租户已启用、有权限或有数据；客户实际取数走运行时核实（`qiqi-cli`）。

## 安装

需要 Node.js 22.20+ 与 `npx`。

### 方式一：把这条指令发给你的 AI 工具（推荐）

复制下面整段，直接粘给有 shell 权限的 AI 工具（Claude Code、Codex、Cursor、OpenCode 等），
它会自己完成安装和自检，**你不必记命令**：

```text
请帮我安装一个 Agent Skill（企企管理云产品知识库检索路由）。

仓库：https://github.com/bug-yuan/qiqi-product-knowledge
要求：全局安装（-g），目标 Agent 用 codex；如果你本身就是 Claude Code，则改用 claude-code。

请按顺序执行：
1. 先 `node -v` 确认 Node.js >= 22.20，不满足就停下告诉我，不要自行降级或换包管理器。
2. 执行安装：
   npx --yes skills@latest add bug-yuan/qiqi-product-knowledge -g -a codex -y
3. 装完自检，并逐项回报结果：
   - `~/.agents/skills/qiqi-product-knowledge/SKILL.md` 是否存在，把它的
     frontmatter `description` 一行念给我看（应含「企企产品知识库」）；
   - `references/` 下是否有 kb-index.md、query-playbook.md、routing-map.md 三个文件；
   - 执行 `npx --yes skills@latest ls -g --json`，确认登记项 source 为
     `bug-yuan/qiqi-product-knowledge`。
4. 提醒我重启或刷新 Agent 会话，技能才会加载。

注意：这个 Skill 里没有任何 token 或知识库正文，正文靠乐享 MCP（lexiang）实时读取。
如果你没有 shell 权限、或 npx 报网络错误，直接告诉我失败原因，不要改用其他方式安装或
自己编造一个替代品。
```

若你的 AI 工具只认 Claude Code 目录，把上面第 2 步的 `-a codex` 换成 `-a claude-code` 即可。

### 方式二：手动执行

```bash
# 全局安装（推荐，所有项目可用）
npx --yes skills@latest add bug-yuan/qiqi-product-knowledge -g -a codex -y

# 只装到当前项目
npx --yes skills@latest add bug-yuan/qiqi-product-knowledge -a codex -y

# 先看看仓里有什么，不安装
npx --yes skills@latest add bug-yuan/qiqi-product-knowledge --list
```

`-a` 指定目标 Agent，可用值见
[skills 支持的 Agent 列表](https://github.com/vercel-labs/skills/blob/main/docs/agents.md)：
`claude-code` 装到 `~/.claude/skills/`，`codex` / `cline` / `cursor` / `opencode` / `windsurf` 等装到 `~/.agents/skills/`
（Claude Code 用户也可以直接 `-a claude-code`）。不指定 `-a` 时 CLI 会交互式让你选。

安装完成后需要**重启或刷新 Agent 会话**，技能才会被加载。

### 验证与升级

```bash
# 看已装了什么、来源对不对
npx --yes skills@latest ls -g --json

# 升级到最新（仓库有新提交后重跑，等价于重新 add）
npx --yes skills@latest add bug-yuan/qiqi-product-knowledge -g -a codex -y
```

安装是**复制**而非软链：改仓库后不会自动生效，必须重新执行 `add` 覆盖。

### 前置条件：乐享 MCP

本 Skill 只是检索路由，**不含任何数据、token 或凭据**。真正读知识库靠乐享 MCP（`lexiang`）连接，
需要在 Agent 里配好该 MCP 服务并处于 connected 状态，否则路由表能读到、正文取不到。

## 目录结构

```
skills/qiqi-product-knowledge/
├── SKILL.md                      # 六层结构、强制检索流程、引用格式、事实边界
└── references/
    ├── kb-index.md               # 全量 entry_id 索引（导航台账 / 模块 / 机制 / 治理 / 变更档案）
    ├── query-playbook.md         # 检索式模板、分层策略、坑位（约 180 行）
    └── routing-map.md            # 业务意图 → 域码 → 条目码段
```

## 事实边界（重要）

- 知识库内容覆盖至 **2026-09-14**；帮助文档与迭代说明冲突时取发布日期较晚者。
- **检索无果 ≠ 能力不存在**：部分能力只在客开内部文档记载，判定存在性时须作为第三类来源考虑。
- 本仓只放**检索规范**，不放知识库正文，也不放客户交付资料。

## 维护

**本仓是唯一编辑入口**：改动直接在 `skills/qiqi-product-knowledge/` 里改 → 提交推送 →
再跑一次安装命令覆盖本地 `~/.agents/skills/`。

不要反向改：本地安装目录是 `npx ... add` 复制出来的产物，改了会在下次安装时被仓库内容覆盖，
而且改完不推送，别人装到的仍是旧版。

## License

MIT
