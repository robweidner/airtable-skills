---
name: airtable-base-audit
description: Comprehensive health analysis of Airtable bases with scored reports covering field health, performance, and schema quality. Use when user wants to audit a base's overall health.
disable-model-invocation: true
---

# Airtable Base Audit Skill

You are an expert at analyzing Airtable bases for health issues. Help users identify problems across field usage, performance, and schema design.

## When to Use This Skill

- User explicitly invokes `/airtable-base-audit`
- User asks to "audit my base", "check base health", or "analyze my Airtable"

## Overview

This skill performs comprehensive base health analysis:

- **Field Health** (~35% of score) - Unused fields, low fill rates, duplicates, "(copy)" fields
- **Performance** (~35% of score) - Record counts, attachment storage, TODAY() formulas, complex formulas
- **Schema Quality** (~30% of score) - Repeated data, relationship patterns, normalization

Produces a letter grade (A-F) with percentage breakdown and actionable recommendations.

---

## Step 1: Determine Airtable Plan

Ask the user's plan to set correct limits:

```
What Airtable plan are you on?
  1. Teams (50K records/base, 20GB attachments)
  2. Business (125K records/base, 100GB attachments)
  3. Enterprise (500K records/base, 1TB attachments)
  4. I don't know (will use Business defaults)
```

### Plan Limits Reference

| Plan | Records/Base | Attachment Storage |
|------|--------------|-------------------|
| Teams | 50,000 | 20GB |
| Business | 125,000 | 100GB |
| Enterprise | 500,000 | 1TB |

---

## Step 2: Select Base to Audit

List available bases and let user choose:

```
# Get bases using MCP
mcp__airtable__list_bases
```

Present numbered list:

```
Which base would you like to audit?
  1. Projects (appXXX)
  2. CRM (appYYY)
  3. Inventory (appZZZ)
```

---

## Step 3: Gather Data

### 3.1 Fetch Tables and Fields

```
# Get all tables
mcp__airtable__list_tables
  baseId: "appXXX"
  detailLevel: "fullSchema"

# For each table, get detailed field info
mcp__airtable__describe_table
  baseId: "appXXX"
  tableId: "tblXXX"
```

### 3.2 Sample Records for Analysis

To calculate fill rates and uniqueness, sample records from each table:

```
# Sample records (first 100-1000 depending on table size)
mcp__airtable__list_records
  baseId: "appXXX"
  tableIdOrName: "tblXXX"
  maxRecords: 100
```

For large tables (>1000 records), sample in chunks to get representative data.

### Rate Limiting

Airtable API has a 5 requests/second limit. For bases with many tables:
- Add 200ms delay between API calls
- This prevents rate limit errors during large audits

### Progress Indication

```
Analyzing "Projects" base...
  - Fetching tables (5 found)
  - Analyzing fields (47 total)
    - Tasks: 24 fields, sampling records...
    - Milestones: 12 fields, sampling records...
    - ...
  - Checking performance metrics
  - Evaluating schema patterns
```

---

## Step 4: Analyze Field Health

### Check 1: Unused Fields (0% fill rate)

**Detection:** Field has no data in any sampled records.

```javascript
fill_rate = (records_with_value / total_records) * 100
unused = fill_rate === 0
```

**Severity:** Warning (may be new field or intentionally empty)

**Auto-Fix:** Can hide field (reversible)

### Check 2: Low Fill Rate (<10%)

**Detection:** Field has data in less than 10% of records.

**Severity:** Info (flag for review)

**Auto-Fix:** None (requires human judgment)

### Check 3: Uniqueness Analysis

**Detection:** Fields with names containing "email", "id", "key", "code" should have high uniqueness.

```javascript
uniqueness = (unique_values / total_non_empty_values) * 100
```

**Expected uniqueness:**
- Email fields: >95%
- ID/Key/Code fields: >99%

**Severity:** Warning if below threshold

**Auto-Fix:** None (data quality issue)

### Check 4: "(copy)" Fields

**Detection:** Field name contains "(copy)" - indicates UI duplication never renamed.

```javascript
/\(copy\)/i.test(fieldName)
```

**Severity:** Warning

**Auto-Fix:** Suggest rename (removing "(copy)")

### Check 5: Duplicate Data Patterns

**Detection:** Same non-trivial values appearing across multiple fields or tables.

Look for:
- Identical text appearing in 2+ fields
- Same select options duplicated across tables
- Repeated company names, emails, etc. (suggests missing link)

**Severity:** Info (may be intentional)

**Auto-Fix:** None (requires schema redesign)

---

## Step 5: Analyze Performance

### Check 1: Record Count vs Plan Limit

```javascript
usage_percent = (record_count / plan_limit) * 100
```

**Thresholds:**
- <50%: Good
- 50-75%: Info (plan ahead)
- 75-90%: Warning
- >90%: Critical

### Check 2: Attachment Storage

**Note:** Attachment size cannot be directly queried via API. Estimate by:
- Counting attachment fields
- Sampling attachment records
- Noting if storage warnings are mentioned

**Thresholds:** Same as record count

### Check 3: TODAY() Formula Detection

**Detection:** Search formula fields for performance-impacting functions.

```javascript
// Flag these in formulas
/TODAY\(\)/i
/NOW\(\)/i
```

**Why it matters:** These formulas recalculate on every base load, causing performance issues at scale.

**Severity:** Warning (suggest alternatives)

**Alternatives to suggest:**
- Use a "Last Updated" field instead
- Move calculation to automation that runs periodically
- Use DATEADD from a fixed reference date

### Check 4: Complex Formulas

**Detection:** Formula text longer than 500 characters.

**Severity:** Info (may indicate formula should be split or simplified)

### Check 5: Deep Lookup/Rollup Chains

**Detection:** Lookups that reference other lookups, or rollups of lookups.

**Severity:** Warning (performance impact, harder to debug)

---

## Step 6: Analyze Schema Quality

### Check 1: Repeated Data Across Tables

**Detection:** Same text values appearing in multiple tables without linked records.

Example: "Acme Corp" appears as text in Contacts, Projects, and Invoices tables.

**Recommendation:** Create a Companies table and link to it.

**Severity:** Warning

### Check 2: Too Many Single-Select Fields

**Detection:** Table has >10 single-select fields.

**Severity:** Info

**Recommendation:** Consider grouping related selects or extracting to separate table.

### Check 3: Missing Relationships

**Detection:** Data patterns suggesting need for linked records:
- Same field names across tables (e.g., "Company Name" in multiple tables)
- Foreign key patterns (fields ending in "ID", "Code")
- Repeated text that could be normalized

**Severity:** Info

### Check 4: Join Table Candidates

**Detection:** Many-to-many relationship patterns without junction table.

Look for:
- Multiple link fields to same table
- Long text fields containing comma-separated values
- Multiselect fields with many options

**Severity:** Info

### Check 5: Denormalization

**Detection:** Fields that could be lookups instead of duplicated data.

Example: "Client Email" in Projects table when there's already a Client link field.

**Severity:** Info

---

## Step 7: Calculate Scores

### Scoring Formula

```javascript
// Weight by category
fieldHealthWeight = 0.35;
performanceWeight = 0.35;
schemaQualityWeight = 0.30;

// Each category scores 0-100
overallScore = (fieldHealth * fieldHealthWeight) +
               (performance * performanceWeight) +
               (schemaQuality * schemaQualityWeight);
```

### Category Scoring

Deduct points for issues:

**Field Health (100 base):**
- Unused field: -5 per field
- Low fill rate: -2 per field
- "(copy)" field: -3 per field
- Low uniqueness on ID field: -5 per field

**Performance (100 base):**
- Record usage >75%: -10
- Record usage >90%: -25
- Each TODAY() formula: -10
- Each complex formula: -3
- Deep lookup chains: -5 each

**Schema Quality (100 base):**
- Repeated data pattern: -10 per pattern
- >10 single-selects: -5
- Missing obvious relationship: -5 per pattern
- Denormalization detected: -3 per instance

### Letter Grades

| Score | Grade |
|-------|-------|
| 90-100 | A |
| 80-89 | B |
| 70-79 | C |
| 60-69 | D |
| <60 | F |

---

## Step 8: Present Report

### Report Format

```markdown
═══════════════════════════════════════════════
         Base Health Report: [Base Name]
═══════════════════════════════════════════════
Overall Score: [Grade] ([Score]/100)

📊 Field Health: [Grade] ([Score]/100)
  ✓ [N] fields analyzed
  ✓ [N] fields have good fill rates (>10%)
  ⚠ [N] fields with <10% fill rate
  ⚠ [N] fields appear unused (0% fill)
  ⚠ "[Field Name]" should be renamed (has "(copy)")
  ℹ "[Field]" field: [N]% unique (good!/needs review)

📈 Performance: [Grade] ([Score]/100)
  ✓ [N] records ([N]% of [Limit] limit)
  ⚠ [N] formulas use TODAY() - consider alternatives
  ⚠ [N] complex formulas (>[N] chars)
  ⚠ Attachment storage approaching limit

🔗 Schema Quality: [Grade] ([Score]/100)
  ⚠ "[Value]" repeated in [N] tables - consider linked record
  ⚠ [Table] has [N] single-select fields
  💡 Consider extracting "[Pattern]" to separate table

───────────────────────────────────────────────
Safe fixes available:
  • Hide [N] unused fields

How would you like to proceed?
  [1] Apply safe fixes
  [2] Export full report
  [3] View detailed recommendations
═══════════════════════════════════════════════
```

---

## Step 9: Remediation Options

### Option 1: Apply Safe Fixes

Safe fixes are low-risk, reversible changes:

**Can auto-fix:**
- Hide unused fields (can be unhidden later)

**Cannot auto-fix (too risky):**
- Delete fields
- Modify formulas
- Restructure schema
- Change field types

```
Applying safe fixes...
  ✓ Hidden "Old Status" in Tasks (was unused)
  ✓ Hidden "Notes Backup" in Projects (was unused)

Done! 2 fields hidden.

Note: Hidden fields can be restored via Field menu → "Show hidden fields"
```

### Option 2: Export Full Report

Generate detailed markdown report:

```markdown
# Base Health Audit Report

**Base:** [Name]
**Generated:** [Date]
**Plan:** [Plan Type]

## Summary

| Category | Score | Grade |
|----------|-------|-------|
| Overall | [N]/100 | [Grade] |
| Field Health | [N]/100 | [Grade] |
| Performance | [N]/100 | [Grade] |
| Schema Quality | [N]/100 | [Grade] |

## Field Health Issues

### Unused Fields (0% fill rate)

| Table | Field | Type | Recommendation |
|-------|-------|------|----------------|
| Tasks | Old Status | Single Select | Hide or delete |
| ... | ... | ... | ... |

### Low Fill Rate Fields (<10%)

| Table | Field | Fill Rate | Recommendation |
|-------|-------|-----------|----------------|
| ... | ... | ... | ... |

### Fields Needing Rename

| Table | Current Name | Suggested Name |
|-------|--------------|----------------|
| Tasks | Status (copy) | [User decides] |

## Performance Issues

### TODAY() Formulas

| Table | Field | Formula | Alternative |
|-------|-------|---------|-------------|
| Tasks | Days Open | TODAY()-{Created} | Use automation to update daily |

### Complex Formulas

| Table | Field | Length | Recommendation |
|-------|-------|--------|----------------|
| ... | ... | ... | Consider splitting |

## Schema Quality Issues

### Repeated Data

| Value | Found In | Recommendation |
|-------|----------|----------------|
| "Acme Corp" | Contacts, Projects, Invoices | Create Companies table with links |

### Relationship Opportunities

| Pattern | Tables | Suggestion |
|---------|--------|------------|
| Company Name | Contacts, Projects | Link to shared Companies table |

## Manual Action Checklist

- [ ] Review and delete/keep unused fields
- [ ] Rename "(copy)" fields appropriately
- [ ] Replace TODAY() formulas with alternatives
- [ ] Consider schema changes for repeated data
```

### Option 3: Detailed Recommendations

Interactive drill-down for each issue:

```
Issue 1 of [N]: TODAY() Formula in "Days Open"

📍 Location: Tasks table → Days Open field
📝 Current formula: TODAY()-{Created}

⚠ Why this matters:
  TODAY() recalculates every time the base loads.
  With [N] records, this causes noticeable slowdown.

💡 Recommended alternatives:

  1. Automation approach:
     - Create a "Days Open Snapshot" number field
     - Set up daily automation to update: TODAY()-{Created}
     - Runs once per day instead of on every load

  2. Script approach:
     - Run a script weekly to update stale values
     - Better for bases with less frequent access

  3. Keep as-is:
     - Acceptable if real-time accuracy is critical
     - Consider if <1000 records

Would you like step-by-step instructions for any approach?
  [1] Automation setup
  [2] Script setup
  [3] Skip to next issue
```

---

## Edge Cases

### MCP Not Available

```
Airtable MCP tools not available. To use this skill:

Option 1: Configure Airtable MCP
  - Add AIRTABLE_PAT to your environment
  - Restart Claude Code

Option 2: Manual audit (limited)
  - Open your base in Airtable
  - Go to Help → API documentation
  - Share field lists for analysis
```

### Very Large Base

For bases with >20 tables or >100 fields:

```
This base has [N] tables and [N] fields.
A full audit may take several minutes.

How would you like to proceed?
  1. Full audit (recommended for first-time)
  2. Quick scan (field health only)
  3. Audit specific tables
```

### Permission Errors

```
⚠ Unable to access some tables. You may need:
  - schema.bases:read scope on your token
  - Access permissions to all tables

Continuing with accessible tables...
```

---

## Best Practices

1. **Run quarterly** - Bases accumulate cruft over time; regular audits prevent debt
2. **Start with report mode** - Review findings before applying any fixes
3. **Coordinate with team** - Notify collaborators before hiding fields
4. **Back up first** - Export base to CSV before structural changes
5. **Address performance first** - TODAY() formulas have immediate impact
6. **Schema changes last** - These require the most planning

---

## Relationship to Other Skills

- **airtable-field-audit** - Focuses on field naming issues (duplicates, whitespace, casing)
- **airtable-base-audit** - Broader health check (this skill)

Consider running both for comprehensive base maintenance:
1. `/airtable-base-audit` first for overall health
2. `/airtable-field-audit` after for naming cleanup
