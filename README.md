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

需要 Node.js 22.20+。

```bash
# 全局安装（推荐，所有项目可用）
npx --yes skills@latest add https://github.com/bug-yuan/qiqi-product-knowledge -g -a codex -y

# 只装到当前项目
npx --yes skills@latest add https://github.com/bug-yuan/qiqi-product-knowledge -a codex -y

# 先看看仓里有什么
npx --yes skills@latest add https://github.com/bug-yuan/qiqi-product-knowledge --list
```

`-a` 指定目标 Agent，可用值见
[skills 支持的 Agent 列表](https://github.com/vercel-labs/skills/blob/main/docs/agents.md)：
`claude-code` 装到 `~/.claude/skills/`，`codex` / `cline` / `cursor` / `opencode` / `windsurf` 等装到 `~/.agents/skills/`
（Claude Code 用户也可以直接 `-a claude-code`）。

安装完成后需要**重启或刷新 Agent 会话**，技能才会被加载。

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

本仓内容从内部工作副本 `~/.agents/skills/qiqi-product-knowledge/` 发布，改动请在上游改完再同步过来，
避免两处各改一份产生漂移。

## License

MIT
