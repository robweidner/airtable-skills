# Airtable Field Types Reference

Complete reference for all Airtable field types, their API support, and emoji mappings.

## Field Type Summary

| Field Type | Emoji | Can Create via API | Can Modify via API | Notes |
|------------|-------|-------------------|-------------------|-------|
| Single line text | (none) | Yes | Yes | Basic text field |
| Long text | 📝 | Yes | Yes | Rich text supported |
| Attachment | 📎 | Yes | Yes | URLs to files |
| Checkbox | ☑️ | Yes | Yes | Boolean |
| Multiple select | 🔢 | Yes | Yes | Define options on create |
| Single select | 1️⃣ | Yes | Yes | Define options on create |
| User | 👤 | Yes | Limited | Collaborator field |
| Date | 📅 | Yes | Yes | With optional time |
| Phone number | 📞 | Yes | Yes | Formatted phone |
| Email | 📧 | Yes | Yes | Email validation |
| URL | 🌍 | Yes | Yes | URL validation |
| Number | #️⃣ | Yes | Yes | Integer or decimal |
| Currency | 💰 | Yes | Yes | With currency symbol |
| Percent | 📊 | Yes | Yes | Displayed as % |
| Duration | ⏱️ | Yes | Yes | h:mm:ss format |
| Rating | ⭐ | Yes | Yes | 1-10 scale |
| Formula | 🤖 | **No** | **No** | Use workaround |
| Rollup | 🤖 | **No** | **No** | Use workaround |
| Count | 🧮 | Yes | Limited | Counts linked records (supports conditional filtering) |
| Lookup | 🔎 | **No** | **No** | Use workaround |
| Created time | 📅 | Yes | No | Auto-populated |
| Last modified time | 📅 | Yes | No | Auto-populated |
| Created by | 👤 | Yes | No | Auto-populated |
| Last modified by | 👤 | Yes | No | Auto-populated |
| Autonumber | 🆔 | Yes | No | Auto-incrementing |
| Barcode | | Yes | Yes | Barcode/QR |
| Button | 👊 | Limited | Limited | Triggers actions |
| Link to another record | 🔗 | Yes | Yes | Related records |

## Conditional Filtering on Count, Lookup, and Rollup Fields

**Count, Lookup, and Rollup fields all support conditional filtering** on linked records in the Airtable UI. This is configured via:

> "Only include linked records from [Table] that meet certain conditions"

This toggle enables a full condition builder (field + operator + value) with AND/OR groups.

**Example:** A Count field on Companies that counts linked Properties where `Aging Bucket` is "Past Due" — gives a filtered count of only past-due properties, not all linked properties.

**Important:** The Airtable API (`describe_table`) does **NOT** expose these filter conditions. The API only shows the base field type and linked record field ID. Do not assume filters are absent just because the API doesn't show them.

**This eliminates the need for workarounds** like "binary formula + SUM rollup" patterns for filtered counting. Use Count fields with conditions directly instead.

## Fields That Cannot Be Created via API

These fields require manual creation or the workaround pattern:

### Formula Fields
**Limitation:** API cannot create formula fields.

**Workaround:**
1. Create as `singleLineText` field
2. Add `🔧` prefix to name
3. Put formula in field description
4. After API creation, manually convert field type and paste formula

```json
{
  "name": "🔧🤖 Full Name",
  "type": "singleLineText",
  "description": "CONVERT TO FORMULA: CONCATENATE({First Name}, \" \", {Last Name})"
}
```

### Rollup Fields
**Limitation:** API cannot create rollup fields.

**Workaround:** Same as formula - create as text, document the rollup configuration in description.

```json
{
  "name": "🔧🤖 Total Orders",
  "type": "singleLineText",
  "description": "CONVERT TO ROLLUP: Link field: Orders, Summarize: COUNT(values)"
}
```

### Lookup Fields
**Limitation:** API cannot create lookup fields.

**Workaround:** Same pattern - create as text with lookup config in description.

```json
{
  "name": "🔧🔎 Customer Email",
  "type": "singleLineText",
  "description": "CONVERT TO LOOKUP: Link field: Customer, Field: Email"
}
```

## Field Type Details

### Single Line Text
```json
{
  "name": "Title",
  "type": "singleLineText"
}
```

### Long Text (Rich Text)
```json
{
  "name": "📝 Description",
  "type": "multilineText"
}
```
Supports markdown formatting when rich text is enabled.

### Attachment
```json
{
  "name": "📎 Documents",
  "type": "multipleAttachments"
}
```
When creating records, provide array of `{url: "..."}` objects.

### Checkbox
```json
{
  "name": "☑️ Completed",
  "type": "checkbox",
  "options": {
    "icon": "check",
    "color": "greenBright"
  }
}
```

### Multiple Select
```json
{
  "name": "🔢 Tags",
  "type": "multipleSelects",
  "options": {
    "choices": [
      {"name": "Urgent", "color": "redBright"},
      {"name": "Review", "color": "yellowBright"},
      {"name": "Done", "color": "greenBright"}
    ]
  }
}
```
**Important:** Define all choices upfront. Adding choices later requires a separate API call.

### Single Select
```json
{
  "name": "1️⃣ Status",
  "type": "singleSelect",
  "options": {
    "choices": [
      {"name": "To Do", "color": "blueBright"},
      {"name": "In Progress", "color": "yellowBright"},
      {"name": "Done", "color": "greenBright"}
    ]
  }
}
```

### User (Collaborator)
```json
{
  "name": "👤 Assignee",
  "type": "singleCollaborator"
}
```
For multiple users, use `multipleCollaborators`.

### Date
```json
{
  "name": "📅 Due Date",
  "type": "date",
  "options": {
    "dateFormat": {
      "name": "local"
    }
  }
}
```
With time:
```json
{
  "name": "📅 Meeting Time",
  "type": "dateTime",
  "options": {
    "dateFormat": {"name": "local"},
    "timeFormat": {"name": "12hour"},
    "timeZone": "client"
  }
}
```

### Phone Number
```json
{
  "name": "📞 Phone",
  "type": "phoneNumber"
}
```

### Email
```json
{
  "name": "📧 Email",
  "type": "email"
}
```

### URL
```json
{
  "name": "🌍 Website",
  "type": "url"
}
```

### Number
```json
{
  "name": "#️⃣ Quantity",
  "type": "number",
  "options": {
    "precision": 0
  }
}
```
For decimals, set precision (0-8).

### Currency
```json
{
  "name": "💰 Price",
  "type": "currency",
  "options": {
    "precision": 2,
    "symbol": "$"
  }
}
```

### Percent
```json
{
  "name": "📊 Progress",
  "type": "percent",
  "options": {
    "precision": 0
  }
}
```

### Duration
```json
{
  "name": "⏱️ Time Spent",
  "type": "duration",
  "options": {
    "durationFormat": "h:mm"
  }
}
```
Formats: `h:mm`, `h:mm:ss`, `h:mm:ss.S`, `h:mm:ss.SS`, `h:mm:ss.SSS`

### Rating
```json
{
  "name": "⭐ Priority",
  "type": "rating",
  "options": {
    "max": 5,
    "icon": "star",
    "color": "yellowBright"
  }
}
```

### Link to Another Record
```json
{
  "name": "🔗 Company",
  "type": "multipleRecordLinks",
  "options": {
    "linkedTableId": "tblXXXXXXXXXXXXXX"
  }
}
```
**Note:** The linked table must exist first. Get its ID before creating this field.

### Created Time / Last Modified Time
```json
{
  "name": "📅 Created",
  "type": "createdTime"
}
```
```json
{
  "name": "📅 Updated",
  "type": "lastModifiedTime"
}
```

### Created By / Last Modified By
```json
{
  "name": "👤 Created By",
  "type": "createdBy"
}
```
```json
{
  "name": "👤 Modified By",
  "type": "lastModifiedBy"
}
```

### Autonumber
```json
{
  "name": "🆔 ID",
  "type": "autoNumber"
}
```

### Barcode
```json
{
  "name": "Product Code",
  "type": "barcode"
}
```

### Button
```json
{
  "name": "👊 Open URL",
  "type": "button",
  "options": {
    "label": "Open",
    "style": "primary"
  }
}
```
Button actions are limited via API - typically configured in UI.

## Fields That Cannot Be Deleted via API

**The Airtable API and MCP have no delete field or delete table capability.** Fields and tables can only be deleted in the Airtable UI.

**Never offer or propose to delete fields via API.** When fields need to be removed, provide the user with a list of field names and IDs to delete manually in the UI.

## Recommended Standard Fields

When creating any new table, suggest these standard fields:

```json
[
  {"name": "🆔 ID", "type": "autoNumber"},
  {"name": "📅 Created", "type": "createdTime"},
  {"name": "📅 Updated", "type": "lastModifiedTime"},
  {"name": "👤 Created By", "type": "createdBy"},
  {"name": "👤 Modified By", "type": "lastModifiedBy"}
]
```

Let user opt out if they don't need them.
