# find-skills 多源搜索与安装 实现计划

> **For agentic workers:** REQUIRED SUB-SKILL: Use superpowers:subagent-driven-development (recommended) or superpowers:executing-plans to implement this plan task-by-task. Steps use checkbox (`- [ ]`) syntax for tracking.

**Goal:** 将 find-skills SKILL.md 从单一 skills.sh 来源扩展为多源搜索（已注册市场 → GitHub → 第三方网站）和多级安装（/plugin → npx skills add）

**Architecture:** 纯文档修改。仅修改 `skills/find-skills/SKILL.md`，重写搜索和安装工作流，新增市场知识库章节。不改动任何代码文件。

**Tech Stack:** Markdown + Claude Code skill 指令语法

---

### Task 1: 重写 SKILL.md — Frontmatter 和触发条件

**Files:**
- Modify: `skills/find-skills/SKILL.md:1-19`

- [ ] **Step 1: 更新 frontmatter 和头部章节**

保留现有 frontmatter，增强 description 以反映多源搜索能力。重写 "When to Use This Skill" 和 "What is the Skills CLI?" 为更全面的 "Skill Ecosystem Overview"。

替换文件开头到 "## How to Help Users Find Skills" 之前的内容：

```markdown
---
name: find-skills
description: Helps users discover and install agent skills from multiple sources (registered marketplaces, GitHub, skill directories). Use when users ask "how do I do X", "find a skill for X", "is there a skill that can...", or express interest in extending capabilities. Supports searching across registered Claude Code marketplaces, GitHub repositories, and third-party skill directory websites.
---

# Find Skills

This skill helps you discover and install skills from the entire agent skills ecosystem — registered Claude Code marketplaces, GitHub, and third-party skill directories.

## When to Use This Skill

Use this skill when the user:

- Asks "how do I do X" where X might be a common task with an existing skill
- Says "find a skill for X" or "is there a skill for X"
- Asks "can you do X" where X is a specialized capability
- Expresses interest in extending agent capabilities
- Wants to search for tools, templates, or workflows
- Mentions they wish they had help with a specific domain (design, testing, deployment, etc.)

## Skill Ecosystem Overview

Skills exist across multiple sources. Understanding this landscape is key to finding the right skill.

### Category A: Registrable Claude Code Marketplaces

These are Git repositories containing a `.claude-plugin/marketplace.json` file. They can be registered with `/plugin marketplace add` and then searched/installed via `/plugin` commands.

| Marketplace | Register Command |
|-------------|-----------------|
| Anthropic Official | `/plugin marketplace add anthropics/claude-code` |
| Vercel Skills | `/plugin marketplace add vercel-labs/skills` |
| Hugging Face Skills | `/plugin marketplace add huggingface/skills` |
| ClawHub (OpenClaw) | `/plugin marketplace add openclaw/clawhub` |
| Tencent SkillHub | `/plugin marketplace add tencent-skillhub/registry` |

### Category B: Skill Directory Websites

These websites let you browse skills via web UI. Skills listed on them ultimately point to GitHub repositories. Use WebFetch to search these sites, then install found skills via `npx skills add`.

| Directory | URL |
|-----------|-----|
| skills.sh | https://skills.sh |
| SkillsMP | https://skillsmp.com/zh |
| Killer Skills | https://killer-skills.com/zh |
| Agentic Skills | https://agenticskills.io |
| Best Skills | https://bestskills.app |
| Cocoloop Hub | https://hub.cocoloop.cn/ |
| Claw Skills | https://clawskills.sh |
| Awesome OpenClaw (GitHub) | https://github.com/VoltAgent/awesome-openclaw-Skills |
| Awesome Claude (GitHub) | https://github.com/ComposioHQ/awesome-claude-skills |
```

- [ ] **Step 2: 验证文件格式**

```bash
head -5 skills/find-skills/SKILL.md
```
确认 frontmatter 格式正确。

---

### Task 2: 重写搜索工作流（三级降级）

**Files:**
- Modify: `skills/find-skills/SKILL.md`（替换原 "## How to Help Users Find Skills" 章节）

- [ ] **Step 1: 写入搜索工作流**

替换原有的 "## How to Help Users Find Skills" 及其 Step 1-3 内容，写入完整的搜索工作流：

```markdown
## Search Workflow (Three-Tier Cascade)

**Rule:** Only proceed to the next tier if the current tier yields no relevant results.

### Tier 1: Search Registered Marketplaces

Start here — it's fastest and uses already-configured sources.

1. List registered marketplaces:
   ```bash
   /plugin marketplace list
   ```

2. Search for the skill in registered marketplaces. Use `/plugin search <query>` if available, or browse the marketplace the user mentions.

3. If you find a relevant skill, skip to the Install Workflow.

4. If no registered marketplaces exist or no relevant skill is found, proceed to Tier 2.

### Tier 2: Register Known Marketplaces + Search GitHub

If Tier 1 fails, broaden the search.

1. **Register Category A marketplaces** the user doesn't have yet:
   ```bash
   /plugin marketplace add anthropics/claude-code
   /plugin marketplace add vercel-labs/skills
   /plugin marketplace add huggingface/skills
   /plugin marketplace add openclaw/clawhub
   /plugin marketplace add tencent-skillhub/registry
   ```

2. Search again in the newly registered marketplaces.

3. **Also search GitHub** in parallel:
   - Use `gh search repos` to find skill repositories:
     ```bash
     gh search repos "SKILL.md <query>" --limit 20
     ```
   - Or search by topic:
     ```bash
     gh search repos "topic:claude-skills <query>" --limit 20
     ```
   - Or use WebSearch: `site:github.com SKILL.md <query>`

4. **Read GitHub README** of any found repos — they often list a marketplace registration URL you can use with `/plugin marketplace add`.

5. If you find a relevant skill, register its market (if needed) and skip to the Install Workflow.

6. If still no relevant skill found, proceed to Tier 3.

### Tier 3: Search Category B Directory Websites

Last resort — crawl skill directory sites.

1. Use WebFetch to search each Category B site. Start with the most popular ones:

   ```
   WebFetch: https://skills.sh/ → search for <query>
   WebFetch: https://agenticskills.io → search for <query>
   WebFetch: https://skillsmp.com/zh → search for <query>
   ```

2. For GitHub awesome-list directories, read the README directly:
   ```bash
   gh repo view ComposioHQ/awesome-claude-skills --json description
   ```
   Or use WebFetch on `https://github.com/ComposioHQ/awesome-claude-skills` and `https://github.com/VoltAgent/awesome-openclaw-Skills` to scan the skill list.

3. From search results, identify the skill's GitHub repository (`owner/repo`). The directory site will typically link to it.

4. Try to register the repo as a marketplace:
   ```bash
   /plugin marketplace add <owner/repo>
   ```
   If it's a valid marketplace, search/install from there. If not, fall back to `npx skills add`.

5. If nothing is found across all tiers, go to "When No Skills Are Found".
```

- [ ] **Step 2: 验证文件完整性**

```bash
wc -l skills/find-skills/SKILL.md
```

---

### Task 3: 重写安装工作流（两级降级）+ 质量验证

**Files:**
- Modify: `skills/find-skills/SKILL.md`（替换原 Step 4-6 内容）

- [ ] **Step 1: 写入安装工作流**

插入安装工作流章节，替换原有的 Step 4-6：

```markdown
## Install Workflow (Two-Tier Cascade)

Once a skill is found, install it using the best available method.

### Tier 1: Install via /plugin (Preferred)

Claude Code's `/plugin` system provides the best integration — skill updates, dependency management, and auto-loading.

1. **Ensure the marketplace is registered:**
   ```bash
   /plugin marketplace add <owner/repo>
   ```

2. **Install the skill from the marketplace:**
   ```bash
   /plugin install <skill-name>@<marketplace-name>
   ```

3. Verify installation — the skill should appear in the user's available skills list.

### Tier 2: Install via npx skills (Fallback)

Use this when the skill's source is not a valid Claude Code marketplace (no `.claude-plugin/marketplace.json`).

```bash
npx skills add <owner/repo@skill> -g -y
```

Flags:
- `-g` — install globally (user-level)
- `-y` — skip confirmation prompts

### Presenting Installation to the User

When recommending a skill, always present the best installation method first:

```
I found a skill that matches your need!

**Skill:** <name> — <description>
**Source:** <owner/repo> (<installs> installs)

To install (recommended):
  /plugin marketplace add <owner/repo>
  /plugin install <skill>@<marketplace>

Or via npx:
  npx skills add <owner/repo@skill> -g -y
```
```

- [ ] **Step 2: 将质量验证合并到安装工作流后**

保留原 Step 4 "Verify Quality Before Recommending" 内容，放在 Install Workflow 之后，并增强：

```markdown
## Quality Verification

**Do not recommend a skill based solely on search results.** Always verify:

1. **Install count** — Prefer skills with 1K+ installs. Be cautious with anything under 100.
2. **Source reputation** — Official sources (`anthropics`, `vercel-labs`, `huggingface`, `microsoft`, `openclaw`) are more trustworthy than unknown authors.
3. **GitHub stars** — Check the source repository. A skill from a repo with <50 stars should be treated with skepticism.
4. **Has marketplace.json?** — Prefer skills that support `/plugin` installation (have `.claude-plugin/marketplace.json`). This indicates the author follows Claude Code ecosystem conventions.
5. **Recency** — Check last commit date. Skills unmaintained for >1 year may have compatibility issues.

### How to Verify

- For GitHub repos: `gh repo view <owner/repo> --json stargazersCount,updatedAt,description`
- For marketplace skills: Check the marketplace listing for install counts and ratings
- Read the skill's SKILL.md to understand what it actually does before recommending
```

---

### Task 4: 保留并整理其余章节

**Files:**
- Modify: `skills/find-skills/SKILL.md`（末尾部分）

- [ ] **Step 1: 保留 Common Skill Categories、Tips、When No Skills Are Found**

确保以下章节在文件末尾完整保留（从原文件复制），并将 "Tips for Effective Searches" 扩展为包含多源搜索技巧：

```markdown
## Common Skill Categories

When searching, consider these common categories:

| Category        | Example Queries                          |
| --------------- | ---------------------------------------- |
| Web Development | react, nextjs, typescript, css, tailwind |
| Testing         | testing, jest, playwright, e2e           |
| DevOps          | deploy, docker, kubernetes, ci-cd        |
| Documentation   | docs, readme, changelog, api-docs        |
| Code Quality    | review, lint, refactor, best-practices   |
| Design          | ui, ux, design-system, accessibility     |
| Productivity    | workflow, automation, git                |

## Tips for Effective Multi-Source Searches

1. **Use specific keywords**: "react testing" is better than just "testing"
2. **Try alternative terms**: If "deploy" doesn't work, try "deployment" or "ci-cd"
3. **Check marketplace first**: Always start with `/plugin marketplace list` — it's the fastest path
4. **Register markets proactively**: If you haven't registered the Category A markets, do it — they cover 80%+ of available skills
5. **GitHub is your fallback**: `gh search repos "SKILL.md <topic>"` catches skills not listed on any directory
6. **Read before recommending**: Always read the skill's SKILL.md or README to understand what it does
7. **Prefer /plugin install**: It gives the user automatic updates and better integration

## When No Skills Are Found

If no relevant skills exist after searching all tiers:

1. Acknowledge that no existing skill was found
2. Offer to help with the task directly using your general capabilities
3. Suggest the user could create their own skill with `npx skills init`

Example:

```
I searched across registered marketplaces, GitHub, and skill directories
for "xyz" but didn't find any matches. I can still help you with this task
directly! Would you like me to proceed?

If this is something you do often, you could create your own skill:
  npx skills init my-xyz-skill
  /plugin marketplace add <your-repo>  # after pushing to GitHub
```
```

- [ ] **Step 2: 验证完整文件**

```bash
cat skills/find-skills/SKILL.md
```
确认所有章节完整无遗漏，markdown 格式正确。

---

### Task 5: 最终审查和提交

**Files:**
- Modify: `skills/find-skills/SKILL.md`

- [ ] **Step 1: 自审查 — 检查清单**

逐项检查：
- [ ] Frontmatter 格式正确（`---` 包裹，name/description 字段存在）
- [ ] 所有 5 个 Category A 市场已列出，注册命令正确
- [ ] 所有 9 个 Category B 目录已列出，URL 正确
- [ ] 三级搜索降级逻辑清晰，每级有具体命令
- [ ] 两级安装降级逻辑清晰，优先 `/plugin`
- [ ] 质量验证标准明确
- [ ] 无 TBD/TODO/placeholder
- [ ] 无模糊指令

- [ ] **Step 2: Git diff 审查**

```bash
git diff skills/find-skills/SKILL.md
```
确认仅修改了目标文件，变更内容符合预期。

- [ ] **Step 3: 提交**

```bash
git add skills/find-skills/SKILL.md
git commit -m "feat: expand find-skills to multi-source search and install

- Add Category A registrable marketplaces (5) with /plugin commands
- Add Category B skill directory websites (9) for fallback search
- Implement three-tier search cascade: registered markets → GitHub → directories
- Implement two-tier install cascade: /plugin → npx skills add
- Preserve quality verification, categories, and fallback guidance

Co-Authored-By: Claude Opus 4.8 <noreply@anthropic.com>"
```
```

<｜｜DSML｜｜parameter name="content" string="true"># find-skills 多源搜索与安装 实现计划

> **For agentic workers:** REQUIRED SUB-SKILL: Use superpowers:subagent-driven-development (recommended) or superpowers:executing-plans to implement this plan task-by-task. Steps use checkbox (`- [ ]`) syntax for tracking.

**Goal:** 将 find-skills SKILL.md 从单一 skills.sh 来源扩展为多源搜索（已注册市场 → GitHub → 第三方网站）和多级安装（/plugin → npx skills add）

**Architecture:** 纯文档修改。仅修改 `skills/find-skills/SKILL.md`，重写搜索和安装工作流，新增市场知识库章节。不改动任何代码文件。

**Tech Stack:** Markdown + Claude Code skill 指令语法

---

### Task 1: 重写 SKILL.md — Frontmatter 和触发条件

**Files:**
- Modify: `skills/find-skills/SKILL.md:1-19`

- [ ] **Step 1: 更新 frontmatter 和头部章节**

保留现有 frontmatter，增强 description 以反映多源搜索能力。重写 "When to Use This Skill" 和 "What is the Skills CLI?" 为更全面的 "Skill Ecosystem Overview"。

替换文件开头到 "## How to Help Users Find Skills" 之前的内容：

```markdown
---
name: find-skills
description: Helps users discover and install agent skills from multiple sources (registered marketplaces, GitHub, skill directories). Use when users ask "how do I do X", "find a skill for X", "is there a skill that can...", or express interest in extending capabilities. Supports searching across registered Claude Code marketplaces, GitHub repositories, and third-party skill directory websites.
---

# Find Skills

This skill helps you discover and install skills from the entire agent skills ecosystem — registered Claude Code marketplaces, GitHub, and third-party skill directories.

## When to Use This Skill

Use this skill when the user:

- Asks "how do I do X" where X might be a common task with an existing skill
- Says "find a skill for X" or "is there a skill for X"
- Asks "can you do X" where X is a specialized capability
- Expresses interest in extending agent capabilities
- Wants to search for tools, templates, or workflows
- Mentions they wish they had help with a specific domain (design, testing, deployment, etc.)

## Skill Ecosystem Overview

Skills exist across multiple sources. Understanding this landscape is key to finding the right skill.

### Category A: Registrable Claude Code Marketplaces

These are Git repositories containing a `.claude-plugin/marketplace.json` file. They can be registered with `/plugin marketplace add` and then searched/installed via `/plugin` commands.

| Marketplace | Register Command |
|-------------|-----------------|
| Anthropic Official | `/plugin marketplace add anthropics/claude-code` |
| Vercel Skills | `/plugin marketplace add vercel-labs/skills` |
| Hugging Face Skills | `/plugin marketplace add huggingface/skills` |
| ClawHub (OpenClaw) | `/plugin marketplace add openclaw/clawhub` |
| Tencent SkillHub | `/plugin marketplace add tencent-skillhub/registry` |

### Category B: Skill Directory Websites

These websites let you browse skills via web UI. Skills listed on them ultimately point to GitHub repositories. Use WebFetch to search these sites, then install found skills via `npx skills add` (or register the underlying GitHub repo with `/plugin marketplace add` if it's a valid marketplace).

| Directory | URL |
|-----------|-----|
| skills.sh | https://skills.sh |
| SkillsMP | https://skillsmp.com/zh |
| Killer Skills | https://killer-skills.com/zh |
| Agentic Skills | https://agenticskills.io |
| Best Skills | https://bestskills.app |
| Cocoloop Hub | https://hub.cocoloop.cn/ |
| Claw Skills | https://clawskills.sh |
| Awesome OpenClaw (GitHub) | https://github.com/VoltAgent/awesome-openclaw-Skills |
| Awesome Claude (GitHub) | https://github.com/ComposioHQ/awesome-claude-skills |
```

- [ ] **Step 2: 验证文件格式**

```bash
head -5 skills/find-skills/SKILL.md
```
确认 frontmatter 格式正确。

---

### Task 2: 重写搜索工作流（三级降级）

**Files:**
- Modify: `skills/find-skills/SKILL.md`（替换原 "## How to Help Users Find Skills" 章节）

- [ ] **Step 1: 写入搜索工作流**

替换原有的 "## How to Help Users Find Skills" 及其 Step 1-3 内容，写入完整的搜索工作流：

```markdown
## Search Workflow (Three-Tier Cascade)

**Rule:** Only proceed to the next tier if the current tier yields no relevant results.

### Tier 1: Search Registered Marketplaces

Start here — it's fastest and uses already-configured sources.

1. List registered marketplaces:
   ```bash
   /plugin marketplace list
   ```

2. Search for the skill in registered marketplaces. If the marketplace supports a search command, use it; otherwise browse the marketplace listing.

3. If you find a relevant skill, skip to the Install Workflow.

4. If no registered marketplaces exist or no relevant skill is found, proceed to Tier 2.

### Tier 2: Register Known Marketplaces + Search GitHub

If Tier 1 fails, broaden the search.

1. **Register Category A marketplaces** the user doesn't have yet:
   ```bash
   /plugin marketplace add anthropics/claude-code
   /plugin marketplace add vercel-labs/skills
   /plugin marketplace add huggingface/skills
   /plugin marketplace add openclaw/clawhub
   /plugin marketplace add tencent-skillhub/registry
   ```

2. Search again in the newly registered marketplaces.

3. **Also search GitHub** in parallel:
   - Use `gh search repos` to find skill repositories:
     ```bash
     gh search repos "SKILL.md <query>" --limit 20
     ```
   - Or search by topic:
     ```bash
     gh search repos "topic:claude-skills <query>" --limit 20
     ```
   - Or use WebSearch: `site:github.com SKILL.md <query>`

4. **Read GitHub README** of any found repos — they often list a marketplace registration URL you can use with `/plugin marketplace add`.

5. If you find a relevant skill, register its market (if needed) and skip to the Install Workflow.

6. If still no relevant skill found, proceed to Tier 3.

### Tier 3: Search Category B Directory Websites

Last resort — crawl skill directory sites.

1. Use WebFetch to search each Category B site. Start with the most popular ones:

   ```
   WebFetch: https://skills.sh/ → search for <query>
   WebFetch: https://agenticskills.io → search for <query>
   WebFetch: https://skillsmp.com/zh → search for <query>
   ```

2. For GitHub awesome-list directories, read the README directly:
   ```bash
   gh repo view ComposioHQ/awesome-claude-skills --json description
   ```
   Or use WebFetch on `https://github.com/ComposioHQ/awesome-claude-skills` and `https://github.com/VoltAgent/awesome-openclaw-Skills` to scan the skill list.

3. From search results, identify the skill's GitHub repository (`owner/repo`). The directory site will typically link to it.

4. Try to register the repo as a marketplace:
   ```bash
   /plugin marketplace add <owner/repo>
   ```
   If it's a valid marketplace, search/install from there. If not, fall back to `npx skills add`.

5. If nothing is found across all tiers, go to "When No Skills Are Found".
```

- [ ] **Step 2: 验证文件完整性**

```bash
wc -l skills/find-skills/SKILL.md
```

---

### Task 3: 重写安装工作流（两级降级）+ 质量验证

**Files:**
- Modify: `skills/find-skills/SKILL.md`（替换原 Step 4-6 内容）

- [ ] **Step 1: 写入安装工作流**

插入安装工作流章节，替换原有的 Step 4-6：

```markdown
## Install Workflow (Two-Tier Cascade)

Once a skill is found, install it using the best available method.

### Tier 1: Install via /plugin (Preferred)

Claude Code's `/plugin` system provides the best integration — skill updates, dependency management, and auto-loading.

1. **Ensure the marketplace is registered:**
   ```bash
   /plugin marketplace add <owner/repo>
   ```

2. **Install the skill from the marketplace:**
   ```bash
   /plugin install <skill-name>@<marketplace-name>
   ```

3. Verify installation — the skill should appear in the user's available skills list.

### Tier 2: Install via npx skills (Fallback)

Use this when the skill's source is not a valid Claude Code marketplace (no `.claude-plugin/marketplace.json`).

```bash
npx skills add <owner/repo@skill> -g -y
```

Flags:
- `-g` — install globally (user-level)
- `-y` — skip confirmation prompts

### Presenting Installation to the User

When recommending a skill, always present the best installation method first:

```
I found a skill that matches your need!

**Skill:** <name> — <description>
**Source:** <owner/repo> (<installs> installs)

To install (recommended):
  /plugin marketplace add <owner/repo>
  /plugin install <skill>@<marketplace>

Or via npx:
  npx skills add <owner/repo@skill> -g -y
```
```

- [ ] **Step 2: 将质量验证合并到安装工作流后**

保留原 Step 4 "Verify Quality Before Recommending" 内容，放在 Install Workflow 之后，并增强：

```markdown
## Quality Verification

**Do not recommend a skill based solely on search results.** Always verify:

1. **Install count** — Prefer skills with 1K+ installs. Be cautious with anything under 100.
2. **Source reputation** — Official sources (`anthropics`, `vercel-labs`, `huggingface`, `microsoft`, `openclaw`) are more trustworthy than unknown authors.
3. **GitHub stars** — Check the source repository. A skill from a repo with <50 stars should be treated with skepticism.
4. **Has marketplace.json?** — Prefer skills that support `/plugin` installation (have `.claude-plugin/marketplace.json`). This indicates the author follows Claude Code ecosystem conventions.
5. **Recency** — Check last commit date. Skills unmaintained for >1 year may have compatibility issues.

### How to Verify

- For GitHub repos: `gh repo view <owner/repo> --json stargazersCount,updatedAt,description`
- For marketplace skills: Check the marketplace listing for install counts and ratings
- Read the skill's SKILL.md to understand what it actually does before recommending
```

---

### Task 4: 保留并整理其余章节

**Files:**
- Modify: `skills/find-skills/SKILL.md`（末尾部分）

- [ ] **Step 1: 保留 Common Skill Categories、Tips、When No Skills Are Found**

确保以下章节在文件末尾完整保留，并将 "Tips for Effective Searches" 扩展为包含多源搜索技巧：

```markdown
## Common Skill Categories

When searching, consider these common categories:

| Category        | Example Queries                          |
| --------------- | ---------------------------------------- |
| Web Development | react, nextjs, typescript, css, tailwind |
| Testing         | testing, jest, playwright, e2e           |
| DevOps          | deploy, docker, kubernetes, ci-cd        |
| Documentation   | docs, readme, changelog, api-docs        |
| Code Quality    | review, lint, refactor, best-practices   |
| Design          | ui, ux, design-system, accessibility     |
| Productivity    | workflow, automation, git                |

## Tips for Effective Multi-Source Searches

1. **Use specific keywords**: "react testing" is better than just "testing"
2. **Try alternative terms**: If "deploy" doesn't work, try "deployment" or "ci-cd"
3. **Check marketplace first**: Always start with `/plugin marketplace list` — it's the fastest path
4. **Register markets proactively**: If you haven't registered the Category A markets, do it — they cover 80%+ of available skills
5. **GitHub is your fallback**: `gh search repos "SKILL.md <topic>"` catches skills not listed on any directory
6. **Read before recommending**: Always read the skill's SKILL.md or README to understand what it does
7. **Prefer /plugin install**: It gives the user automatic updates and better integration

## When No Skills Are Found

If no relevant skills exist after searching all tiers:

1. Acknowledge that no existing skill was found
2. Offer to help with the task directly using your general capabilities
3. Suggest the user could create their own skill with `npx skills init`

Example:

```
I searched across registered marketplaces, GitHub, and skill directories
for "xyz" but didn't find any matches. I can still help you with this task
directly! Would you like me to proceed?

If this is something you do often, you could create your own skill:
  npx skills init my-xyz-skill
  /plugin marketplace add <your-repo>  # after pushing to GitHub
```
```

- [ ] **Step 2: 验证完整文件**

```bash
cat skills/find-skills/SKILL.md
```
确认所有章节完整无遗漏，markdown 格式正确。

---

### Task 5: 最终审查和提交

**Files:**
- Modify: `skills/find-skills/SKILL.md`

- [ ] **Step 1: 自审查 — 检查清单**

逐项检查：
- Frontmatter 格式正确（`---` 包裹，name/description 字段存在）
- 所有 5 个 Category A 市场已列出，注册命令正确
- 所有 9 个 Category B 目录已列出，URL 正确
- 三级搜索降级逻辑清晰，每级有具体命令
- 两级安装降级逻辑清晰，优先 `/plugin`
- 质量验证标准明确
- 无 TBD/TODO/placeholder
- 无模糊指令（如 "add error handling" 之类）

- [ ] **Step 2: Git diff 审查**

```bash
git diff skills/find-skills/SKILL.md
```
确认仅修改了目标文件，变更内容符合预期。

- [ ] **Step 3: 提交**

```bash
git add skills/find-skills/SKILL.md
git commit -m "feat: expand find-skills to multi-source search and install

- Add Category A registrable marketplaces (5) with /plugin commands
- Add Category B skill directory websites (9) for fallback search
- Implement three-tier search cascade: registered markets → GitHub → directories
- Implement two-tier install cascade: /plugin → npx skills add
- Preserve quality verification, categories, and fallback guidance

Co-Authored-By: Claude Opus 4.8 <noreply@anthropic.com>"
```
```
