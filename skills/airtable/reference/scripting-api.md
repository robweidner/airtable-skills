# Airtable Scripting API Reference

Reference for writing scripts in Airtable's Scripting app and automation script blocks.

## Script Execution Context

Scripts run in a sandboxed JavaScript environment with:
- **Vanilla JavaScript** (ES2020+)
- **Airtable SDK** via global objects (`base`, `table`, `input`, `output`)
- **No Node.js** - no `require()`, no `fs`, no `fetch` to external URLs
- **Async/await** supported

## Global Objects

### `base`
The current base. Access tables and records.

```javascript
// Get a table
let table = base.getTable("Contacts");

// Get table by ID
let table = base.getTable("tblXXXXXXXXXXXXXX");
```

### `input`
Get user input or data from automation triggers.

```javascript
// In Scripting app - show dialog
let name = await input.textAsync("Enter name:");

// In automation - get trigger data
let config = input.config();
let recordId = config.recordId;
```

### `output`
Display results to user or pass to next automation step.

```javascript
// Simple text
output.text("Done!");

// Markdown
output.markdown("# Results\n- Item 1\n- Item 2");

// Table
output.table(records.map(r => ({
  Name: r.getCellValue("Name"),
  Email: r.getCellValue("Email")
})));

// For automations - return data to next step
output.set("recordId", record.id);
output.set("status", "complete");
```

## Querying Records

### Select Records

```javascript
let table = base.getTable("Contacts");

// Get all records (loads into memory)
let query = await table.selectRecordsAsync({
  fields: ["Name", "Email", "Status"]
});

// Access records
for (let record of query.records) {
  let name = record.getCellValue("Name");
  let email = record.getCellValue("Email");
  console.log(`${name}: ${email}`);
}
```

**Options:**
```javascript
let query = await table.selectRecordsAsync({
  fields: ["Name", "Email"],     // Only load these fields
  sorts: [                        // Sort order
    {field: "Name", direction: "asc"}
  ],
  recordColorMode: "none"         // Skip record colors (faster)
});
```

### Filter with View

```javascript
let view = table.getView("Active Contacts");
let query = await view.selectRecordsAsync({
  fields: ["Name", "Email"]
});
```

### Get Single Record

```javascript
let record = await table.selectRecordAsync(recordId);
let name = record.getCellValue("Name");
```

## Creating Records

### Create Single Record

```javascript
let table = base.getTable("Contacts");

let newRecord = await table.createRecordAsync({
  "Name": "John Doe",
  "Email": "john@example.com",
  "Status": {name: "Active"}  // Single select
});

console.log("Created: " + newRecord);
```

### Create Multiple Records

```javascript
let records = [
  {fields: {"Name": "John", "Email": "john@example.com"}},
  {fields: {"Name": "Jane", "Email": "jane@example.com"}}
];

// Batch create (max 50 per call)
let newIds = await table.createRecordsAsync(records);
```

**Batch limit:** 50 records per `createRecordsAsync` call.

## Updating Records

### Update Single Record

```javascript
await table.updateRecordAsync(recordId, {
  "Status": {name: "Complete"}
});
```

### Update Multiple Records

```javascript
let updates = [
  {id: "recXXX", fields: {"Status": {name: "Complete"}}},
  {id: "recYYY", fields: {"Status": {name: "Complete"}}}
];

// Batch update (max 50 per call)
await table.updateRecordsAsync(updates);
```

## Deleting Records

```javascript
// Single
await table.deleteRecordAsync(recordId);

// Multiple (max 50)
await table.deleteRecordsAsync([recordId1, recordId2]);
```

## Working with Field Types

### Text Fields
```javascript
let text = record.getCellValue("Name");  // String or null
```

### Number/Currency/Percent
```javascript
let amount = record.getCellValue("Price");  // Number or null
```

### Single Select
```javascript
let status = record.getCellValue("Status");
// Returns: {id: "selXXX", name: "Active", color: "green"} or null

// Setting:
await table.updateRecordAsync(recordId, {
  "Status": {name: "Active"}  // Use name, not ID
});
```

### Multiple Select
```javascript
let tags = record.getCellValue("Tags");
// Returns: [{id, name, color}, ...] or null

// Setting:
await table.updateRecordAsync(recordId, {
  "Tags": [{name: "Urgent"}, {name: "Review"}]
});
```

### Date/DateTime
```javascript
let date = record.getCellValue("Due Date");
// Returns: string "2024-12-31" or null

// With time:
let datetime = record.getCellValue("Meeting");
// Returns: string "2024-12-31T14:30:00.000Z"
```

### Linked Records
```javascript
let companies = record.getCellValue("Company");
// Returns: [{id: "recXXX", name: "Acme Inc"}, ...] or null

// Setting:
await table.updateRecordAsync(recordId, {
  "Company": [{id: "recXXX"}]  // Array of record IDs
});
```

### Attachments
```javascript
let files = record.getCellValue("Documents");
// Returns: [{id, url, filename, size, type}, ...] or null

// Setting (provide URLs):
await table.updateRecordAsync(recordId, {
  "Documents": [{url: "https://example.com/file.pdf"}]
});
```

### User/Collaborator
```javascript
let assignee = record.getCellValue("Assignee");
// Returns: {id: "usrXXX", email: "user@example.com", name: "User"} or null

// Setting:
await table.updateRecordAsync(recordId, {
  "Assignee": {id: "usrXXX"}  // Use ID
});
```

### Checkbox
```javascript
let done = record.getCellValue("Completed");
// Returns: true, false, or null (treat null as false)
```

## Input Dialogs (Scripting App Only)

Not available in automation scripts - only in the Scripting app.

```javascript
// Text input
let name = await input.textAsync("Enter name:");

// Button choice
let action = await input.buttonsAsync("What to do?", ["Save", "Cancel"]);

// Record picker
let record = await input.recordAsync("Select a contact:", table);

// Table picker
let table = await input.tableAsync("Select table:");

// Field picker
let field = await input.fieldAsync("Select field:", table);
```

## Automation Script Blocks

In automations, scripts receive trigger data via `input.config()`:

```javascript
// Get trigger record
let config = input.config();
let recordId = config.recordId;
let table = base.getTable(config.tableId);

let record = await table.selectRecordAsync(recordId);
let name = record.getCellValue("Name");

// Pass data to next step
output.set("customerName", name);
output.set("success", true);
```

**Defining inputs in automation:**
When adding script to automation, define input variables that map from trigger.

## Common Patterns

### Find and Update

```javascript
let table = base.getTable("Tasks");
let query = await table.selectRecordsAsync({
  fields: ["Name", "Status"]
});

let updates = [];
for (let record of query.records) {
  let status = record.getCellValue("Status");
  if (status?.name === "Pending") {
    updates.push({
      id: record.id,
      fields: {"Status": {name: "In Progress"}}
    });
  }
}

// Batch update (50 at a time)
while (updates.length > 0) {
  let batch = updates.splice(0, 50);
  await table.updateRecordsAsync(batch);
}

output.text(`Updated ${updates.length} records`);
```

### Create Linked Records

```javascript
let contactsTable = base.getTable("Contacts");
let companiesTable = base.getTable("Companies");

// First create company
let companyId = await companiesTable.createRecordAsync({
  "Name": "Acme Inc"
});

// Then create contact linked to company
await contactsTable.createRecordAsync({
  "Name": "John Doe",
  "Company": [{id: companyId}]
});
```

### Summarize Data

```javascript
let table = base.getTable("Orders");
let query = await table.selectRecordsAsync({
  fields: ["Status", "Total"]
});

let summary = {};
for (let record of query.records) {
  let status = record.getCellValue("Status")?.name || "Unknown";
  let total = record.getCellValue("Total") || 0;

  if (!summary[status]) {
    summary[status] = {count: 0, total: 0};
  }
  summary[status].count++;
  summary[status].total += total;
}

output.table(Object.entries(summary).map(([status, data]) => ({
  Status: status,
  Count: data.count,
  Total: `$${data.total.toFixed(2)}`
})));
```

## Script vs Native Automation Steps

**Use scripts when:**
- Complex logic (loops, conditionals)
- Multiple table operations
- Custom data transformation
- Calculations across records

**Use native steps when:**
- Simple field updates
- Sending emails/notifications
- Single record operations
- You want non-developers to maintain it

**Minimal script approach:**
Script does data gathering → outputs values → native steps do actions

```javascript
// Script: Find records needing notification
let query = await table.selectRecordsAsync({fields: ["Email", "Name"]});
let toNotify = query.records
  .filter(r => /* your logic */)
  .map(r => ({
    email: r.getCellValue("Email"),
    name: r.getCellValue("Name")
  }));

// Output for native email step
output.set("recipients", toNotify);
```

Then use "Send email" action with dynamic values from script output.
