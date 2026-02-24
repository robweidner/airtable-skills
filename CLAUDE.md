# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## What This Is

A Claude Code plugin shipping Airtable-focused skills as Markdown files plus a bundled MCP configuration. There is no application code — the "runtime" lives entirely in `SKILL.md` files and their referenced docs.

## Validation Commands

No build, lint, or test scripts exist. Use these for validation:

```bash
# Validate JSON manifests
jq -e . plugins/airtable/.claude-plugin/plugin.json
jq -e . plugins/airtable/.mcp.json

# Full plugin validation (requires Claude Code CLI)
claude plugin validate .
```

## Repository Layout

```
plugins/airtable/
├── .claude-plugin/plugin.json    # Plugin manifest (name, version, metadata)
├── .mcp.json                     # Bundled MCP server config (AIRTABLE_PAT env var)
├── CLAUDE.md                     # Plugin-level docs (user preferences, limitations)
└── skills/
    ├── airtable/                 # Main auto-invoking skill
    │   ├── SKILL.md
    │   ├── reference/*.md        # 8 canonical reference docs
    │   └── examples/             # Script examples
    ├── airtable-extensions/      # React extensions skill (manual invoke)
    │   ├── SKILL.md
    │   └── reference/*.md
    ├── airtable-field-audit/     # Field naming audit (manual invoke)
    │   └── SKILL.md
    └── airtable-base-audit/      # Base health scoring (manual invoke)
        └── SKILL.md
```

Top-level: `README.md`, `CHANGELOG.md`, `AGENTS.md`, `.claude-plugin/marketplace.json`

## Key Conventions

### Versioning — Three-Place Lockstep
Every user-visible change must bump version in all three places simultaneously:
1. `plugins/airtable/.claude-plugin/plugin.json` → `version`
2. `.claude-plugin/marketplace.json` → `metadata.version` AND `plugins[0].version`
3. `README.md` → version badge

### Release Workflow
1. Bump versions (above)
2. Add dated entry to `CHANGELOG.md` (format: `## [X.Y.Z] - YYYY-MM-DD`)
3. Tag: `git tag -a vX.Y.Z -m "vX.Y.Z - Description"` + `git push origin vX.Y.Z`
4. GitHub release: `gh release create vX.Y.Z --title "vX.Y.Z" --notes "..."`

### Cross-File Sync Requirements
When adding/removing skills or reference files, update:
- README skill table ("What's Included")
- README "Reference Files" table
- `CHANGELOG.md`

### Skill File Format
Each `SKILL.md` uses YAML frontmatter with `name`, `description`, and optionally `disable-model-invocation: true` (for procedural/manual-only skills).

### Commit Messages
Format: `<type>: <description>` (e.g., `docs:`, `fix:`, `feat:`, `chore:`)

## Airtable API Limitations Encoded in Skills

These constraints are baked into skill instructions and workarounds — don't break them:
- **Bases**: Cannot create via API (user provides `appXXX` ID)
- **Automations**: Cannot create via API (skills generate ASCII diagrams + setup steps)
- **Formula/Rollup/Lookup fields**: Cannot create directly (workaround: create as text with `[convert]` marker, put formula in description, output manual conversion checklist)

## User Preferences System

Defined in `plugins/airtable/CLAUDE.md` and referenced throughout the main skill:
- `airtable_experience`: beginner | power-user | developer
- `airtable_emoji_mode`: auto | new-only | ask | none
- `airtable_script_style`: minimal | comprehensive
- `airtable_use_field_ids`: true | false

Changes to preference behavior must update both `plugins/airtable/CLAUDE.md` and the main `SKILL.md`.

## Context Management

Use subagents (Task tool) for exploration and research to keep the main context clean. The SKILL.md and reference files in this repo are large — reading them all directly burns context fast.

**Spawn a subagent for:** codebase exploration (3+ files), PR reviews, cross-file consistency checks, researching SDK docs
**Stay in main context for:** direct file edits, 1-2 file reads, back-and-forth with the user
