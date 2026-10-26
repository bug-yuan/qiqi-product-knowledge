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

### 方式一：把这一句话发给你的 AI 工具（推荐）

复制下面整句，粘给任意有 shell 权限的 AI 工具（Claude Code、Codex、Cursor、OpenCode…），
**不必说明你用哪个工具、不必记命令**：

```text
请帮我安装乐享 skill（企企管理云产品知识库检索路由）。如果已经装过，先删掉再装最新版：
npx --yes skills@latest remove -g -s qiqi-product-knowledge -y
npx --yes skills@latest add bug-yuan/qiqi-product-knowledge -g -a universal -y
装完确认 ~/.agents/skills/qiqi-product-knowledge/SKILL.md 存在，并提醒我重启或刷新会话。
```

`-a universal` 是「通用安装」：只写 `~/.agents/skills/` 这一份，再由 CLI 链到各 Agent 目录，
所以**不绑定任何具体工具**（codex / claude-code / cursor / cline 都能直接读到）。

### 方式二：手动执行

```bash
# 先清掉旧版（没装过也不报错），再装最新
npx --yes skills@latest remove -g -s qiqi-product-knowledge -y
npx --yes skills@latest add bug-yuan/qiqi-product-knowledge -g -a universal -y

# 只装到当前项目（不加 -g）
npx --yes skills@latest add bug-yuan/qiqi-product-knowledge -a universal -y

# 先看看仓里有什么，不安装
npx --yes skills@latest add bug-yuan/qiqi-product-knowledge --list
```

装到哪：`-a universal` → `~/.agents/skills/qiqi-product-knowledge/`（Windows 为 `%USERPROFILE%\.agents\skills\`）。
这是跨工具通用目录，也是本 Skill 唯一的落地位置。

> ⚠️ **不要省略 `-a`**。不加 `-a` 时 CLI 会按 `--yes` 默认装到**所有**它认识的 Agent，
> 在用户目录下创建 50 多个 `.xxx` 文件夹。只用 `-a universal` 就只落一个 `.agents`。

安装完成后需要**重启或刷新 Agent 会话**，技能才会被加载。

### 验证与升级

```bash
# 看已装了什么、来源对不对（agents 字段为空属正常，universal 就是这样）
npx --yes skills@latest ls -g --json

# 升级：重跑上面方式一的 remove + add 两步即可
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
