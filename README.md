# skills

Claude / AI agent skills used across Lync Systems projects, organized by project:

```
<project>/<skill-name>/SKILL.md
```

## Projects

- [`feedlab/`](feedlab) — skills for working with [FeedLab](https://feedlab.cloud): designing test cases, reviewing code for issues, and running security reviews, all registered back into FeedLab via its MCP server or API. See [feedlab.cloud/ai](https://feedlab.cloud/ai).

Each skill is a standard [Claude Code skill](https://docs.claude.com/en/docs/claude-code/skills) — a `SKILL.md` with YAML frontmatter (`name`, `description`) a client triggers on, plus instructions in the body. This repo is private, so grab a skill with `gh` (authenticated) rather than a plain `curl`:

```bash
mkdir -p ~/.claude/skills/<skill-name>
gh api repos/lync-systems-mw/skills/contents/<project>/<skill-name>/SKILL.md \
  --jq '.content' | base64 -d > ~/.claude/skills/<skill-name>/SKILL.md
```

FeedLab's own skills also ship publicly, no auth needed, at `feedlab.cloud/skills/<skill-name>/SKILL.md` (see [feedlab.cloud/ai](https://feedlab.cloud/ai)) — use that instead when you don't need the private copy here.
