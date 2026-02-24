# Airtable Skills for Claude Code

> Turn Claude into an Airtable expert — schema design, scripting, automations, interfaces, and API integrations.

[![Claude Code Plugin](https://img.shields.io/badge/Claude_Code-Plugin-6366f1)](https://claude.ai/code)
[![Version](https://img.shields.io/badge/version-1.5.0-blue)](https://github.com/robweidner/airtable-skills/releases)
[![License: MIT](https://img.shields.io/badge/License-MIT-green.svg)](LICENSE)

## What's Included

| Skill | How to Use | What It Does |
|-------|-----------|--------------|
| **airtable** | Auto-invokes when you mention Airtable | Full Airtable assistant — schema, scripts, automations, interfaces, API |
| **airtable-extensions** | `/airtable:airtable-extensions` | Custom React extensions — Blocks SDK + Interface Extensions SDK, dev server troubleshooting, auto-release hooks |
| **airtable-field-audit** | `/airtable:airtable-field-audit` | Find and fix field naming issues — duplicates, whitespace, similar names |
| **airtable-base-audit** | `/airtable:airtable-base-audit` | Comprehensive base health analysis — field usage, performance, schema quality |

> **How invocation works:** The main `airtable` skill activates automatically whenever you ask Claude about Airtable. The other three skills are slash commands you type manually. All slash commands are prefixed with `airtable:` because Claude Code namespaces plugin skills.

## Prerequisites

Before installing, make sure you have:

- [ ] **Claude Code** installed and authenticated ([setup guide](https://code.claude.com/docs/en/quickstart))
- [ ] **Claude Code v1.0.33+** — run `claude --version` to check. If below 1.0.33, update first.
- [ ] **Node.js 18+** — required for the Airtable MCP server (`npx airtable-mcp-server`)
- [ ] **An Airtable account** with a Team, Business, or Enterprise plan (for extension development; schema/scripting works on any plan)

## Quick Start

### Step 1: Install the Plugin

**From a marketplace (recommended):**

```bash
# In Claude Code, run these commands:
/plugin marketplace add robweidner/airtable-skills
/plugin install airtable@robweidner-airtable
```

The first command registers the marketplace. The second installs the plugin. You only need to do this once.

**For development/testing (load without installing):**

```bash
# Clone the repo anywhere
git clone https://github.com/robweidner/airtable-skills.git

# Start Claude Code with the plugin loaded directly
claude --plugin-dir ./airtable-skills/plugins/airtable
```

### Step 2: Create Your Airtable Token

The plugin uses the Airtable MCP server, which needs a Personal Access Token (PAT) to talk to your bases.

1. Go to [airtable.com/create/tokens](https://airtable.com/create/tokens)
2. Click **Create new token**
3. Name it something like "Claude Code"
4. **Scopes** — add these four:
   - `data.records:read`
   - `data.records:write`
   - `schema.bases:read`
   - `schema.bases:write`
5. **Access** — select only the base(s) you're working with (don't grant access to all bases)
6. Click **Create token** and copy it

### Step 3: Set the Token as an Environment Variable

Add this line to your shell profile (`~/.zshrc`, `~/.bashrc`, or `~/.bash_profile`):

```bash
export AIRTABLE_PAT="patXXXXXXXXXXXXXX"
```

Then reload your shell:

```bash
source ~/.zshrc  # or ~/.bashrc
```

> **Why an environment variable?** The plugin's `.mcp.json` auto-configures the Airtable MCP server and reads `AIRTABLE_PAT` from your environment. No manual MCP setup needed.

### Step 4: Verify It Works

Start (or restart) Claude Code and try:

```
I have an Airtable base for tracking customer orders. Help me add a Status field with options: Pending, In Progress, Shipped, Delivered.
```

Claude should activate the Airtable skill automatically and guide you through it. You'll know it's working when Claude references Airtable-specific patterns like field types, API operations, or the MCP tools.

To verify the slash commands work:

```
/airtable:airtable-field-audit
```

This should start the field audit workflow.

### Step 5: Save Your Preferences (Optional)

The skill adapts to your experience level. Add these to your `CLAUDE.md` (either `~/.claude/CLAUDE.md` for global, or `.claude/CLAUDE.md` in your project):

```markdown
## Airtable Preferences
airtable_experience: developer
airtable_emoji_mode: auto
airtable_script_style: minimal
airtable_use_field_ids: true
```

| Preference | Options | Description |
|------------|---------|-------------|
| Experience | `beginner` / `power-user` / `developer` | Controls explanation depth |
| Emoji Mode | `auto` / `new-only` / `ask` / `none` | Field name emoji prefixes |
| Script Style | `minimal` / `comprehensive` | Output data vs. all-in-one scripts |
| Field IDs | `true` / `false` | Store IDs in descriptions for robust scripts |

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
| **React extensions** | Blocks SDK + Interface Extensions SDK, data source config, `block run` troubleshooting, auto-release on git push |

### API Limitations (with Workarounds)

The Airtable API has known limitations. This skill handles them automatically:

| Can't Create via API | Workaround |
|---------------------|------------|
| **Bases** | User creates manually, provides base ID |
| **Formula fields** | Creates as text with 🔧 marker, formula in description |
| **Rollup fields** | Same as formula |
| **Lookup fields** | Same as formula |
| **Automations** | Generates ASCII flow diagrams + setup steps |

After API operations, you get a checklist of manual conversions needed.

### Emoji Naming System

Visual field identification at a glance:

```
🔗 Company
📧 Email
#️⃣ Quantity
💰 Total
📅 Due Date
1️⃣ Status
🤖 Full Name
```

Stack markers for context: `👷🤖 Commission Calc`

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

**Extensions** (slash command):
```
/airtable:airtable-extensions
Build a custom extension to visualize sales pipeline as a Kanban board
```

**Interface Extensions troubleshooting** (slash command):
```
/airtable:airtable-extensions
I'm getting "Field does not exist" errors in my interface extension — help me debug it
```

**Field Audit** (slash command):
```
/airtable:airtable-field-audit
```

**Base Health Check** (slash command):
```
/airtable:airtable-base-audit
```

## Documentation

### Reference Files

| File | When to Use |
|------|-------------|
| [field-types.md](plugins/airtable/skills/airtable/reference/field-types.md) | Creating/modifying fields |
| [emoji-conventions.md](plugins/airtable/skills/airtable/reference/emoji-conventions.md) | Naming conventions |
| [api-patterns.md](plugins/airtable/skills/airtable/reference/api-patterns.md) | REST API operations |
| [mcp-patterns.md](plugins/airtable/skills/airtable/reference/mcp-patterns.md) | Airtable MCP tools |
| [scripting-api.md](plugins/airtable/skills/airtable/reference/scripting-api.md) | Writing scripts |
| [automations.md](plugins/airtable/skills/airtable/reference/automations.md) | Automation setup |
| [interface-designer.md](plugins/airtable/skills/airtable/reference/interface-designer.md) | Building interfaces |
| [pat-security.md](plugins/airtable/skills/airtable/reference/pat-security.md) | Token security |
| [interface-extensions-workflow.md](plugins/airtable/skills/airtable-extensions/reference/interface-extensions-workflow.md) | Interface extension dev workflow, troubleshooting, checklists |

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

## Common Gotchas

| Issue | Solution |
|-------|----------|
| "Base ID required" | Create base in Airtable UI, provide `appXXXXXX` ID |
| Field names are case-sensitive | Use field IDs instead |
| Linked records fail | Create the linked table first |
| Formula field not created | Use the 🔧 workaround |
| Rate limited | Skill handles this — uses exponential backoff |
| Extension out of sync with repo | Set up the auto-release Claude Code hook (see `/airtable:airtable-extensions`) |
| `block run` won't connect in Chrome | Check CORS flags — see Interface Extensions troubleshooting |
| MCP tools not available | Verify `AIRTABLE_PAT` is set: `echo $AIRTABLE_PAT` |
| Slash commands not found | Use full namespaced name: `/airtable:airtable-extensions` |

## Troubleshooting Installation

**"Unknown command" when running `/plugin`:**
Update Claude Code to v1.0.33+. Run `claude --version` to check.

**MCP server not starting:**
- Verify Node.js is installed: `node --version` (need 18+)
- Verify token is set: `echo $AIRTABLE_PAT`
- Restart Claude Code after setting the environment variable

**Skills not appearing:**
- Run `/plugin` and check the **Installed** tab — the `airtable` plugin should be listed
- Check the **Errors** tab for any loading issues
- Try clearing the cache: `rm -rf ~/.claude/plugins/cache`, restart Claude Code, and reinstall

**Plugin validation** (for contributors):
```bash
cd /path/to/airtable-skills
claude plugin validate .
# Or from within Claude Code:
/plugin validate .
```

## Contributing

This plugin belongs to the community. Here's how you can get involved:

### Improve This Plugin

Fork it, make it better, and submit a PR:

- **Add scripting patterns** — `plugins/airtable/skills/airtable/examples/scripting-patterns.md`
- **Update API docs** — as Airtable changes
- **New automation templates** — `plugins/airtable/skills/airtable/reference/automations.md`
- **Bug fixes and improvements**

### Build Your Own Skills

The best part of Claude Code skills? **You can create your own.**

1. Fork this repo as a starting point
2. Modify the skills in `plugins/airtable/skills/` to match your workflow
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
