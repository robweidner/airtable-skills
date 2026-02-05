---
title: "feat: Airtable Field Audit Skill"
type: feat
date: 2026-02-05
---

# feat: Airtable Field Audit Skill

## Overview

Add a new `/airtable-field-audit` skill that analyzes Airtable field schemas to detect naming issues (duplicates, whitespace, similar names, inconsistent casing) and optionally auto-fixes them via the Airtable API.

## Problem Statement

Airtable bases accumulate field naming inconsistencies over time:
- Trailing spaces (`"Status "` vs `"Status"`)
- Near-duplicates (`"Email"` and `"E-mail"`)
- Confusing similar names (`"Status"`, `"Status 2"`, `"Task Status"`)
- Inconsistent casing (`"first_name"` vs `"FirstName"`)

These cause confusion, broken integrations, and data quality issues. Currently there's no easy way to audit and clean these up.

## Proposed Solution

A dedicated skill that:
1. Fetches table schemas via MCP tools (`describe_table`)
2. Uses AI to analyze field names holistically (names + types + descriptions)
3. Presents findings in a clean table format
4. Offers three remediation paths: auto-fix all, review each, or export report

---

## Technical Approach

### Architecture

```
/airtable-field-audit
         │
         ▼
┌─────────────────────┐
│ 1. Scope Selection  │  ← AskUserQuestion: table / base / all bases
└─────────────────────┘
         │
         ▼
┌─────────────────────┐
│ 2. Schema Fetch     │  ← MCP: list_bases → list_tables → describe_table
└─────────────────────┘
         │
         ▼
┌─────────────────────┐
│ 3. AI Analysis      │  ← Claude analyzes field names for issues
└─────────────────────┘
         │
         ▼
┌─────────────────────┐
│ 4. Present Findings │  ← Markdown table with issues + suggested fixes
└─────────────────────┘
         │
         ▼
┌─────────────────────┐
│ 5. Remediation      │  ← User chooses: auto-fix / review each / export
└─────────────────────┘
         │
         ▼
┌─────────────────────┐
│ 6. Apply Fixes      │  ← API: PATCH /fields/{fieldId} to rename
└─────────────────────┘
```

### Key Technical Details

**Field Rename API Endpoint:**
```bash
PATCH /v0/meta/bases/{baseId}/tables/{tableId}/fields/{fieldId}
{
  "name": "New Field Name",
  "description": "Optional updated description"
}
```

**MCP Tools Used:**
- `mcp__airtable__list_bases` - Let user select base(s)
- `mcp__airtable__list_tables` - Get tables in selected base
- `mcp__airtable__describe_table` - Get field details (name, type, ID, description)

**Rate Limiting:**
- 5 requests/second per base
- For large audits (50+ tables), implement 200ms delay between calls

---

## Issue Detection Rules

### 1. Whitespace Issues
**Pattern:** Leading or trailing spaces in field name
**Detection:** Regex `/^\s|\s$/`
**Fix:** Trim whitespace

### 2. Near-Duplicate Fields
**Pattern:** Fields with very similar names (typos, hyphens, etc.)
**Detection:** AI judgment considering:
- Levenshtein distance ≤ 2
- Same semantic meaning ("Email" vs "E-mail" vs "email")
- Same field type
**Fix:** Suggest merging or clarifying distinction

### 3. Similar/Confusing Names
**Pattern:** Same base name with numeric suffix or vague modifier
**Detection:** Pattern matching + AI judgment:
- `"Status"` vs `"Status 2"`
- `"Date"` vs `"Date Created"` vs `"Date Modified"`
**Fix:** Suggest clearer, distinct names

### 4. Inconsistent Casing
**Pattern:** Same logical field name, different casing conventions
**Detection:** Case-insensitive match + different actual casing:
- `"firstName"` vs `"FirstName"` vs `"first_name"`
**Fix:** Standardize to dominant convention in base (or Title Case default)

### Emoji Convention Awareness

Strip emoji prefixes (e.g., `[email]`, `[date]`) before comparison:
- `"[email] Email"` and `"Email"` are NOT duplicates
- `"[email] Primary Email"` and `"[email] Email"` ARE similar

---

## User Flow

### Step 1: Invoke and Select Scope

```
User: /airtable-field-audit

Claude: What would you like to audit?
  1. Single table
  2. Entire base
  3. All accessible bases
```

### Step 2: Select Target (if applicable)

For single table:
```
Claude: Which base?
  1. Projects (appXXX)
  2. CRM (appYYY)
  3. Inventory (appZZZ)

User: 1

Claude: Which table?
  1. Tasks
  2. Milestones
  3. Team Members

User: 1
```

### Step 3: Analysis Progress

```
Claude: Analyzing fields in Projects > Tasks...
        Found 24 fields. Checking for issues...
```

### Step 4: Present Findings

```
Claude: Found 4 issues in "Projects" base:

| # | Table | Field | Issue | Suggested Fix |
|---|-------|-------|-------|---------------|
| 1 | Tasks | "Status " | Trailing space | → "Status" |
| 2 | Tasks | "email" | Similar to "Email" in Contacts | Clarify or merge |
| 3 | Projects | "Start Date" | Inconsistent with "Start date" | → "Start Date" (Title Case) |
| 4 | Contacts | "  Name" | Leading space | → "Name" |

How would you like to proceed?
  1. Apply all fixes automatically
  2. Review and approve each fix
  3. Export report only (I'll fix manually)
```

### Step 5a: Auto-Fix All

```
User: 1

Claude: Applying fixes...
  ✓ Renamed "Status " → "Status" in Tasks
  ✓ Renamed "Start date" → "Start Date" in Projects
  ✓ Renamed "  Name" → "Name" in Contacts
  ⚠ Skipped "email" - requires manual decision (potential merge)

Done! 3 fixes applied, 1 skipped.
```

### Step 5b: Review Each

```
User: 2

Claude: Issue 1 of 4:
  Table: Tasks
  Field: "Status " (trailing space)
  Suggested: Rename to "Status"

  [1] Apply fix  [2] Skip  [3] Custom name  [4] Skip remaining

User: 1

Claude: ✓ Renamed "Status " → "Status"

Issue 2 of 4:
  Table: Tasks
  Field: "email"
  Similar to: "Email" in Contacts table
  Suggested: Clarify the distinction or merge data

  [1] Rename to "Task Email"  [2] Skip  [3] Custom name  [4] Skip remaining
```

### Step 5c: Export Report

```
User: 3

Claude: ## Field Audit Report - Projects Base

Generated: 2026-02-05

### Issues Found

| Table | Field | Issue Type | Suggested Fix |
|-------|-------|------------|---------------|
| Tasks | "Status " | Whitespace | Rename to "Status" |
| Tasks | "email" | Similar name | Clarify vs "Email" in Contacts |
| Projects | "Start Date" | Casing | Standardize with "Start date" |
| Contacts | "  Name" | Whitespace | Rename to "Name" |

### Manual Fix Steps

1. Open Tasks table → Click "Status " dropdown → Edit field → Change name to "Status"
2. Open Tasks table → Review "email" vs Contacts "Email" → Decide if distinct or merge
3. Open Projects table → Click "Start date" dropdown → Edit field → Change to "Start Date"
4. Open Contacts table → Click "  Name" dropdown → Edit field → Change name to "Name"
```

---

## Edge Cases

### Zero Issues Found
```
Claude: No field naming issues found in Projects base. All 47 fields look good!
```

### Naming Conflict
If renaming `"Status "` to `"Status"` but `"Status"` already exists:
```
Claude: ⚠ Cannot rename "Status " → "Status" (field already exists)
        Suggested alternatives:
        1. Rename to "Status (copy)"
        2. Delete the duplicate field first
        3. Skip this fix
```

### Cross-Table Duplicates
Present separately as informational:
```
Claude: ℹ️ Cross-table observations (may be intentional):
  - "Email" exists in both Tasks and Contacts tables
  - "Status" exists in Tasks, Projects, and Milestones tables

These are common patterns and likely intentional. No action needed unless you want to clarify naming.
```

### Permission Error
```
Claude: ⚠ Unable to rename "Status" - insufficient permissions.
        You may need Editor access to this base.
        Switching to report-only mode...
```

---

## Files to Create/Modify

### New Files

| File | Purpose |
|------|---------|
| `skills/airtable-field-audit/SKILL.md` | Main skill definition |

### Existing Files (No Changes Needed)

| File | Notes |
|------|-------|
| `skills/airtable/reference/api-patterns.md` | Already has field update endpoint |
| `skills/airtable/reference/emoji-conventions.md` | Reference for stripping prefixes |

---

## Implementation Phases

### Phase 1: Core Skill Structure

**Tasks:**
- [x] Create `skills/airtable-field-audit/SKILL.md` with frontmatter
- [x] Add role statement and overview
- [x] Define the scope selection flow (single table / base / all)

**Files:**
```
skills/airtable-field-audit/SKILL.md
```

### Phase 2: Schema Fetching Logic

**Tasks:**
- [x] Document MCP tool usage for list_bases, list_tables, describe_table
- [x] Add progress indication pattern for large audits
- [x] Handle rate limiting (200ms delays)

### Phase 3: Issue Detection Rules

**Tasks:**
- [x] Define whitespace detection (regex)
- [x] Define near-duplicate detection (AI-assisted)
- [x] Define similar name detection (pattern + AI)
- [x] Define casing inconsistency detection
- [x] Document emoji prefix stripping

### Phase 4: Findings Presentation

**Tasks:**
- [x] Define markdown table format for issues
- [x] Add cross-table observations section
- [x] Define the three remediation options

### Phase 5: Fix Application

**Tasks:**
- [x] Document API call pattern for field rename
- [x] Handle naming conflicts
- [x] Handle permission errors
- [x] Implement review-each flow
- [x] Implement export-report flow

### Phase 6: Testing & Polish

**Tasks:**
- [x] Test with sample base (various issue types)
- [x] Verify rate limiting doesn't cause issues
- [x] Confirm emoji prefix handling

---

## Acceptance Criteria

### Functional Requirements

- [x] Skill invokable via `/airtable-field-audit`
- [x] Supports three scope options: single table, base, all bases
- [x] Detects whitespace issues (leading/trailing spaces)
- [x] Detects near-duplicate field names
- [x] Detects similar/confusing names
- [x] Detects inconsistent casing
- [x] Presents findings in clean table format
- [x] Offers three remediation paths: auto-fix, review each, export
- [x] Successfully renames fields via API when auto-fix selected
- [x] Handles naming conflicts gracefully
- [x] Handles permission errors gracefully
- [x] Respects emoji prefix conventions (doesn't flag as duplicates)

### Non-Functional Requirements

- [x] Respects 5 req/sec rate limit
- [x] Shows progress for large audits (10+ tables)
- [x] Clear error messages for API failures

---

## Success Metrics

- User can audit a base and identify naming issues in < 2 minutes
- Auto-fix successfully renames fields without errors
- No false positives from emoji-prefixed fields

---

## References

### Internal References

- Brainstorm: `docs/brainstorms/2026-02-05-field-audit-brainstorm.md`
- Field update API: `skills/airtable/reference/api-patterns.md:264-274`
- Emoji conventions: `skills/airtable/reference/emoji-conventions.md`
- MCP patterns: `skills/airtable/reference/mcp-patterns.md`

### External References

- [Airtable Update Field API](https://airtable.com/developers/web/api/update-field)
