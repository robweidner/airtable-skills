# Airtable MCP Patterns

Reference for using the Airtable MCP (Model Context Protocol) tools when available.

## When to Use MCP vs REST API

| Use MCP when... | Use REST API when... |
|-----------------|---------------------|
| MCP is configured and available | MCP not available |
| Working interactively | Writing scripts for automation |
| Need quick exploration | Need precise control |
| Complex queries | Batch operations > 10 records |

**Check MCP availability:** Look for `mcp__airtable__*` tools in your tool list.

## Available MCP Tools

### Discovery Tools

#### list_bases
Lists all bases accessible to the configured token.

```
mcp__airtable__list_bases
```

Returns array of `{id, name, permissionLevel}`.

#### list_tables
Lists tables in a base.

```
mcp__airtable__list_tables
  baseId: "appXXXXXXXXXXXXXX"
  detailLevel: "identifiersOnly"  # or "full" for field details
```

**Tip:** Use `identifiersOnly` to save tokens when you just need table IDs/names.

#### describe_table
Gets full schema for a single table.

```
mcp__airtable__describe_table
  baseId: "appXXXXXXXXXXXXXX"
  tableId: "tblXXXXXXXXXXXXXX"
```

Returns all fields with their types, options, and IDs.

### Reading Records

#### list_records
Retrieves records from a table.

```
mcp__airtable__list_records
  baseId: "appXXXXXXXXXXXXXX"
  tableId: "tblXXXXXXXXXXXXXX"
  filterByFormula: "Status='Active'"
  fields: ["Name", "Email", "Status"]
  maxRecords: 100
```

**Parameters:**
- `filterByFormula` - Airtable formula to filter records
- `fields` - Array of field names to return
- `maxRecords` - Limit number of records
- `sort` - Array of `{field, direction}` objects
- `view` - View name or ID to use

**Large results:** May be saved to a temp file. Use `jq` or similar to parse.

#### search_records
Full-text search across records.

```
mcp__airtable__search_records
  baseId: "appXXXXXXXXXXXXXX"
  tableId: "tblXXXXXXXXXXXXXX"
  searchTerm: "john doe"
```

Searches all text fields.

#### get_record
Gets a single record by ID.

```
mcp__airtable__get_record
  baseId: "appXXXXXXXXXXXXXX"
  tableId: "tblXXXXXXXXXXXXXX"
  recordId: "recXXXXXXXXXXXXXX"
```

### Writing Records

#### create_record
Creates a single record.

```
mcp__airtable__create_record
  baseId: "appXXXXXXXXXXXXXX"
  tableId: "tblXXXXXXXXXXXXXX"
  fields: {
    "Name": "John Doe",
    "Email": "john@example.com",
    "Status": "Active"
  }
```

**Note:** Creates one record at a time. For batch creation, use REST API or call multiple times.

#### update_records
Updates existing records.

```
mcp__airtable__update_records
  baseId: "appXXXXXXXXXXXXXX"
  tableId: "tblXXXXXXXXXXXXXX"
  records: [
    {
      "id": "recXXXXXXXXXXXXXX",
      "fields": {"Status": "Complete"}
    },
    {
      "id": "recYYYYYYYYYYYYYY",
      "fields": {"Status": "Complete"}
    }
  ]
```

**Limit:** Max 10 records per call. Loop for larger batches.

### Schema Operations

#### create_table
Creates a new table with fields.

```
mcp__airtable__create_table
  baseId: "appXXXXXXXXXXXXXX"
  name: "Contacts"
  fields: [
    {"name": "Name", "type": "singleLineText"},
    {"name": "Email", "type": "email"},
    {"name": "Status", "type": "singleSelect", "options": {
      "choices": [
        {"name": "Active", "color": "greenBright"},
        {"name": "Inactive", "color": "grayBright"}
      ]
    }}
  ]
```

#### create_field
Adds a field to an existing table.

```
mcp__airtable__create_field
  baseId: "appXXXXXXXXXXXXXX"
  tableId: "tblXXXXXXXXXXXXXX"
  name: "Phone"
  type: "phoneNumber"
```

## Common Patterns

### Get Base ID from URL

Extract from Airtable URL: `https://airtable.com/appXXX/tblYYY/viwZZZ`
- Base ID: `appXXX`
- Table ID: `tblYYY`
- View ID: `viwZZZ`

### Explore a Base

```
1. mcp__airtable__list_bases
   → Find base by name, get ID

2. mcp__airtable__list_tables(baseId, detailLevel: "identifiersOnly")
   → See all tables

3. mcp__airtable__describe_table(baseId, tableId)
   → Get field details for specific table
```

### Create Table with Standard Fields

```
mcp__airtable__create_table
  baseId: "appXXX"
  name: "Contacts"
  fields: [
    {"name": "Name", "type": "singleLineText"},
    {"name": "Email", "type": "email"},
    {"name": "🆔 ID", "type": "autoNumber"},
    {"name": "📅 Created", "type": "createdTime"},
    {"name": "📅 Updated", "type": "lastModifiedTime"}
  ]
```

### Batch Update Pattern

MCP limits updates to 10 records. For larger batches:

```
1. Query records that need updating
   mcp__airtable__list_records(filterByFormula: "Status='Pending'")

2. Group into batches of 10

3. For each batch:
   mcp__airtable__update_records(records: batch)
   (wait 200ms between batches for rate limit)
```

### Find and Update Pattern

```
1. mcp__airtable__search_records
     searchTerm: "john doe"
   → Get matching record IDs

2. mcp__airtable__update_records
     records: [{id: "recXXX", fields: {Status: "Updated"}}]
```

## Error Handling

MCP tools return errors in a consistent format:

```json
{
  "error": {
    "type": "INVALID_REQUEST",
    "message": "Description of what went wrong"
  }
}
```

**Common errors:**

| Error Type | Cause | Solution |
|------------|-------|----------|
| `INVALID_REQUEST` | Bad parameters | Check field names, types |
| `NOT_FOUND` | Bad ID | Verify base/table/record IDs |
| `AUTHENTICATION_REQUIRED` | Token issue | Check MCP configuration |
| `INVALID_PERMISSIONS` | Scope issue | Token needs more scopes |

## MCP Configuration

The Airtable MCP requires configuration with your PAT. Typically set in:
- `~/.claude/mcp_settings.json`
- Or project-specific MCP configuration

```json
{
  "mcpServers": {
    "airtable": {
      "command": "npx",
      "args": ["-y", "airtable-mcp-server"],
      "env": {
        "AIRTABLE_API_KEY": "${AIRTABLE_PAT}"
      }
    }
  }
}
```

If MCP isn't configured, fall back to REST API patterns.

## MCP Limitations

### Cannot Delete Fields or Tables
The MCP has no `delete_field` or `delete_table` tool. Fields and tables can only be deleted in the Airtable UI. **Never offer to delete fields via MCP — always provide a list of fields for the user to delete manually.**

### Cannot Create Formula, Rollup, or Lookup Fields
Returns `UNSUPPORTED_FIELD_TYPE_FOR_CREATE`. Use the `🔧` placeholder pattern (create as `singleLineText` with conversion instructions in the description).

### Filter Conditions Not Exposed
Count, Lookup, and Rollup fields support conditional filtering in the Airtable UI, but `describe_table` does **not** expose these filter conditions. The API only returns the base field type and link field ID. Do not assume filters are absent just because the API doesn't show them.
