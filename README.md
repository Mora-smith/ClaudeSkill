# mora-smith-skills

Personal Claude Code skill collection — 个人 Claude Code 技能集合。

## Installation

In Claude Code, run:

```
/plugin marketplace add Mora-smith/ClaudeSkill
/plugin install find-skills@mora-smith-skills
```

## Skills

| Skill | Description |
|-------|-------------|
| find-skills | Discover and install agent skills from the open ecosystem |

## Add New Skills

1. Create a new directory under `skills/`, add a `SKILL.md` file
2. Update `.claude-plugin/marketplace.json` — append the skill path to the relevant plugin's `skills` array
3. `git commit && git push`

Then users can update:

```
/plugin update find-skills@mora-smith-skills
```
