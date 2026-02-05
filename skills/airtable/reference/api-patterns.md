# Airtable REST API Patterns

Reference for interacting with Airtable's Web API.

## Authentication

All requests require a Personal Access Token (PAT) in the Authorization header:

```bash
curl "https://api.airtable.com/v0/{baseId}/{tableName}" \
  -H "Authorization: Bearer YOUR_PAT_HERE"
```

**Important:** Create scoped PATs per project. See [pat-security.md](pat-security.md).

## Base URL

```
https://api.airtable.com/v0/{baseId}/{tableIdOrName}
```

- `baseId` - Starts with `app` (e.g., `appXXXXXXXXXXXXXX`)
- `tableIdOrName` - Table ID (starts with `tbl`) or URL-encoded table name

**Get IDs from URL:** `https://airtable.com/appXXX/tblYYY/viwZZZ`

## Rate Limiting

- **5 requests per second** per base
- Returns `429 Too Many Requests` when exceeded
- Implement exponential backoff for retries

```javascript
async function withRetry(fn, maxRetries = 3) {
  for (let i = 0; i < maxRetries; i++) {
    try {
      return await fn();
    } catch (error) {
      if (error.status === 429 && i < maxRetries - 1) {
        await new Promise(r => setTimeout(r, Math.pow(2, i) * 1000));
        continue;
      }
      throw error;
    }
  }
}
```

## Reading Records

### List Records

```bash
GET /v0/{baseId}/{tableName}
```

**Query parameters:**
- `fields[]` - Only return specified fields
- `filterByFormula` - Airtable formula to filter
- `maxRecords` - Limit results (default: 100)
- `pageSize` - Records per page (max: 100)
- `offset` - Pagination cursor
- `sort[0][field]` - Sort by field
- `sort[0][direction]` - `asc` or `desc`
- `view` - Use a specific view's filters/sorts

**Example - Get active contacts:**
```bash
curl "https://api.airtable.com/v0/appXXX/Contacts?filterByFormula=Status='Active'&fields[]=Name&fields[]=Email" \
  -H "Authorization: Bearer $AIRTABLE_PAT"
```

**Pagination:**
```javascript
let offset = null;
const allRecords = [];

do {
  const url = `https://api.airtable.com/v0/${baseId}/${tableName}` +
    (offset ? `?offset=${offset}` : '');
  const response = await fetch(url, {headers});
  const data = await response.json();
  allRecords.push(...data.records);
  offset = data.offset;
} while (offset);
```

### Get Single Record

```bash
GET /v0/{baseId}/{tableName}/{recordId}
```

## Creating Records

### Create Records (Batch)

```bash
POST /v0/{baseId}/{tableName}
Content-Type: application/json

{
  "records": [
    {
      "fields": {
        "Name": "John Doe",
        "Email": "john@example.com",
        "Status": "Active"
      }
    },
    {
      "fields": {
        "Name": "Jane Doe",
        "Email": "jane@example.com"
      }
    }
  ]
}
```

**Limits:**
- Max **10 records** per request
- For more, make multiple requests

```javascript
async function createRecords(records) {
  const batches = [];
  for (let i = 0; i < records.length; i += 10) {
    batches.push(records.slice(i, i + 10));
  }

  const results = [];
  for (const batch of batches) {
    const response = await fetch(url, {
      method: 'POST',
      headers,
      body: JSON.stringify({ records: batch.map(r => ({ fields: r })) })
    });
    const data = await response.json();
    results.push(...data.records);

    // Respect rate limit
    await new Promise(r => setTimeout(r, 200));
  }
  return results;
}
```

## Updating Records

### Update Records (PATCH - Partial)

Only updates specified fields, leaves others unchanged.

```bash
PATCH /v0/{baseId}/{tableName}
Content-Type: application/json

{
  "records": [
    {
      "id": "recXXXXXXXXXXXXXX",
      "fields": {
        "Status": "Complete"
      }
    }
  ]
}
```

### Replace Records (PUT - Full)

Clears unspecified fields.

```bash
PUT /v0/{baseId}/{tableName}
Content-Type: application/json

{
  "records": [
    {
      "id": "recXXXXXXXXXXXXXX",
      "fields": {
        "Name": "John Doe",
        "Email": "john@example.com",
        "Status": "Complete"
      }
    }
  ]
}
```

**Limits:** Max 10 records per request (same as create)

## Deleting Records

```bash
DELETE /v0/{baseId}/{tableName}?records[]={recordId1}&records[]={recordId2}
```

**Limits:** Max 10 records per request

## Schema Operations

### List Bases

```bash
GET /v0/meta/bases
```

Returns all bases the token can access.

### Get Base Schema

```bash
GET /v0/meta/bases/{baseId}/tables
```

Returns all tables and their fields.

### Create Table

```bash
POST /v0/meta/bases/{baseId}/tables
Content-Type: application/json

{
  "name": "Contacts",
  "fields": [
    {
      "name": "Name",
      "type": "singleLineText"
    },
    {
      "name": "Email",
      "type": "email"
    },
    {
      "name": "Status",
      "type": "singleSelect",
      "options": {
        "choices": [
          {"name": "Active", "color": "greenBright"},
          {"name": "Inactive", "color": "grayBright"}
        ]
      }
    }
  ]
}
```

### Create Field

```bash
POST /v0/meta/bases/{baseId}/tables/{tableId}/fields
Content-Type: application/json

{
  "name": "Phone",
  "type": "phoneNumber"
}
```

### Update Field

```bash
PATCH /v0/meta/bases/{baseId}/tables/{tableId}/fields/{fieldId}
Content-Type: application/json

{
  "name": "Mobile Phone",
  "description": "Primary contact number"
}
```

## Error Handling

### Common Error Codes

| Status | Meaning | Action |
|--------|---------|--------|
| 400 | Invalid request | Check field names, types, values |
| 401 | Unauthorized | Check PAT is valid |
| 403 | Forbidden | PAT lacks required scope |
| 404 | Not found | Check base/table/record IDs |
| 422 | Unprocessable | Validation error (see body) |
| 429 | Rate limited | Wait and retry with backoff |
| 500+ | Server error | Retry later |

### Error Response Format

```json
{
  "error": {
    "type": "INVALID_REQUEST_BODY",
    "message": "Could not parse request body"
  }
}
```

### Validation Errors

```json
{
  "error": {
    "type": "INVALID_VALUE_FOR_COLUMN",
    "message": "Field \"Status\" cannot accept the provided value"
  }
}
```

## Working with Specific Field Types

### Linked Records

```json
{
  "fields": {
    "Company": ["recXXXXXXXXXXXXXX"]
  }
}
```

Array of record IDs from the linked table.

### Attachments

```json
{
  "fields": {
    "Documents": [
      {"url": "https://example.com/file.pdf"}
    ]
  }
}
```

Airtable will download and host the file.

### Multiple Select

```json
{
  "fields": {
    "Tags": ["Urgent", "Review"]
  }
}
```

Array of option names (must exist in field options).

### Collaborators

```json
{
  "fields": {
    "Assignee": {"email": "user@example.com"}
  }
}
```

Or use `{"id": "usrXXX"}` if you have the user ID.

### Dates

```json
{
  "fields": {
    "Due Date": "2024-12-31",
    "Meeting Time": "2024-12-31T14:30:00.000Z"
  }
}
```

ISO 8601 format. For datetime, include timezone or use UTC.
