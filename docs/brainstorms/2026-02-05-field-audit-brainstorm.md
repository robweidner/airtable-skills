# Airtable Field Audit Skill

**Date:** 2026-02-05
**Status:** Ready for planning

---

## What We're Building

A new skill `/airtable-field-audit` that uses AI to analyze Airtable field schemas and detect quality issues:

- **Duplicate or near-duplicate fields** (e.g., "Email" and "E-mail")
- **Whitespace issues** (leading/trailing spaces in field names)
- **Similar/confusing names** (e.g., "Status" vs "Status 2" vs "Project Status")
- **Inconsistent casing** (e.g., "first_name" vs "FirstName" vs "First Name")

The skill supports two modes:
1. **Report only** – Generate findings, user fixes manually
2. **Auto-fix with confirmation** – Propose fixes, user approves before applying

---

## Why This Approach

**Inline MCP execution** was chosen over script-based or report-only approaches because:

1. **Integrated experience** – User stays in Claude Code, no context switching
2. **AI-native analysis** – Claude reviews field names, types, and descriptions holistically to judge similarity (not just string matching)
3. **Leverages existing tools** – Uses `mcp__airtable__describe_table` already in the plugin
4. **Graceful degradation** – If field renaming isn't supported via MCP, skill can output manual instructions

---

## Key Decisions

| Decision | Choice | Rationale |
|----------|--------|-----------|
| Invocation | `/airtable-field-audit` | Explicit user action, not auto-triggered |
| Scope | Prompt user: table / base / all | Flexibility for small cleanups or full audits |
| Detection method | AI context analysis | Better than simple string matching; considers types + descriptions |
| Fix options | Report-only OR auto-fix | User chooses based on comfort level |
| Issue types | Basics only | Duplicates, whitespace, similar names, casing (no usage analysis) |

---

## User Flow

```
User: /airtable-field-audit

Claude: What would you like to audit?
  1. Single table
  2. Entire base
  3. All accessible bases

User: [selects base]

Claude: [fetches schema, analyzes fields]

Claude: Found 4 issues in "Projects" base:

| Table    | Field         | Issue                    | Suggested Fix       |
|----------|---------------|--------------------------|---------------------|
| Tasks    | "Status "     | Trailing space           | Rename to "Status"  |
| Tasks    | "email"       | Similar to "Email" in Contacts | Merge or clarify |
| Projects | "Start Date"  | Similar to "Start date"  | Standardize casing  |
| Contacts | "  Name"      | Leading space            | Rename to "Name"    |

Would you like to:
  1. Apply all fixes automatically
  2. Review and approve each fix
  3. Export report only
```

---

## Open Questions

1. **MCP field update support** – Can we rename fields via `mcp__airtable__update_field`? If not, we'll output manual instructions.
2. **Cross-table duplicate handling** – When duplicates span tables (e.g., "Email" in both Tasks and Contacts), should we flag these or assume intentional?
3. **Undo support** – Should we log original names for rollback? (Probably YAGNI for v1)

---

## Next Steps

Run `/workflows:plan` to create detailed implementation plan.
