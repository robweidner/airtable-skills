# Airtable Field Types Reference

Complete reference for all Airtable field types, their API support, and emoji mappings.

## Field Type Summary

| Field Type | Emoji | Can Create via API | Can Modify via API | Notes |
|------------|-------|-------------------|-------------------|-------|
| Single line text | (none) | Yes | Yes | Basic text field |
| Long text | [text] | Yes | Yes | Rich text supported |
| Attachment | [attach] | Yes | Yes | URLs to files |
| Checkbox | [check] | Yes | Yes | Boolean |
| Multiple select | [multi] | Yes | Yes | Define options on create |
| Single select | [select] | Yes | Yes | Define options on create |
| User | [user] | Yes | Limited | Collaborator field |
| Date | [date] | Yes | Yes | With optional time |
| Phone number | [phone] | Yes | Yes | Formatted phone |
| Email | [email] | Yes | Yes | Email validation |
| URL | [url] | Yes | Yes | URL validation |
| Number | [num] | Yes | Yes | Integer or decimal |
| Currency | [money] | Yes | Yes | With currency symbol |
| Percent | [pct] | Yes | Yes | Displayed as % |
| Duration | [time] | Yes | Yes | h:mm:ss format |
| Rating | [star] | Yes | Yes | 1-10 scale |
| Formula | [formula] | **No** | **No** | Use workaround |
| Rollup | [rollup] | **No** | **No** | Use workaround |
| Count | [count] | Yes | Limited | Counts linked records |
| Lookup | [lookup] | **No** | **No** | Use workaround |
| Created time | [created] | Yes | No | Auto-populated |
| Last modified time | [modified] | Yes | No | Auto-populated |
| Created by | [user] | Yes | No | Auto-populated |
| Last modified by | [user] | Yes | No | Auto-populated |
| Autonumber | [auto] | Yes | No | Auto-incrementing |
| Barcode | [barcode] | Yes | Yes | Barcode/QR |
| Button | [button] | Limited | Limited | Triggers actions |
| Link to another record | [link] | Yes | Yes | Related records |

## Fields That Cannot Be Created via API

These fields require manual creation or the workaround pattern:

### Formula Fields
**Limitation:** API cannot create formula fields.

**Workaround:**
1. Create as `singleLineText` field
2. Add `[convert]` prefix to name
3. Put formula in field description
4. After API creation, manually convert field type and paste formula

```json
{
  "name": "[convert][formula] Full Name",
  "type": "singleLineText",
  "description": "CONVERT TO FORMULA: CONCATENATE({First Name}, \" \", {Last Name})"
}
```

### Rollup Fields
**Limitation:** API cannot create rollup fields.

**Workaround:** Same as formula - create as text, document the rollup configuration in description.

```json
{
  "name": "[convert][rollup] Total Orders",
  "type": "singleLineText",
  "description": "CONVERT TO ROLLUP: Link field: Orders, Summarize: COUNT(values)"
}
```

### Lookup Fields
**Limitation:** API cannot create lookup fields.

**Workaround:** Same pattern - create as text with lookup config in description.

```json
{
  "name": "[convert][lookup] Customer Email",
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
  "name": "[text] Description",
  "type": "multilineText"
}
```
Supports markdown formatting when rich text is enabled.

### Attachment
```json
{
  "name": "[attach] Documents",
  "type": "multipleAttachments"
}
```
When creating records, provide array of `{url: "..."}` objects.

### Checkbox
```json
{
  "name": "[check] Completed",
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
  "name": "[multi] Tags",
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
  "name": "[select] Status",
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
  "name": "[user] Assignee",
  "type": "singleCollaborator"
}
```
For multiple users, use `multipleCollaborators`.

### Date
```json
{
  "name": "[date] Due Date",
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
  "name": "[date] Meeting Time",
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
  "name": "[phone] Phone",
  "type": "phoneNumber"
}
```

### Email
```json
{
  "name": "[email] Email",
  "type": "email"
}
```

### URL
```json
{
  "name": "[url] Website",
  "type": "url"
}
```

### Number
```json
{
  "name": "[num] Quantity",
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
  "name": "[money] Price",
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
  "name": "[pct] Progress",
  "type": "percent",
  "options": {
    "precision": 0
  }
}
```

### Duration
```json
{
  "name": "[time] Time Spent",
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
  "name": "[star] Priority",
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
  "name": "[link] Company",
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
  "name": "[created] Created",
  "type": "createdTime"
}
```
```json
{
  "name": "[modified] Updated",
  "type": "lastModifiedTime"
}
```

### Created By / Last Modified By
```json
{
  "name": "[user] Created By",
  "type": "createdBy"
}
```
```json
{
  "name": "[user] Modified By",
  "type": "lastModifiedBy"
}
```

### Autonumber
```json
{
  "name": "[auto] ID",
  "type": "autoNumber"
}
```

### Barcode
```json
{
  "name": "[barcode] Product Code",
  "type": "barcode"
}
```

### Button
```json
{
  "name": "[button] Open URL",
  "type": "button",
  "options": {
    "label": "Open",
    "style": "primary"
  }
}
```
Button actions are limited via API - typically configured in UI.

## Recommended Standard Fields

When creating any new table, suggest these standard fields:

```json
[
  {"name": "[auto] ID", "type": "autoNumber"},
  {"name": "[created] Created", "type": "createdTime"},
  {"name": "[modified] Updated", "type": "lastModifiedTime"},
  {"name": "[user] Created By", "type": "createdBy"},
  {"name": "[user] Modified By", "type": "lastModifiedBy"}
]
```

Let user opt out if they don't need them.
