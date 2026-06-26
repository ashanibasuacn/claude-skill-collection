# claude-skill-collection

A collection of [Claude Code](https://claude.com/claude-code) skills, distributed as the **`ppes-rde`** plugin.

Each skill packages a focused, repeatable workflow that Claude Code invokes by intent. The skill source lives under [`skills/`](skills/) (one folder per skill, each with a `SKILL.md` plus optional `references/` and `scripts/`); installing the plugin makes all of them available under the `ppes-rde` namespace.

## Skills

All skills live under [`skills/`](skills/), one folder each.

| Family | Skills | Description |
|---|---|---|
| AIOps engagement | [`aiops-analysis`](skills/aiops-analysis/) · [`aiops-architecture`](skills/aiops-architecture/) · [`aiops-tep`](skills/aiops-tep/) | A three-session AIOps consulting engagement — use-case discovery, platform architecture, and effort estimation — each producing markdown working files and a branded HTML report. |
| SDLC documents | [`requirements-docx`](skills/requirements-docx/) · [`technical-discovery`](skills/technical-discovery/) | Turn markdown into formal Word deliverables: `requirements-docx` produces an IEEE 830 Software Requirements Specification from a requirements file; `technical-discovery` consolidates one or more discovery notes into a Current Landscape / Technical Discovery document. Both detect structure, confirm the plan, and flag gaps as TBD rather than inventing content. |

See each skill's `SKILL.md` under [`skills/`](skills/) for what it does and how it works.

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

That's it — all skills appear in your skills list immediately. No manual ZIP extraction, no per-skill steps.

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

### Manual install (single skill, per-project)

If you only need one skill in a specific project, you can copy it directly into that project's `.claude/` folder without installing the plugin:

1. Copy the skill folder from `skills/<skill-name>/` into `.claude/skills/<skill-name>/` at the root of your target project:

   ```powershell
   # Example: install just requirements-docx into your project
   Copy-Item -Recurse skills/requirements-docx C:/your-project/.claude/skills/requirements-docx
   ```

   ```bash
   # Or with bash
   cp -r skills/requirements-docx /your-project/.claude/skills/requirements-docx
   ```

2. The skill is immediately available in that project without a namespace prefix — invoke it as `/requirements-docx` or let it trigger automatically.

This approach is project-scoped (the skill won't appear in other projects) and requires no plugin machinery. To update, re-copy the folder.

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
- Evaluation runs, sample inputs, and benchmark data are kept in local, untracked working directories (not committed) and don't ship with the plugin.

## Development

Edit skills in place under [`skills/`](skills/) — the plugin serves them directly, with no build step. See [CLAUDE.md](CLAUDE.md) for skill-authoring guidance, the plugin layout, and the evaluation workflow.
