# Brainstorm: Airtable Claude Code Plugin

**Date:** 2026-02-05
**Status:** Complete
**Next:** `/workflows:plan`

---

## What We're Building

A comprehensive Claude Code plugin that teaches Claude how to work effectively with Airtable across the full workflow: schema design, API interactions, scripting, interfaces, and automations.

### Core Capabilities

1. **Schema Management via API/MCP**
   - Create and modify tables, fields, and views
   - Handle field types correctly (including limitations)
   - Naming conventions and best practices

2. **Airtable Scripting**
   - Vanilla JavaScript with Airtable SDK patterns
   - Ask user: script-heavy vs. minimal script + native automation steps
   - Support both experienced devs and power users

3. **Custom Interfaces**
   - Interface Designer layouts
   - Custom extensions development
   - Views vs. Interfaces guidance

4. **Automation Workflows**
   - ASCII diagrams for automation setup (can't create via API)
   - Embedded advanced actions (conditionals, single repeat step)
   - Script blocks within automations

5. **Best Practices Encoded**
   - Personal Access Token management (scoped per project)
   - Field marker convention: 🔄 emoji for formula fields needing manual conversion
   - Field descriptions containing the actual formula to copy/paste
   - **Auto-create standard fields:** Record ID, Created Time, Last Modified Time, Created By, Last Modified By
   - Formula and rollup field assistance (generate formulas, explain syntax)

6. **AI & Advanced Features**
   - Guidance on AI fields (when to use, setup patterns)
   - Verified data sets awareness
   - Power user features: linked record lookups, rollup aggregations, sync tables

7. **User Experience Adaptation**
   - Skill asks user type on first invocation: "Airtable beginner", "Power user", "Developer"
   - Adjusts explanation depth and assumed knowledge based on response
   - Friendly onboarding for Claude Code newcomers

8. **Emoji Naming Conventions**
   - Include comprehensive emoji prefix system as reference file
   - Emoji preference options: auto-add to all fields, new fields only, ask each time, none
   - Explain WHY emojis help: visual identification, signals to clients not to modify, easier maintenance

   **Complete Field Type Emojis:**
   | Field Type | Emoji | Notes |
   |------------|-------|-------|
   | Link to another record | 🔗 | |
   | Single line text | (none) | Basic field, emoji optional |
   | Long text | 📝 | |
   | Attachment | 📎 | |
   | Checkbox | ☑️ | |
   | Multiple select | 📋 | Changed from 🔢 |
   | Single select | 1️⃣ | |
   | User | 👤 | |
   | Date | 📅 | |
   | Phone number | 📱 | |
   | Email | 📧 | |
   | URL | 🌍 | |
   | Number | #️⃣ | |
   | Currency | 💰 | |
   | Percent | 📊 | |
   | Duration | ⏱️ | |
   | Rating | ⭐ | |
   | Formula | 🤖 | |
   | Rollup | 📈 | Aggregation |
   | Count | 🧮 | |
   | Lookup | 🔎 | |
   | Created time | 🕐 | |
   | Last modified time | 🕐 | Same as created |
   | Created by | 👤 | Same as user |
   | Last modified by | 👤 | Same as user |
   | Autonumber | 🔢 | Now available |
   | Barcode | 🔲 | |
   | Button | 👊 | Triggers action |

   **Purpose/Status Emojis (layer on top of type):**
   | Purpose | Emoji | When to use |
   |---------|-------|-------------|
   | Automation-involved | 🤖 | Fields used in automations |
   | Permissions | 🔒 | Access control fields |
   | Tested/Working | ✅ | Verified automation |
   | Work in progress | 👷 | Still being built |
   | Emergency/backup | 🆘 | Just-in-case workflows |
   | Archived | 💀 | Should be removed |
   | Needs review | 👀 | Attention required |
   | Helper tool | 🛟 | Utility field |
   | Trigger | 🚀 | Starts an action |
   | Synced | 🔄 | From sync source |
   | AI-powered | 🪄 | AI field |
   | External integration | ⚙️ | Depends on Make/Zapier/API |

   **Emoji stacking:** When multiple apply, stack important ones (status + type). Example: `👷🤖 Sales Commission` = WIP formula field

### Airtable Limitations Handled

| Limitation | Workaround |
|------------|------------|
| Can't create bases via API | Instruct user to create base first, provide base ID |
| Can't create automations via API | Generate ASCII diagrams with step-by-step setup |
| Formula fields need manual conversion | Create as single-line text with 🔄 marker, formula in description |
| Webhooks require setup | Document webhook configuration steps |

---

## Why This Approach

**Monolithic skill with reference files** chosen because:

1. **End-to-end workflows** - Users want to go from concept to working Airtable setup without switching contexts
2. **Adaptive explanations** - Single skill can adjust depth based on user expertise
3. **Progressive loading** - Reference files loaded only when needed (keeps main skill focused)
4. **Easier maintenance** - One place to update core patterns

### Plugin Structure

```
airtable-plugin/
├── skills/
│   ├── airtable/
│   │   ├── SKILL.md              # Main skill (comprehensive)
│   │   ├── reference/
│   │   │   ├── api-patterns.md   # REST API examples
│   │   │   ├── scripting-api.md  # Airtable scripting SDK
│   │   │   ├── field-types.md    # All field types + limitations
│   │   │   ├── interface-designer.md  # No-code Interface Designer basics
│   │   │   └── automations.md    # Automation patterns + ASCII templates
│   │   └── examples/
│   │       └── scripting-patterns.md
│   └── airtable-extensions/
│       ├── SKILL.md              # Custom interface extensions (React-based)
│       └── reference/
│           ├── extension-api.md
│           └── component-patterns.md
└── README.md                     # Installation instructions
```

**Two skills in plugin:**
- `airtable` - Main skill (auto-invocable) covering schema, API, scripting, Interface Designer basics, automations
- `airtable-extensions` - Custom interface extensions development (user-invocable only)

---

## Key Decisions

1. **MCP is optional enhancement** - Skill works without it, documents MCP usage when available
2. **Dynamic freshness** - Web fetch Airtable docs/changelog at runtime for latest patterns
3. **Plugin format** - Installable via Claude Code plugin system (not just copyable skill folder)
4. **Workaround-focused** - Provide concrete solutions for API limitations, not just "not possible"
5. **Both audiences** - Adapt explanation depth: terse for devs, detailed for power users
6. **Script preference via CLAUDE.md** - Users set prose guidance like "When writing Airtable scripts, prefer minimal scripts with native automation steps" in their CLAUDE.md
7. **User profiling** - Ask user type once, suggest adding to CLAUDE.md for persistence (e.g., `airtable_experience: power-user`)
8. **Standard fields** - Always suggest (not auto-add) standard fields (Record ID, timestamps, created/modified by) when creating tables; user confirms
9. **Emoji naming conventions** - Include Rob's emoji prefix system as a reference file; skill asks user preference for emoji handling

---

## Resolved Questions

1. **Webhook integration** - Setup documentation only (how to configure, available events)
2. **Interface extensions** - Separate skill (`airtable-extensions`) for custom React-based extensions
3. **Skill invocation** - Main skill auto-invocable; extensions skill user-only
4. **Example bases** - No templates; teach patterns, users build from scratch

---

## Target Deliverables

- [ ] Plugin structure with two skills: `airtable` and `airtable-extensions`
- [ ] Main skill reference files: API, scripting, fields, Interface Designer, automations
- [ ] Extensions skill reference files: extension API, component patterns
- [ ] Dynamic web fetch integration for Airtable docs freshness
- [ ] README with installation and PAT setup instructions
- [ ] Scripting patterns examples (not full base templates)

---

## Research Links

- https://airtable.com/developers
- https://airtable.com/developers/scripting
- https://airtable.com/developers/web
- https://airtable.com/developers/interface-extensions
- https://github.com/Airtable (official repos)
