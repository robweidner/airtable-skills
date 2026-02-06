# Airtable Skills for Claude Code

> Turn Claude into an Airtable expert — schema design, scripting, automations, interfaces, and API integrations.

[![Claude Code Plugin](https://img.shields.io/badge/Claude_Code-Plugin-6366f1)](https://claude.ai/code)
[![Version](https://img.shields.io/badge/version-1.2.0-blue)](https://github.com/robweidner/airtable-skills/releases)
[![License: MIT](https://img.shields.io/badge/License-MIT-green.svg)](LICENSE)

## What's Included

| Skill | Trigger | What It Does |
|-------|---------|--------------|
| **airtable** | Auto-invokes on Airtable work | Full Airtable assistant — schema, scripts, automations, interfaces, API |
| **airtable-extensions** | `/airtable-extensions` | Custom React extensions (Blocks SDK) |
| **airtable-field-audit** | `/airtable-field-audit` | Find and fix field naming issues — duplicates, whitespace, similar names |

## Quick Start

### Installation

**Claude Code (CLI):**
```bash
# Add the marketplace
/plugin marketplace add robweidner/airtable-skills

# Install the plugin
/plugin install airtable@robweidner-airtable
```

**Claude Cowork (Web):**
1. Visit [claude.com/plugins](https://claude.com/plugins/)
2. Search for "Airtable"
3. Click Install

**Manual Clone:**
```bash
cd ~/.claude/plugins
git clone https://github.com/robweidner/airtable-skills.git airtable
```

> **Note:** This plugin includes an `.mcp.json` that auto-configures the Airtable MCP server. Just set your `AIRTABLE_PAT` environment variable.

### Setup Your Token

1. Go to [airtable.com/create/tokens](https://airtable.com/create/tokens)
2. Create a scoped token for your project (don't use a global token)
3. Select only the base(s) you need
4. Add scopes: `data.records:read`, `data.records:write`, `schema.bases:read`, `schema.bases:write`
5. Store in `.secrets.env`:
   ```bash
   AIRTABLE_PAT=patXXXXXXXXXXXXXX
   ```

### First Use

The skill asks for your preferences on first run:

| Preference | Options | Description |
|------------|---------|-------------|
| Experience | `beginner` / `power-user` / `developer` | Controls explanation depth |
| Emoji Mode | `auto` / `new-only` / `ask` / `none` | Field name emoji prefixes |
| Script Style | `minimal` / `comprehensive` | Output data vs. all-in-one scripts |
| Field IDs | `true` / `false` | Store IDs in descriptions for robust scripts |

Save to your `CLAUDE.md`:
```markdown
## Airtable Preferences
airtable_experience: developer
airtable_emoji_mode: auto
airtable_script_style: minimal
airtable_use_field_ids: true
```

## Capabilities

### What You Can Do

| Task | How It Works |
|------|--------------|
| **Create tables & fields** | API creates schema, suggests standard fields (timestamps, audit) |
| **All 28 field types** | Full support matrix with API parameters |
| **Write scripts** | Airtable scripting SDK patterns, adapts to experience level |
| **Build interfaces** | Interface Designer guidance, component patterns |
| **Set up automations** | ASCII diagrams + click-by-click instructions |
| **API integrations** | REST API patterns, rate limiting, batch operations |
| **React extensions** | Full Blocks SDK with hooks and component patterns |

### API Limitations (with Workarounds)

The Airtable API has known limitations. This skill handles them automatically:

| Can't Create via API | Workaround |
|---------------------|------------|
| **Bases** | User creates manually, provides base ID |
| **Formula fields** | Creates as text with `[convert]` marker, formula in description |
| **Rollup fields** | Same as formula |
| **Lookup fields** | Same as formula |
| **Automations** | Generates ASCII flow diagrams + setup steps |

After API operations, you get a checklist of manual conversions needed.

### Emoji Naming System

Visual field identification at a glance:

```
📝 Name           [text]
📧 Email          [email]
🔢 Quantity       [number]
💰 Total          [currency]
📅 Due Date       [date]
🔗 Company        [link]
📊 Status         [select]
🧮 Full Name      [formula]
```

Stack markers for context: `[wip][formula] Commission Calc`

## Example Prompts

**Schema Design:**
```
Create a Contacts table with name, email, phone, company, and status fields
```

**Scripting:**
```
Write a script to find all Pending tasks and update them to In Progress
```

**Automations:**
```
Set up an automation that sends Slack when a new order is created over $1000
```

**Interfaces:**
```
Design an interface for sales reps to manage their assigned leads
```

**Extensions:**
```
/airtable-extensions
Build a custom extension to visualize sales pipeline as a Kanban board
```

## Documentation

### Reference Files

| File | When to Use |
|------|-------------|
| [field-types.md](skills/airtable/reference/field-types.md) | Creating/modifying fields |
| [emoji-conventions.md](skills/airtable/reference/emoji-conventions.md) | Naming conventions |
| [api-patterns.md](skills/airtable/reference/api-patterns.md) | REST API operations |
| [mcp-patterns.md](skills/airtable/reference/mcp-patterns.md) | Airtable MCP tools |
| [scripting-api.md](skills/airtable/reference/scripting-api.md) | Writing scripts |
| [automations.md](skills/airtable/reference/automations.md) | Automation setup |
| [interface-designer.md](skills/airtable/reference/interface-designer.md) | Building interfaces |
| [pat-security.md](skills/airtable/reference/pat-security.md) | Token security |

### External Resources

- [Airtable Web API](https://airtable.com/developers/web/api/introduction)
- [Airtable Scripting](https://airtable.com/developers/scripting)
- [Airtable Extensions SDK](https://airtable.com/developers/extensions)

## Why Field IDs?

Field names change. Field IDs don't.

When `airtable_use_field_ids: true`, the skill:
1. Stores field IDs in descriptions after creation: `Field ID: fldXXXXXXXXXXXXXX`
2. Uses IDs in scripts instead of names
3. Your scripts survive field renames

## Airtable MCP (Optional)

If you have the Airtable MCP configured, the skill prefers it over REST API. Works fine without MCP — falls back to REST API with rate limiting.

## Common Gotchas

| Issue | Solution |
|-------|----------|
| "Base ID required" | Create base in Airtable UI, provide `appXXXXXX` ID |
| Field names are case-sensitive | Use field IDs instead |
| Linked records fail | Create the linked table first |
| Formula field not created | Use the `[convert]` workaround |
| Rate limited | Skill handles this — uses exponential backoff |

## Contributing

This plugin belongs to the community. Here's how you can get involved:

### Improve This Plugin

Fork it, make it better, and submit a PR:

- **Add scripting patterns** — `examples/scripting-patterns.md`
- **Update API docs** — as Airtable changes
- **New automation templates** — `reference/automations.md`
- **Bug fixes and improvements**

### Build Your Own Skills

The best part of Claude Code skills? **You can create your own.**

1. Fork this repo as a starting point
2. Modify the skills in `skills/` to match your workflow
3. Add new skills for your specific Airtable use cases
4. Keep it private or share it back with the community

**Pro tip:** Ask Claude to help you write skills. Literally just say:

```
Help me create a Claude Code skill for [your use case]
```

Claude understands the skill format and can generate new skills, modify existing ones, or help you debug issues. You don't need to be a developer — Claude can guide you through the entire process.

### Skill Ideas We'd Love to See

- Industry-specific templates (CRM, inventory, project management)
- Data migration helpers
- Sync patterns with other tools
- Reporting and analytics scripts
- Custom interface components

If you build something cool, share it! Open a PR or create your own plugin.

## License

MIT — Use it, fork it, build on it.

---

**Built by [Rob Weidner](https://github.com/robweidner)** — This is a community project. Make it yours.
