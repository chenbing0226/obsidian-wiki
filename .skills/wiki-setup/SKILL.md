---
name: wiki-setup
description: >
  Initialize a new Obsidian wiki vault with the correct structure, special files, and configuration.
  Triggers: "set up my wiki", "initialize obsidian", "create a new vault", "get started with the wiki",
  reconfigure vault, fix broken setup.
  中文：初始化或修复 Obsidian 知识库；用户说「搭建 wiki」「初始化 vault」「创建新库」「配置 .env」「修复 wiki 设置」时使用本技能。
---

# Obsidian 搭建 — 知识库初始化

你正在搭建新的 Obsidian wiki 知识库（或修复已有知识库）。

## 步骤 1：创建 .env

若不存在 `.env`，从 `.env.example` 复制并创建。向用户确认：

1. **知识库放在哪里？** → `OBSIDIAN_VAULT_PATH`
   - 默认：`~/Documents/obsidian-wiki-vault`
   - 须为绝对路径（展开 `~` 之后）

2. **源文档目录在哪里？** → `OBSIDIAN_SOURCES_DIR`
   - 可为多个路径，逗号分隔
   - 默认：`~/Documents`

3. **是否导入 Claude 历史？** → `CLAUDE_HISTORY_PATH`
   - 默认：从 `~/.claude` 自动发现
   - 若数据在其他位置则显式设置

4. **是否已安装 QMD？** → `QMD_WIKI_COLLECTION` / `QMD_PAPERS_COLLECTION` / `QMD_TRANSPORT`
   - 可选。启用后可在 `wiki-query` 中做语义检索，在 `wiki-ingest` 中发现相关文献。
   - 除非用户希望代理直接调用本机 `qmd` CLI，否则默认 `QMD_TRANSPORT=mcp`。
   - 若使用 CLI 模式，默认设 `QMD_CLI_SEARCH_MODE=quality`；若重排序过慢可建议 `balanced`。
   - 若不确定可先跳过 — 两个技能都会自动回退到 `Grep`。
   - 安装说明：见 `.env.example`（QMD 小节）。

## 步骤 2：创建知识库目录结构

```bash
mkdir -p "$OBSIDIAN_VAULT_PATH"/{concepts,entities,skills,references,synthesis,journal,projects,_archives,_raw,.obsidian}
```

- `.obsidian/` — Obsidian 自身配置，用于被识别为库。
- `projects/` — 按项目的知识（在导入过程中填充）。
- `_archives/` — 存放 wiki 快照，供重建/恢复使用。
- `_raw/` — 未加工草稿暂存区。把零碎笔记丢在这里；`wiki-ingest` 会将其提升为正式 wiki 页并删除原文件。

## 步骤 3：创建特殊文件

### index.md

```markdown
---
title: Wiki Index
---

# Wiki Index

*本索引由系统自动维护。上次更新：TIMESTAMP*

## Concepts

*尚无页面。使用 `wiki-ingest` 添加第一个来源。*

## Entities

## Skills

## References

## Synthesis

## Journal
```

### log.md

```markdown
---
title: Wiki Log
---

# Wiki Log

- [TIMESTAMP] INIT vault_path="OBSIDIAN_VAULT_PATH" categories=concepts,entities,skills,references,synthesis,journal
```

### hot.md

```markdown
---
title: Hot Cache
updated: TIMESTAMP
---

# Hot Cache

*约 500 字的近期活动语义快照。每次重大写入后更新。*

## Recent Activity

- [TIMESTAMP] INIT — 知识库创建于 OBSIDIAN_VAULT_PATH

## Active Threads

*尚无 — 开始导入来源以填充。*

## Key Takeaways

*尚无。*

## Flagged Contradictions

*尚无。*
```

## 步骤 4：创建 .obsidian 配置

创建最小 Obsidian 配置以获得开箱即用体验：

### .obsidian/app.json
```json
{
  "strictLineBreaks": false,
  "showFrontmatter": false,
  "defaultViewMode": "preview",
  "livePreview": true
}
```

### .obsidian/appearance.json
```json
{
  "baseFontSize": 16
}
```

## 步骤 5：推荐 Obsidian 插件

告知用户以下社区插件（需手动安装）：

1. **Dataview** — 查询页面元数据、构建动态表格。对 wiki 很实用。
2. **Graph Analysis** — 增强图谱，便于探索关联。
3. **Templater** — 若希望用模板手动建页。
4. **Obsidian Git** — 将知识库自动备份到 git 仓库。

## 步骤 6：验证搭建

快速自检：

- [ ] 知识库目录存在且包含：`concepts/`、`entities/`、`skills/`、`references/`、`synthesis/`、`journal/`、`projects/`、`_archives/`、`_raw/`
- [ ] 根目录存在 `index.md`
- [ ] 根目录存在 `log.md`
- [ ] 根目录存在 `hot.md`
- [ ] `.env` 中已设置 `OBSIDIAN_VAULT_PATH`
- [ ] 存在 `.obsidian/` 目录
- [ ] 若已配置源目录，则这些目录存在且可读

汇报结果，并告知用户现在可以：

1. 在 Obsidian 中打开该库（文件 → 打开库 → 选择目录）
2. 运行 `wiki-status` 查看可导入内容
3. 运行 `wiki-ingest` 添加第一批来源
4. 运行 `claude-history-ingest` 挖掘 Claude 对话
5. 若使用 Codex，可运行 `codex-history-ingest` 挖掘会话
6. 随时再次运行 `wiki-status` 查看增量
