# Airtable Skills Plugin for Claude Code

A comprehensive Claude Code plugin that teaches Claude how to work effectively with Airtable across the full workflow: schema design, API interactions, scripting, interfaces, and automations.

## Skills Included

| Skill | Invocation | Description |
|-------|------------|-------------|
| `airtable` | Auto-invokes | Schema, API, scripting, Interface Designer, automations |
| `airtable-extensions` | `/airtable-extensions` | Custom React-based interface extensions |

## Installation

Clone this repository into your Claude Code plugins directory:

```bash
# Navigate to your Claude Code plugins directory
cd ~/.claude/plugins

# Clone the plugin
git clone https://github.com/yourusername/airtable-skills.git airtable
```

Or copy the `skills/` directory to your project's `.claude/skills/` folder for project-specific use.

## Setup

### Create a Personal Access Token (PAT)

**Important:** Create a scoped PAT for each project. This ensures Claude can only access bases for that specific project, protecting your other Airtable data.

1. Go to [airtable.com/create/tokens](https://airtable.com/create/tokens)
2. Click "Create new token"
3. Name it descriptively (e.g., "CRM Project - Claude")
4. **Select ONLY the base(s)** you want Claude to access
5. Add required scopes:
   - `data.records:read` - Read records
   - `data.records:write` - Create/update records
   - `schema.bases:read` - Read table schemas
   - `schema.bases:write` - Create/modify tables and fields
6. Click "Create token" and copy it

### Store Your Token

Add to your project's `.secrets.env` (never commit this file):

```bash
AIRTABLE_PAT=patXXXXXXXXXXXXXX
```

Or configure in your environment for the Airtable MCP if using that.

## Quick Start

Once installed, Claude will automatically invoke the Airtable skill when it detects Airtable-related work.

### First Use

On first invocation, the skill will ask:

1. **Your experience level** - Beginner, Power User, or Developer
2. **Emoji preference** - How to handle field name prefixes

Add these to your CLAUDE.md to save preferences:

```markdown
## Airtable Preferences

airtable_experience: power-user
airtable_emoji_mode: auto
airtable_script_style: minimal
```

### Example Workflows

**Create a table:**
```
Create a Contacts table with name, email, phone, and company fields
```

**Write a script:**
```
Write an Airtable script to find all records where Status is "Pending" and update them to "In Progress"
```

**Set up an automation:**
```
Help me create an automation that sends a Slack message when a new record is added to the Orders table
```

**Build an interface:**
```
Design an interface for my sales team to view and update their assigned leads
```

## Features

### Schema Management
- Create and modify tables, fields, and views
- Suggest standard fields (Record ID, timestamps, created/modified by)
- Handle all 28 Airtable field types

### Formula Field Workaround
Airtable's API cannot create formula fields directly. The skill uses a workaround:
- Creates field as Single Line Text with marker
- Adds the formula to the field description
- Outputs a checklist of fields needing manual conversion

### Airtable Scripting
- Vanilla JavaScript with Airtable SDK patterns
- Adapts explanation depth to your experience level
- Offers "comprehensive" or "minimal" script styles

### Automation Diagrams
Since automations can't be created via API, the skill generates:
- ASCII diagrams showing the automation flow
- Step-by-step click-by-click setup instructions

### Emoji Naming Conventions
Optional emoji prefixes for visual field identification:
- Field types: `[link]`, `[date]`, `[formula]`, `[currency]`, etc.
- Status: `[wip]`, `[tested]`, `[archived]`, `[external-integration]`

## Airtable MCP (Optional)

If you have the Airtable MCP configured, the skill can use it for enhanced capabilities. The skill works without MCP using the REST API.

## Documentation

- [Field Types Reference](skills/airtable/reference/field-types.md)
- [Emoji Conventions](skills/airtable/reference/emoji-conventions.md)
- [API Patterns](skills/airtable/reference/api-patterns.md)
- [Scripting API](skills/airtable/reference/scripting-api.md)
- [Automations](skills/airtable/reference/automations.md)
- [Interface Designer](skills/airtable/reference/interface-designer.md)
- [PAT Security](skills/airtable/reference/pat-security.md)

## External Resources

- [Airtable Web API](https://airtable.com/developers/web)
- [Airtable Scripting](https://airtable.com/developers/scripting)
- [Airtable Interface Extensions](https://airtable.com/developers/interface-extensions)

## License

MIT
