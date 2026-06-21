# claude-skill-collection

A collection of [Claude Code](https://claude.com/claude-code) skills, distributed as the **`ppes-rde`** plugin.

Each skill packages a focused, repeatable workflow that Claude Code invokes by intent. The skill source lives under [`skills/`](skills/) (one folder per skill, each with a `SKILL.md` plus optional `references/` and `scripts/`); installing the plugin makes all of them available under the `ppes-rde` namespace.

## Skills

| Family | Skill | Description |
|---|---|---|
| [`AIOps/`](AIOps/) | `aiops-analysis` · `aiops-architecture` · `aiops-tep` | A three-session AIOps consulting engagement — use-case discovery, platform architecture, and effort estimation — each producing markdown working files and a branded HTML report. |
| [`sdlc/`](sdlc/) | `requirements-docx` | Converts a requirements markdown file into a formal IEEE 830 Software Requirements Specification (DOCX) with cover page, version history, TOC, numbered sections, and appendices. |

See each folder's `README.md` for details on what the skills do and how to use them.

---

## Installing — the `ppes-rde` plugin

All skills in this repo ship as a single Claude Code plugin named **`ppes-rde`**. Installing the plugin installs every skill at once; they become available under the `ppes-rde` namespace (e.g. `ppes-rde:aiops-analysis`, `ppes-rde:requirements-docx`).

### Quick install (two commands)

Inside Claude Code, run:

```
/plugin marketplace add ashanibasuacn/claude-skill-collection
/plugin install ppes-rde@ppes-rde
```

1. The first command registers this repo as a plugin marketplace (Claude Code reads `.claude-plugin/marketplace.json`).
2. The second installs the `ppes-rde` plugin and all its skills.

That's it — the four skills appear in your skills list immediately. No manual ZIP extraction, no per-skill steps.

### Install from a local clone

If you've cloned this repo locally, point the marketplace at the folder instead of GitHub:

```
/plugin marketplace add C:/GIT/claude-skill-collection
/plugin install ppes-rde@ppes-rde
```

### Managing the plugin

```
/plugin                      # browse, enable, or disable installed plugins
/plugin marketplace update ppes-rde   # pull the latest skills after a repo update
/plugin uninstall ppes-rde@ppes-rde
```

---

## Using the skills

Once installed, skills activate automatically when your request matches a skill's trigger description — no slash command needed.

**Examples:**
- *"I need to generate a formal requirements document from my MD file"* → triggers `ppes-rde:requirements-docx`
- *"Let's start an AIOps engagement"* → triggers `ppes-rde:aiops-analysis`

You can also invoke a skill explicitly: `/ppes-rde:aiops-analysis`.

---

## Repository conventions

- The plugin is defined by `.claude-plugin/plugin.json` (manifest, name `ppes-rde`) and `.claude-plugin/marketplace.json` (makes the repo installable as a marketplace).
- `skills/` is the single source of truth — the skills the plugin installs. Each skill directory contains a `SKILL.md` (YAML frontmatter + workflow instructions), an optional `references/` folder for detailed reference docs, and an optional `scripts/` folder for bundled automation scripts.
- `AIOps/` and `sdlc/` hold family-level READMEs and the local `*-workspace/` eval directories (gitignored) — the engagement docs and benchmark data that don't ship with the plugin.

## Development

Edit skills in place under [`skills/`](skills/) — the plugin serves them directly, with no build step. See [CLAUDE.md](CLAUDE.md) for skill-authoring guidance, the plugin layout, and the evaluation workflow.
