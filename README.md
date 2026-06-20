# claude-skill-collection

A collection of [Claude Code](https://claude.com/claude-code) skills.

Each skill packages a focused, repeatable workflow that Claude Code can invoke by intent. Skills live in their own top-level folder, with the skill source (`SKILL.md` + `references/`) and a packaged `.skill` archive.

## Skills

| Folder | Description |
|---|---|
| [`AIOps/`](AIOps/) | A three-session AIOps consulting engagement — analysis, architecture, and effort estimation — each producing markdown working files and a branded HTML report. |

See each folder's own `README.md` for details on what the skills do and how to use them.

## Repository conventions

- Each skill is a directory containing a `SKILL.md` (YAML frontmatter + workflow instructions) and a `references/` folder with the detailed prompts.
- The `.skill` files are ZIP archives of the skill source, used for distribution.

## Development

See [CLAUDE.md](CLAUDE.md) for guidance on authoring skills, packaging `.skill` archives, and running evaluations.
