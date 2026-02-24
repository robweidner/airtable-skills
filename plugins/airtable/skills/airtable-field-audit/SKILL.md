---
name: airtable-field-audit
description: Audit Airtable field names for issues like duplicates, whitespace, similar names, and inconsistent casing. Use when user wants to clean up field naming in their bases.
disable-model-invocation: true
---

# Airtable Field Audit Skill

You are an expert at analyzing Airtable schemas for field naming issues. Help users identify and fix problematic field names.

## When to Use This Skill

- User explicitly invokes `/airtable-field-audit`
- User asks to "clean up fields", "audit field names", or "find duplicate fields"

## Overview

This skill analyzes Airtable field schemas to detect:
- **Whitespace issues** - Leading or trailing spaces in field names
- **Near-duplicates** - Very similar names (typos, hyphens, etc.)
- **Similar/confusing names** - Same base name with numeric suffixes
- **Inconsistent casing** - Same logical name with different casing

After detection, offer three remediation paths:
1. **Auto-fix all** - Apply suggested fixes automatically
2. **Review each** - Approve fixes one-by-one
3. **Export report** - Generate report for manual fixes

---

## Step 1: Select Audit Scope

Ask the user what they want to audit:

```
What would you like to audit?
  1. Single table
  2. Entire base
  3. All accessible bases
```

**For single table:**
1. List available bases using `mcp__airtable__list_bases`
2. Present numbered list, let user select
3. List tables in selected base using `mcp__airtable__list_tables`
4. Present numbered list, let user select

**For entire base:**
1. List available bases
2. Present numbered list, let user select

**For all bases:**
- Will audit everything accessible to the token

---

## Step 2: Fetch Schemas

Use MCP tools to retrieve field information.

### MCP Tool Usage

```
# List all bases
mcp__airtable__list_bases

# List tables in a base (use identifiersOnly for speed)
mcp__airtable__list_tables
  baseId: "appXXX"
  detailLevel: "identifiersOnly"

# Get full schema for a table
mcp__airtable__describe_table
  baseId: "appXXX"
  tableId: "tblXXX"
```

### Progress Indication

For large audits (10+ tables), show progress:

```
Analyzing fields...
  Base: Projects (1 of 3)
    Table: Tasks (1 of 5)... found 24 fields
    Table: Milestones (2 of 5)... found 12 fields
    ...
```

### Rate Limiting

Airtable API has a 5 requests/second limit. For bases with many tables:
- Add 200ms delay between `describe_table` calls
- This prevents rate limit errors during large audits

---

## Step 3: Analyze Fields for Issues

For each field, check for these issue types:

### Issue Type 1: Whitespace

**Detection:** Field name has leading or trailing whitespace

```javascript
// Regex pattern
/^\s|\s$/
```

**Examples:**
- `"Status "` (trailing space)
- `"  Name"` (leading spaces)

**Fix:** Trim the whitespace

### Issue Type 2: Near-Duplicates

**Detection:** Two fields with very similar names that likely represent the same thing

Consider:
- Levenshtein distance <= 2 (typos)
- Semantic similarity: "Email" vs "E-mail" vs "email"
- Same field type increases likelihood of duplicate

**Examples:**
- `"Email"` and `"E-mail"` (hyphen variation)
- `"Phone"` and `"Pbone"` (typo)
- `"Status"` and `"Staus"` (typo)

**Fix:** Suggest keeping one, clarifying, or merging

### Issue Type 3: Similar/Confusing Names

**Detection:** Fields with same base name plus numeric suffix or vague modifier

Patterns:
- `"Field"` and `"Field 2"` and `"Field 3"`
- `"Date"` and `"Date (old)"` and `"Date - new"`

**Examples:**
- `"Status"` and `"Status 2"` (likely accidental duplicate)
- `"Notes"` and `"Notes (backup)"`

**Fix:** Suggest renaming with clearer distinction or merging

### Issue Type 4: Inconsistent Casing

**Detection:** Same field name with different casing conventions

Compare case-insensitively, flag if actual casing differs:
- `"firstName"` vs `"FirstName"` vs `"first_name"` vs `"First Name"`

**Fix:** Standardize to the dominant casing convention in the base, or Title Case if no clear convention

### Emoji Prefix Awareness

The main Airtable skill uses emoji prefixes like `📧`, `📅`, `🔗`.

**IMPORTANT:** Strip emoji prefixes before comparison to avoid false positives.

```javascript
// Strip prefix pattern: emoji characters at start of name
const cleanName = fieldName.replace(/^[\p{Emoji_Presentation}\p{Extended_Pictographic}\uFE0F]+\s*/gu, '');
```

**Examples:**
- `"📧 Email"` and `"Email"` are NOT duplicates (one has emoji prefix)
- `"📧 Primary Email"` and `"📧 Contact Email"` - compare "Primary Email" vs "Contact Email"
- `"👷🤖 Sales"` - strip multiple prefixes: "Sales"

---

## Step 4: Present Findings

### Issues Found

Present in a clean table:

```markdown
Found 4 issues in "Projects" base:

| # | Table | Field | Issue | Suggested Fix |
|---|-------|-------|-------|---------------|
| 1 | Tasks | "Status " | Trailing space | Rename to "Status" |
| 2 | Tasks | "email" | Near-duplicate of "Email" in Contacts | Clarify or merge |
| 3 | Projects | "Start Date" | Inconsistent casing with "Start date" | Standardize to "Start Date" |
| 4 | Contacts | "  Name" | Leading spaces | Rename to "Name" |
```

### Cross-Table Observations

If same-named fields appear across tables, present as informational (not actionable):

```markdown
Cross-table observations (may be intentional):
  - "Email" exists in Tasks, Contacts, and Companies
  - "Status" exists in Tasks, Projects, and Milestones

These are common patterns and often intentional. No action needed unless you want to clarify naming.
```

### Zero Issues Found

```markdown
No field naming issues found in "Projects" base. All 47 fields look good!
```

---

## Step 5: Remediation Options

Ask the user how to proceed:

```
How would you like to proceed?
  1. Apply all fixes automatically
  2. Review and approve each fix
  3. Export report only (I'll fix manually)
```

---

## Step 6: Apply Fixes

### Option 1: Auto-Fix All

For each fixable issue (whitespace, casing), apply the fix using the Airtable API.

**API Endpoint:**
```bash
PATCH /v0/meta/bases/{baseId}/tables/{tableId}/fields/{fieldId}
Content-Type: application/json
Authorization: Bearer {PAT}

{
  "name": "New Field Name"
}
```

**Progress output:**
```
Applying fixes...
  ✓ Renamed "Status " → "Status" in Tasks
  ✓ Renamed "Start date" → "Start Date" in Projects
  ✓ Renamed "  Name" → "Name" in Contacts
  ⚠ Skipped "email" - requires manual decision (cross-table duplicate)

Done! 3 fixes applied, 1 skipped.
```

### Option 2: Review Each Fix

Present each issue one at a time:

```
Issue 1 of 4:
  Table: Tasks
  Field: "Status " (trailing space)
  Type: Single Select
  Suggested: Rename to "Status"

  [1] Apply fix  [2] Skip  [3] Custom name  [4] Skip remaining
```

If user chooses `[3] Custom name`:
```
Enter new name for field: _
```

After each decision, apply fix (if approved) and move to next.

### Option 3: Export Report

Generate a markdown report:

```markdown
## Field Audit Report

**Base:** Projects
**Generated:** 2026-02-05
**Total fields analyzed:** 47
**Issues found:** 4

### Issues

| Table | Field | Issue Type | Suggested Fix |
|-------|-------|------------|---------------|
| Tasks | "Status " | Whitespace | Rename to "Status" |
| Tasks | "email" | Near-duplicate | Clarify vs "Email" in Contacts |
| Projects | "Start date" | Casing | Rename to "Start Date" |
| Contacts | "  Name" | Whitespace | Rename to "Name" |

### Manual Fix Instructions

1. **Tasks > "Status "**
   - Click field dropdown → Edit field
   - Change name to: `Status`
   - Save

2. **Tasks > "email"**
   - Review relationship with Contacts > "Email"
   - If same purpose: delete one and update records
   - If different purpose: rename to clarify (e.g., "Task Email")

3. **Projects > "Start date"**
   - Click field dropdown → Edit field
   - Change name to: `Start Date`
   - Save

4. **Contacts > "  Name"**
   - Click field dropdown → Edit field
   - Change name to: `Name`
   - Save
```

---

## Edge Cases

### Naming Conflict

If the target name already exists:

```
⚠ Cannot rename "Status " → "Status" (field already exists)

Options:
  1. Rename to "Status (copy)"
  2. Skip this fix (delete duplicate manually first)
  3. Enter custom name
```

### Permission Error

If rename fails due to permissions:

```
⚠ Unable to rename "Status" - insufficient permissions.
   You may need Editor access to this base.

Switching to report-only mode for remaining fixes...
```

### MCP Not Available

If MCP tools aren't configured:

```
Airtable MCP tools not available. To use this skill, configure the Airtable MCP.

Alternatively, you can provide your schema manually by:
1. Opening your base in Airtable
2. Going to Help → API documentation
3. Copying the field list for each table
```

---

## Best Practices

1. **Start with report mode** for large bases to review changes before applying
2. **Back up** critical bases before auto-fixing (though renames are reversible)
3. **Communicate with team** before bulk renaming fields in shared bases
4. **Consider formulas** - Airtable updates formulas automatically when fields are renamed, but external integrations may break
5. **Run periodically** - audit bases every few months to prevent naming debt
