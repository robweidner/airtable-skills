---
title: "feat: Airtable Claude Code Plugin"
type: feat
date: 2026-02-05
---

# Airtable Claude Code Plugin

## Overview

A comprehensive Claude Code plugin that teaches Claude how to work effectively with Airtable across the full workflow: schema design, API interactions, scripting, interfaces, and automations.

**Two skills in one plugin:**
- `airtable` - Main skill (auto-invocable) covering schema, API, scripting, Interface Designer, automations
- `airtable-extensions` - Custom interface extensions development (user-invocable only)

## Problem Statement / Motivation

Airtable users ranging from beginners to developers need Claude to understand:
1. Airtable's unique field types and their API limitations
2. Best practices for schema design, naming conventions, and security
3. How to write Airtable scripts (vanilla JS + Airtable SDK)
4. How to set up automations (which can't be created via API)
5. When to use Views vs Interfaces vs Custom Extensions

Currently, Claude has general knowledge but lacks Airtable-specific patterns, workarounds for API limitations, and best practices that power users have developed.

## Proposed Solution

A Claude Code plugin with:
1. **Main SKILL.md** - Core instructions, user profiling, workflow routing
2. **Reference files** - Detailed documentation loaded on demand
3. **Emoji naming system** - Visual field identification conventions
4. **Workaround patterns** - ASCII diagrams for automations, formula field markers

---

## Technical Approach

### Plugin Architecture

```
airtable-plugin/
├── skills/
│   ├── airtable/
│   │   ├── SKILL.md                      # Main skill (~400 lines)
│   │   ├── reference/
│   │   │   ├── api-patterns.md           # REST API examples
│   │   │   ├── mcp-patterns.md           # MCP tool usage
│   │   │   ├── scripting-api.md          # Airtable scripting SDK
│   │   │   ├── field-types.md            # All field types + limitations
│   │   │   ├── emoji-conventions.md      # Naming system
│   │   │   ├── interface-designer.md     # No-code interfaces
│   │   │   ├── automations.md            # Automation patterns + ASCII templates
│   │   │   └── pat-security.md           # Personal Access Token guide
│   │   └── examples/
│   │       └── scripting-patterns.md     # Real-world script examples
│   └── airtable-extensions/
│       ├── SKILL.md                      # Extensions skill (~200 lines)
│       └── reference/
│           ├── extension-api.md          # Airtable Blocks SDK
│           └── component-patterns.md     # React component patterns
├── README.md                             # Installation + PAT setup
└── CLAUDE.md                             # Plugin-specific guidance
```

### User Experience Flow

```
User invokes skill (auto or manual)
         │
         ▼
┌─────────────────────────────────┐
│ Check CLAUDE.md for preferences │
│ - airtable_experience: ?        │
│ - airtable_emoji_mode: ?        │
│ - airtable_script_style: ?      │
└─────────────────────────────────┘
         │
    Found? ─────No─────►  Ask user, suggest saving
         │
        Yes
         │
         ▼
┌─────────────────────────────────┐
│ Route to appropriate workflow   │
│ - Schema creation               │
│ - Script writing                │
│ - Automation setup (ASCII)      │
│ - Interface design              │
└─────────────────────────────────┘
         │
         ▼
┌─────────────────────────────────┐
│ Load reference files as needed  │
│ Adapt explanation depth         │
│ Apply emoji preferences         │
└─────────────────────────────────┘
```

---

## Implementation Phases

### Phase 1: Foundation (Core Structure)

**Deliverables:**
- [ ] Initialize git repo: `git init`
- [ ] Create plugin directory structure
- [ ] Write `README.md` with installation instructions
- [ ] Write `CLAUDE.md` with plugin-specific guidance
- [ ] Create main `skills/airtable/SKILL.md` skeleton

**SKILL.md skeleton:**
```yaml
---
name: airtable
description: Comprehensive Airtable assistant for schema design, API interactions, scripting, interfaces, and automations. Use when working with Airtable bases, tables, fields, or automations.
---

# Airtable Skill

## First-Time Setup

[Check for preferences in user's CLAUDE.md]
[If missing, ask experience level and emoji preference]
[Suggest adding to CLAUDE.md for persistence]

## Core Workflows

### Creating Tables & Fields
[Load reference/field-types.md]
[Apply emoji conventions if enabled]
[Handle formula field workaround with 🔄 marker]

### Writing Scripts
[Load reference/scripting-api.md]
[Ask script style preference if not set]
[Adapt explanation depth to user level]

### Setting Up Automations
[Load reference/automations.md]
[Generate ASCII diagram]
[Provide step-by-step UI instructions]

### Building Interfaces
[Load reference/interface-designer.md]
[Explain Views vs Interfaces decision]

## Reference Files
- For API patterns, see [api-patterns.md](reference/api-patterns.md)
- For MCP tools, see [mcp-patterns.md](reference/mcp-patterns.md)
- For scripting, see [scripting-api.md](reference/scripting-api.md)
- For field types, see [field-types.md](reference/field-types.md)
- For emoji conventions, see [emoji-conventions.md](reference/emoji-conventions.md)
- For automations, see [automations.md](reference/automations.md)
```

**Success criteria:**
- [ ] Plugin can be cloned and installed
- [ ] Main skill loads without errors
- [ ] Skill appears in `/` menu

---

### Phase 2: Reference Documentation

**Deliverables:**

#### reference/field-types.md
- [ ] All 28 Airtable field types
- [ ] API support matrix (can create, can modify, limitations)
- [ ] Field type decision tree
- [ ] Formula field workaround with 🔄 marker pattern

#### reference/emoji-conventions.md
- [ ] Complete emoji mapping (28 field types + 12 status/purpose)
- [ ] Emoji stacking rules (status first, then type)
- [ ] WHY emojis help explanation
- [ ] Examples of named fields

#### reference/api-patterns.md
- [ ] REST API authentication
- [ ] CRUD operations (create/read/update/delete records)
- [ ] Schema operations (create/modify tables/fields)
- [ ] Rate limiting (5 req/sec) and error handling
- [ ] Batch operations (max 10 records)

#### reference/mcp-patterns.md
- [ ] MCP tool reference (list_bases, list_tables, create_record, etc.)
- [ ] When MCP is available vs not
- [ ] Differences from REST API
- [ ] Configuration requirements

#### reference/pat-security.md
- [ ] Why scoped PATs matter
- [ ] Step-by-step token creation
- [ ] Scope matrix by operation type
- [ ] Storing tokens safely (.secrets.env)

**Success criteria:**
- [ ] Each reference file under 300 lines
- [ ] All field types documented with examples
- [ ] API patterns include error handling

---

### Phase 3: Scripting & Automations

**Deliverables:**

#### reference/scripting-api.md
- [ ] Airtable scripting SDK methods
- [ ] Input/output patterns
- [ ] Common patterns (query, update, create)
- [ ] Error handling in scripts
- [ ] Script vs automation decision guide

#### reference/automations.md
- [ ] Trigger types
- [ ] Action types
- [ ] Conditional patterns (embedded advanced actions)
- [ ] Single repeat step limitation
- [ ] Script block usage
- [ ] 5+ ASCII diagram templates

**ASCII Automation Template Example:**
```
┌─────────────────────────────────────────────┐
│  AUTOMATION: [Name]                         │
├─────────────────────────────────────────────┤
│                                             │
│  ┌────────────────────┐                     │
│  │ TRIGGER            │                     │
│  │ [Type]: [Details]  │                     │
│  └─────────┬──────────┘                     │
│            ▼                                │
│  ┌────────────────────┐                     │
│  │ CONDITION (if any) │                     │
│  │ [Logic]            │                     │
│  └─────────┬──────────┘                     │
│            ▼                                │
│  ┌────────────────────┐                     │
│  │ ACTION 1           │                     │
│  │ [Type]: [Details]  │                     │
│  └─────────┬──────────┘                     │
│            ▼                                │
│  ┌────────────────────┐                     │
│  │ ACTION 2 (if any)  │                     │
│  └────────────────────┘                     │
└─────────────────────────────────────────────┘
```

#### examples/scripting-patterns.md
- [ ] Query and filter records
- [ ] Batch update pattern
- [ ] Create linked records
- [ ] Input dialog pattern
- [ ] Output formatting

**Success criteria:**
- [ ] Scripts run correctly when copied to Airtable
- [ ] ASCII diagrams render properly in terminal
- [ ] Both "script-heavy" and "minimal script" approaches documented

---

### Phase 4: Interface Designer

**Deliverables:**

#### reference/interface-designer.md
- [ ] Views vs Interfaces decision tree
- [ ] Interface components overview
- [ ] Layout patterns (dashboard, form, list)
- [ ] Filtering and permissions
- [ ] Embedding interfaces

**Success criteria:**
- [ ] Clear guidance on when to use Views vs Interfaces
- [ ] Layout wireframes as ASCII art
- [ ] Step-by-step Interface Designer instructions

---

### Phase 5: Custom Extensions Skill

**Deliverables:**

#### skills/airtable-extensions/SKILL.md
```yaml
---
name: airtable-extensions
description: Build custom React-based interface extensions for Airtable. Use when creating custom apps, visualizations, or tools that go beyond Interface Designer.
disable-model-invocation: true
---
```
- [ ] Prerequisites check (Node.js, blocks-cli)
- [ ] Project setup instructions
- [ ] Development workflow
- [ ] Deployment guide

#### reference/extension-api.md
- [ ] Airtable Blocks SDK overview
- [ ] Hooks (useBase, useRecords, useField, etc.)
- [ ] Permissions model
- [ ] Global config vs per-base storage

#### reference/component-patterns.md
- [ ] Common UI patterns
- [ ] Form components
- [ ] Table components
- [ ] Chart patterns
- [ ] TypeScript support

**Success criteria:**
- [ ] Extension skill loads without auto-invoking
- [ ] Clear path from zero to deployed extension
- [ ] TypeScript examples included

---

### Phase 6: Integration & Polish

**Deliverables:**
- [ ] Dynamic web fetch for Airtable docs freshness (optional)
- [ ] Webhook setup documentation (setup only, not listeners)
- [ ] Preference template for CLAUDE.md
- [ ] Final README polish
- [ ] Test all workflows end-to-end

**CLAUDE.md Preference Template:**
```markdown
## Airtable Preferences

<!-- Add these lines to your CLAUDE.md to save preferences -->

airtable_experience: power-user   <!-- beginner | power-user | developer -->
airtable_emoji_mode: auto         <!-- auto | new-only | ask | none -->
airtable_script_style: minimal    <!-- minimal | comprehensive -->

When writing Airtable scripts, prefer minimal scripts that output data
for native automation steps rather than doing everything in code.
```

**Success criteria:**
- [ ] All 8 core user flows work correctly
- [ ] Preferences persist across sessions
- [ ] Plugin ready for GitHub release

---

## Acceptance Criteria

### Functional Requirements

- [ ] Skill auto-invokes when Claude detects Airtable-related work
- [ ] User experience level asked once, saved to CLAUDE.md
- [ ] Emoji preference asked once, applied consistently
- [ ] Formula fields created with 🔄 marker and formula in description
- [ ] Automations output as ASCII diagrams with step-by-step instructions
- [ ] Scripts adapt explanation depth to user level
- [ ] Extensions skill only invokes via explicit `/airtable-extensions`

### Non-Functional Requirements

- [ ] Main SKILL.md under 500 lines
- [ ] Each reference file under 300 lines
- [ ] Plugin installs in under 2 minutes
- [ ] All documentation renders correctly in terminal

### Quality Gates

- [ ] All file paths in SKILL.md are valid
- [ ] All API examples tested against Airtable
- [ ] All ASCII diagrams render in monospace font
- [ ] README covers installation + PAT setup + first use

---

## Success Metrics

1. **Adoption:** Plugin cloned/installed by users
2. **Utility:** Users report successful Airtable workflows
3. **Completeness:** All 28 field types documented
4. **Workarounds effective:** Users successfully convert formula fields

---

## Dependencies & Prerequisites

| Dependency | Type | Notes |
|------------|------|-------|
| Claude Code | Required | Plugin system |
| Airtable account | Required | For testing |
| Personal Access Token | Required | API access |
| Airtable MCP | Optional | Enhanced tool access |
| Node.js | Optional | For extensions skill |

---

## Risk Analysis & Mitigation

| Risk | Impact | Mitigation |
|------|--------|------------|
| Airtable API changes | High | Include docs fetch, version note |
| Reference files too large | Medium | Keep under 300 lines each |
| Emoji rendering varies | Low | Test on common terminals |
| MCP not available | Medium | REST API fallback documented |

---

## File Checklist

### Core Files
- [ ] `README.md`
- [ ] `CLAUDE.md`
- [ ] `skills/airtable/SKILL.md`
- [ ] `skills/airtable-extensions/SKILL.md`

### Main Skill Reference Files
- [ ] `skills/airtable/reference/api-patterns.md`
- [ ] `skills/airtable/reference/mcp-patterns.md`
- [ ] `skills/airtable/reference/scripting-api.md`
- [ ] `skills/airtable/reference/field-types.md`
- [ ] `skills/airtable/reference/emoji-conventions.md`
- [ ] `skills/airtable/reference/interface-designer.md`
- [ ] `skills/airtable/reference/automations.md`
- [ ] `skills/airtable/reference/pat-security.md`
- [ ] `skills/airtable/examples/scripting-patterns.md`

### Extensions Skill Reference Files
- [ ] `skills/airtable-extensions/reference/extension-api.md`
- [ ] `skills/airtable-extensions/reference/component-patterns.md`

**Total: 16 files**

---

## References & Research

### Internal References
- Brainstorm: `docs/brainstorms/2026-02-05-airtable-skill-plugin-brainstorm.md`
- Emoji conventions: `Rob Weidner Emoji & Naming Conventions.md`

### External References
- Airtable Web API: https://airtable.com/developers/web
- Airtable Scripting: https://airtable.com/developers/scripting
- Airtable Interface Extensions: https://airtable.com/developers/interface-extensions
- Claude Code Skills: https://code.claude.com/docs/en/skills
