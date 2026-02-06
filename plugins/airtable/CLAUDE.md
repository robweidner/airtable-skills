# Airtable Skills Plugin

This is the Airtable Skills plugin for Claude Code.

## Skills Provided

- **airtable** - Comprehensive Airtable assistant (auto-invokes on Airtable-related work)
- **airtable-extensions** - Custom React extensions (invoke with `/airtable-extensions`)

## User Preferences

Users can add these to their personal CLAUDE.md to save preferences:

```markdown
## Airtable Preferences

airtable_experience: power-user
airtable_emoji_mode: auto
airtable_script_style: minimal
airtable_use_field_ids: true

When writing Airtable scripts, prefer minimal scripts that output data
for native automation steps rather than doing everything in code.
```

### Preference Values

**airtable_experience:**
- `beginner` - Full explanations, walk through each step
- `power-user` - Key points and gotchas, assume Airtable knowledge
- `developer` - Terse, pattern-focused, skip basics

**airtable_emoji_mode:**
- `auto` - Add type emojis to all fields
- `new-only` - Only add emojis to new fields
- `ask` - Ask before adding emojis
- `none` - Never add emojis

**airtable_script_style:**
- `minimal` - Script outputs data, native steps do actions
- `comprehensive` - All logic in the script

**airtable_use_field_ids:**
- `true` - Store field IDs in field descriptions after creation (recommended)
- `false` - Don't store field IDs

### Why Field IDs?

Field names can change, but field IDs are permanent. When `airtable_use_field_ids: true`:
- After creating fields, skill adds `Field ID: fldXXXXXXXXXXXXXX` to descriptions
- Scripts and integrations should use these IDs instead of names
- Makes code more robust against field renaming

## Key Limitations

These cannot be done via API/MCP:
- Create bases (user must create manually, provide base ID)
- Create automations (skill generates ASCII diagrams instead)
- Create formula/rollup/lookup fields (workaround: create as text with marker)

## Formula Field Workaround

When creating formula fields, the skill:
1. Creates as Single Line Text with `[convert]` prefix
2. Puts formula in field description
3. Outputs checklist of fields needing manual conversion

## Contributing

This plugin is open source. Contributions welcome:
- Add more scripting patterns to `examples/`
- Update API documentation as Airtable changes
- Add new automation diagram templates
