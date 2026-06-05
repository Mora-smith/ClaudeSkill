# find-skills 多源搜索与安装 设计文档

**日期:** 2026-06-05
**状态:** 已批准

## 概述

将 `find-skills` 技能从仅支持 skills.sh 扩展到支持多个技能市场和来源，实现三级降级搜索和两级降级安装。

## 背景

当前 `find-skills` 的 SKILL.md 仅教会 agent 使用 `npx skills find` 命令和 `skills.sh` 网站来搜索和安装技能。用户希望它支持更多来源：GitHub、可注册的 Claude Code 插件市场、以及第三方技能目录网站。

## 设计

### 市场/来源分类

#### A 类 — 可注册 Claude Code 市场

这些是包含 `.claude-plugin/marketplace.json` 的 Git 仓库，可通过 `/plugin marketplace add` 注册：

| 市场 | 注册命令 |
|------|----------|
| Anthropic 官方 | `/plugin marketplace add anthropics/claude-code` |
| Vercel Skills | `/plugin marketplace add vercel-labs/skills` |
| Hugging Face | `/plugin marketplace add huggingface/skills` |
| ClawHub (OpenClaw) | `/plugin marketplace add openclaw/clawhub` |
| 腾讯 SkillHub | `/plugin marketplace add tencent-skillhub/registry` |

#### B 类 — 技能目录网站

这些仅提供网页浏览，技能最终指向 GitHub 仓库，通过 `npx skills add` 安装：

- https://skills.sh
- https://skillsmp.com/zh
- https://killer-skills.com/zh
- https://agenticskills.io
- https://bestskills.app
- https://hub.cocoloop.cn/
- https://clawskills.sh
- https://github.com/VoltAgent/awesome-openclaw-Skills
- https://github.com/ComposioHQ/awesome-claude-skills

### 搜索策略（三级降级）

```
Level 1: 已注册市场
  └─ /plugin marketplace list 列出已注册市场
  └─ 在已注册市场中搜索技能

Level 2: GitHub + A 类市场（如果 Level 1 无结果）
  └─ 注册未注册的 A 类市场，然后在其中搜索
  └─ GitHub 搜索（gh search repos, WebSearch for awesome lists）
  └─ 读取 GitHub README 找到 market 注册 URL

Level 3: B 类目录网站（如果 Level 2 无结果）
  └─ WebFetch 逐个爬取网站搜索
  └─ 从搜索结果中提取 install 命令
```

### 安装策略（两级降级）

```
优先: /plugin marketplace add <repo> → /plugin install <skill>@<marketplace>
降级: npx skills add <owner/repo@skill>
```

### SKILL.md 结构调整

1. **触发条件** — 保持不变
2. **市场/来源知识库** — 新增：A 类可注册市场列表 + B 类目录网站列表
3. **搜索工作流** — 重写：三级降级搜索流程，每级有具体的命令和操作步骤
4. **安装工作流** — 重写：两级降级安装流程
5. **质量验证** — 保持并增强：增加来源信誉评估
6. **常见类别** — 保持
7. **搜不到时怎么办** — 保持

### 修改范围

仅修改 `skills/find-skills/SKILL.md`，不动其他文件。

## 测试要点

- Agent 能正确执行三级降级搜索
- Agent 优先使用 `/plugin` 命令安装
- Agent 在 `/plugin` 不可用时降级到 `npx skills add`
- B 类网站中的技能能正确追溯到其 GitHub 仓库
