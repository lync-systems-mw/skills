# skills

Claude / AI agent skills used across Lync Systems projects, organized by project:

```
<project>/<skill-name>/SKILL.md
```

## Projects

- [`feedlab/`](feedlab) — skills for working with [FeedLab](https://feedlab.cloud): designing test cases, reviewing code for issues, and running security reviews, all registered back into FeedLab via its MCP server or API. See [feedlab.cloud/ai](https://feedlab.cloud/ai).

Each skill is a standard [Claude Code skill](https://docs.claude.com/en/docs/claude-code/skills) — a `SKILL.md` with YAML frontmatter (`name`, `description`) a client triggers on, plus instructions in the body. Install one with:

```bash
mkdir -p ~/.claude/skills/<skill-name>
curl -s https://raw.githubusercontent.com/lync-systems-mw/skills/main/<project>/<skill-name>/SKILL.md \
  -o ~/.claude/skills/<skill-name>/SKILL.md
```

FeedLab's skills also ship from `feedlab.cloud/skills/<skill-name>/SKILL.md` (see [feedlab.cloud/ai](https://feedlab.cloud/ai)) — same content, either source works.

## Adding a project

This repo is public, so treat it as a shared surface: review what a skill tells an agent to do before adding it here, same as any other public-facing code. Keep anything project-specific but not meant for public eyes (internal URLs, non-public workflows) out of the `SKILL.md` body.
