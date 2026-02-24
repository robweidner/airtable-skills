# Airtable Scripting Patterns

Real-world script examples for common Airtable operations.

## Pattern 1: Find and Update Records

Find records matching criteria and update them.

```javascript
// Find all "Pending" tasks and mark as "In Progress"
let table = base.getTable("Tasks");
let query = await table.selectRecordsAsync({
    fields: ["Name", "Status"]
});

let toUpdate = [];
for (let record of query.records) {
    let status = record.getCellValue("Status");
    if (status?.name === "Pending") {
        toUpdate.push({
            id: record.id,
            fields: {
                "Status": { name: "In Progress" }
            }
        });
    }
}

// Batch update (50 at a time)
while (toUpdate.length > 0) {
    let batch = toUpdate.splice(0, 50);
    await table.updateRecordsAsync(batch);
}

output.text(`Updated ${toUpdate.length} records to "In Progress"`);
```

## Pattern 2: Create Linked Records

Create records in multiple tables with relationships.

```javascript
let companiesTable = base.getTable("Companies");
let contactsTable = base.getTable("Contacts");

// Get company data (or use input)
let companyName = await input.textAsync("Company name:");
let contactName = await input.textAsync("Contact name:");
let contactEmail = await input.textAsync("Contact email:");

// Create company first
let companyId = await companiesTable.createRecordAsync({
    "Name": companyName
});

// Create contact linked to company
let contactId = await contactsTable.createRecordAsync({
    "Name": contactName,
    "Email": contactEmail,
    "Company": [{ id: companyId }]  // Link to company
});

output.markdown(`
## Created:
- Company: ${companyName} (${companyId})
- Contact: ${contactName} (${contactId})
`);
```

## Pattern 3: Summarize and Report

Aggregate data and display a summary.

```javascript
let ordersTable = base.getTable("Orders");
let query = await ordersTable.selectRecordsAsync({
    fields: ["Status", "Total", "Customer"]
});

// Aggregate by status
let summary = {};
let grandTotal = 0;

for (let record of query.records) {
    let status = record.getCellValue("Status")?.name || "Unknown";
    let total = record.getCellValue("Total") || 0;

    if (!summary[status]) {
        summary[status] = { count: 0, total: 0 };
    }
    summary[status].count++;
    summary[status].total += total;
    grandTotal += total;
}

// Format output
let rows = Object.entries(summary).map(([status, data]) => ({
    Status: status,
    Orders: data.count,
    Total: `$${data.total.toFixed(2)}`
}));

output.markdown(`# Order Summary`);
output.table(rows);
output.markdown(`**Grand Total:** $${grandTotal.toFixed(2)}`);
```

## Pattern 4: Duplicate Records

Copy records from one table to another (or same table).

```javascript
let sourceTable = base.getTable("Templates");
let destTable = base.getTable("Projects");

// Select template to copy
let template = await input.recordAsync("Select template:", sourceTable);
if (!template) {
    output.text("No template selected");
    return;
}

// Get template values
let name = template.getCellValue("Name");
let description = template.getCellValue("Description");
let tasks = template.getCellValue("Default Tasks");

// Create new project
let projectId = await destTable.createRecordAsync({
    "Name": `Copy of ${name}`,
    "Description": description,
    "Status": { name: "New" }
    // Note: Linked records may need separate handling
});

output.text(`Created project: ${projectId}`);
```

## Pattern 5: Bulk Import from CSV-like Input

Parse multi-line input and create records.

```javascript
let table = base.getTable("Contacts");

let input_data = await input.textAsync(
    "Paste data (Name,Email per line):"
);

let lines = input_data.trim().split("\n");
let records = [];
let errors = [];

for (let line of lines) {
    let parts = line.split(",").map(p => p.trim());
    if (parts.length >= 2) {
        records.push({
            fields: {
                "Name": parts[0],
                "Email": parts[1]
            }
        });
    } else {
        errors.push(line);
    }
}

// Create in batches
let created = 0;
while (records.length > 0) {
    let batch = records.splice(0, 50);
    await table.createRecordsAsync(batch);
    created += batch.length;
}

output.markdown(`
## Import Complete
- Created: ${created} records
- Skipped: ${errors.length} lines
`);

if (errors.length > 0) {
    output.markdown("### Skipped lines:");
    output.text(errors.join("\n"));
}
```

## Pattern 6: Calculate and Update (Automation Script)

For use in automation script blocks.

```javascript
// Input from automation trigger
let config = input.config();
let recordId = config.recordId;

let table = base.getTable("Orders");
let record = await table.selectRecordAsync(recordId);

// Get line items (linked records)
let lineItems = record.getCellValue("Line Items") || [];
let subtotal = 0;

// Calculate subtotal from linked records
let itemsTable = base.getTable("Line Items");
for (let item of lineItems) {
    let itemRecord = await itemsTable.selectRecordAsync(item.id);
    let qty = itemRecord.getCellValue("Quantity") || 0;
    let price = itemRecord.getCellValue("Price") || 0;
    subtotal += qty * price;
}

// Calculate totals
let taxRate = 0.08;
let tax = subtotal * taxRate;
let total = subtotal + tax;

// Update order
await table.updateRecordAsync(recordId, {
    "Subtotal": subtotal,
    "Tax": tax,
    "Total": total
});

// Output for next automation step
output.set("orderId", recordId);
output.set("total", total);
output.set("success", true);
```

## Pattern 7: Clean Up / Deduplicate

Find and handle duplicate records.

```javascript
let table = base.getTable("Contacts");
let query = await table.selectRecordsAsync({
    fields: ["Email", "Name", "Created"]
});

// Group by email
let byEmail = {};
for (let record of query.records) {
    let email = record.getCellValue("Email");
    if (!email) continue;

    email = email.toLowerCase();
    if (!byEmail[email]) {
        byEmail[email] = [];
    }
    byEmail[email].push(record);
}

// Find duplicates
let duplicates = [];
for (let [email, records] of Object.entries(byEmail)) {
    if (records.length > 1) {
        // Keep first (oldest), mark rest as duplicates
        records.sort((a, b) => {
            let aDate = a.getCellValue("Created") || "";
            let bDate = b.getCellValue("Created") || "";
            return aDate.localeCompare(bDate);
        });

        for (let i = 1; i < records.length; i++) {
            duplicates.push({
                email: email,
                record: records[i],
                keepRecord: records[0]
            });
        }
    }
}

// Report duplicates (don't auto-delete)
output.markdown(`# Duplicate Report`);
output.markdown(`Found ${duplicates.length} duplicate records`);

if (duplicates.length > 0) {
    output.table(duplicates.map(d => ({
        "Email": d.email,
        "Duplicate ID": d.record.id,
        "Duplicate Name": d.record.getCellValue("Name"),
        "Keep ID": d.keepRecord.id,
        "Keep Name": d.keepRecord.getCellValue("Name")
    })));
}
```

## Pattern 8: Interactive Menu

Build a multi-step workflow with user choices.

```javascript
let action = await input.buttonsAsync(
    "What would you like to do?",
    [
        { label: "Add Contact", value: "add" },
        { label: "Search", value: "search" },
        { label: "Report", value: "report" }
    ]
);

if (action === "add") {
    let name = await input.textAsync("Name:");
    let email = await input.textAsync("Email:");

    let table = base.getTable("Contacts");
    let id = await table.createRecordAsync({
        "Name": name,
        "Email": email
    });
    output.text(`Created contact: ${id}`);

} else if (action === "search") {
    let query = await input.textAsync("Search term:");
    let table = base.getTable("Contacts");
    let records = await table.selectRecordsAsync({ fields: ["Name", "Email"] });

    let matches = records.records.filter(r =>
        r.getCellValueAsString("Name").toLowerCase().includes(query.toLowerCase()) ||
        r.getCellValueAsString("Email").toLowerCase().includes(query.toLowerCase())
    );

    output.markdown(`## Found ${matches.length} matches`);
    output.table(matches.map(r => ({
        Name: r.getCellValue("Name"),
        Email: r.getCellValue("Email")
    })));

} else if (action === "report") {
    // Run report logic...
    output.text("Report generated!");
}
```

## Tips

1. **Always specify fields** in `selectRecordsAsync` - loads faster
2. **Use batches of 50** for create/update/delete operations
3. **Handle null values** - fields can be null
4. **Test with small data** before running on full table
5. **Use output.table()** for structured data display
6. **Add progress indicators** for long operations
7. **Automation scripts cannot use `return` outside of a function** - this is only supported in Extension scripts