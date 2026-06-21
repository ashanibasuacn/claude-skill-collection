# claude-skill-collection

A collection of [Claude Code](https://claude.com/claude-code) skills.

Each skill packages a focused, repeatable workflow that Claude Code can invoke by intent. Skills live in their own top-level folder, with the skill source (`SKILL.md` + optional `references/` and `scripts/`) and a packaged `.skill` archive ready for installation.

## Skills

| Folder | Skill | Description |
|---|---|---|
| [`AIOps/`](AIOps/) | `aiops-analysis` · `aiops-architecture` · `aiops-tep` | A three-session AIOps consulting engagement — use-case discovery, platform architecture, and effort estimation — each producing markdown working files and a branded HTML report. |
| [`sdlc/`](sdlc/) | `requirements-docx` | Converts a requirements markdown file into a formal IEEE 830 Software Requirements Specification (DOCX) with cover page, version history, TOC, numbered sections, and appendices. |

See each folder's `README.md` for details on what the skills do and how to use them.

---

## Installing a skill

Skills are distributed as `.skill` files (renamed ZIP archives). Install them from the Claude Code command palette or the CLI.

### Option 1 — Claude Code UI (recommended)

1. Open Claude Code.
2. Press `Ctrl+Shift+P` (Windows/Linux) or `Cmd+Shift+P` (Mac) to open the command palette.
3. Type **Install skill** and select it.
4. Browse to the `.skill` file and confirm.

Claude Code extracts the skill and registers it. It will appear in your available skills list immediately.

### Option 2 — CLI

```bash
claude skill install path/to/skill-name.skill
```

To list installed skills:
```bash
claude skill list
```

To remove a skill:
```bash
claude skill remove skill-name
```

### Option 3 — Manual installation

A `.skill` file is a standard ZIP archive. You can also install by extracting it directly:

```bash
# Unzip into your Claude Code skills directory
# Windows (PowerShell)
Expand-Archive -Path skill-name.skill -DestinationPath "$env:USERPROFILE\.claude\skills\skill-name"

# Mac / Linux
unzip skill-name.skill -d ~/.claude/skills/skill-name
```

Restart Claude Code after manual extraction.

---

## Using installed skills

Once installed, skills activate automatically when your request matches the skill's trigger description. You do not need to type a slash command.

**Examples:**
- *"I need to generate a formal requirements document from my MD file"* → triggers `requirements-docx`
- *"Let's start an AIOps engagement"* → triggers `aiops-analysis`

You can also invoke a skill explicitly with the `/` command if the skill name appears in your installed list.

---

## Repository conventions

- Each skill directory contains a `SKILL.md` (YAML frontmatter + workflow instructions), an optional `references/` folder for detailed reference docs, and an optional `scripts/` folder for bundled automation scripts.
- `.skill` files are ZIP archives of the skill source directory, used for distribution and installation.
- Each skill family has a `*-workspace/` sibling directory holding evaluation test cases, benchmark results, and iteration outputs.

## Development

See [CLAUDE.md](CLAUDE.md) for guidance on authoring skills, packaging `.skill` archives, and running evaluations.
