# AGENTS.md

This file provides guidance to WARP (warp.dev) when working with code in this repository.

## What this repo is
This is a **Claude Code plugin** that ships a set of **Airtable-focused skills** (mostly Markdown) plus a bundled MCP configuration for Airtable.

The “runtime” behavior lives in the skill docs themselves (e.g. `SKILL.md` instructions, referenced docs under `reference/`, and example snippets). There is no application code to build.

## Common commands / workflows
### Install & use the plugin (from README)
Claude Code (CLI):
```bash
# Add the marketplace
/plugin marketplace add robweidner/airtable-skills

# Install the plugin
/plugin install airtable@robweidner-airtable
```

For development/testing (load without installing):
```bash
git clone https://github.com/robweidner/airtable-skills.git
claude --plugin-dir ./airtable-skills/plugins/airtable
```

### MCP setup expectation
The plugin ships `plugins/airtable/.mcp.json`, which expects an environment variable:
```bash
export AIRTABLE_PAT=patXXXXXXXXXXXXXX
```
This should be added to the user's shell profile (`~/.zshrc` or `~/.bashrc`).

### "Build / lint / test"
There are **no repo-provided** build, lint, or test commands/scripts.

Practical validation commands used during edits:
```bash
# Validate the plugin manifest and MCP config are valid JSON
jq -e . plugins/airtable/.claude-plugin/plugin.json
jq -e . plugins/airtable/.mcp.json

# Full plugin validation (requires Claude Code)
claude plugin validate .
```

## High-level architecture
### Top-level docs
- `README.md`: primary user-facing documentation (installation, skill list, reference file links).
- `CHANGELOG.md`: release notes; keep this aligned with version bumps.
- `docs/emoji-naming-conventions.md`: team-specific emoji/naming guidance (separate from the skill’s `[emoji]` bracket system, but conceptually related).

### Plugin root
- `plugins/airtable/.claude-plugin/plugin.json`: plugin manifest (name/version/metadata). **Must** be inside `.claude-plugin/` per Claude Code plugin spec.
- `plugins/airtable/.mcp.json`: bundled MCP server configuration that runs `airtable-mcp-server` via `npx` and passes through `AIRTABLE_PAT`.

### Skills layout
Skills live under `plugins/airtable/skills/<skill-name>/`.

Each skill is defined primarily by its `SKILL.md`:
- YAML frontmatter declares `name` and `description`.
- Some skills include `disable-model-invocation: true` in frontmatter (treat as a signal that the skill is designed to be highly procedural / tool-driven).

Key skills in this repo:
- `plugins/airtable/skills/airtable/`: the main “auto-invoking” Airtable assistant.
  - `reference/`: canonical reference docs the main skill tells the agent to “Load” (MCP patterns, REST API patterns, field types, automations, interface designer, token security, emoji conventions).
  - `examples/`: longer-form script examples (e.g. `examples/scripting-patterns.md`).
- `plugins/airtable/skills/airtable-extensions/`: guidance for building Airtable extensions (Blocks SDK + Interface Extensions SDK), including data source configuration, `block run` troubleshooting, Chrome CORS fixes, auto-release hook recipe, and its own `reference/`.
- `plugins/airtable/skills/airtable-field-audit/`: field naming audit workflow.
- `plugins/airtable/skills/airtable-base-audit/`: base health scoring/audit workflow.

## Editing guidelines (repo-specific)
### When changing or adding skills
- Most changes are **content edits** to the relevant `SKILL.md` and (if needed) the corresponding `reference/*.md` docs.
- Keep cross-file pointers in sync:
  - The README skill table (“What’s Included”) should match the actual skill set under `plugins/airtable/skills/`.
  - The README “Reference Files” section should match what exists under `plugins/airtable/skills/airtable/reference/`.

### Versioning / releases
When making user-visible changes:
- Bump **all three** version locations in lockstep:
  - `plugins/airtable/.claude-plugin/plugin.json` → `version`
  - `.claude-plugin/marketplace.json` → both `metadata.version` and `plugins[0].version`
  - `README.md` → version badge text
- Add an entry to `CHANGELOG.md` (the repo uses dated headings like `## [1.5.0] - 2026-02-08`).
- Tag the release: `git tag -a v1.X.0 -m "v1.X.0 - Short description"` and `git push origin v1.X.0`.
- Create a GitHub release: `gh release create v1.X.0 --title "v1.X.0" --notes "..."`.

### Claude Code hook for auto-release
The `airtable-extensions` skill documents a `PostToolUse` hook pattern that auto-runs `npx block release` after every `git push` in an extension repo. If you're editing the hook recipe in the skill, keep these constraints accurate:
- The hook fires on **all** Bash tool calls and filters to `git push` commands only.
- It checks the working directory or command path to scope to a specific extension repo.
- It skips `block release` if the push output contains error indicators.
- Timeout should be 60s (network call to Airtable block registry).

### Preferences supported by the plugin
The plugin docs mention persisted user preferences (kept in a user’s personal `CLAUDE.md`), notably:
- `airtable_experience: beginner | power-user | developer`
- `airtable_emoji_mode: auto | new-only | ask | none`
- `airtable_script_style: minimal | comprehensive`
- `airtable_use_field_ids: true | false`

Behavior tied to these preferences that’s easy to accidentally break:
- When writing Airtable scripts, the docs prefer **minimal scripts** that output data for native automation steps (instead of doing everything in code).
- When `airtable_use_field_ids: true`, the docs prefer storing `Field ID: fld...` in field descriptions after creation and using field IDs in scripts/integrations.

If you change preference behavior or add new preferences, update:
- `plugins/airtable/CLAUDE.md` (plugin-level docs)
- The main `plugins/airtable/skills/airtable/SKILL.md`
- Any referenced docs that describe the preference (e.g. emoji conventions)

### Airtable API limitations baked into the docs
Several skills encode Airtable API/MCP limitations and associated workarounds. If you change these, update both the skill instructions and references:
- Bases cannot be created via API/MCP (user must create in UI and provide `app...` base ID).
- Automations cannot be created via API/MCP (docs generate ASCII diagrams + click-by-click setup steps).
- Formula/rollup/lookup fields cannot be created directly (docs use a 🔧 marker + put the intended config in the field description).
