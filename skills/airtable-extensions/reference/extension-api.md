# Airtable Extension API Reference

SDK hooks and methods for building Airtable extensions.

## Hooks

### useBase

Access the current base.

```jsx
import { useBase } from '@airtable/blocks/ui';

function MyComponent() {
  const base = useBase();

  // Get table by name
  const table = base.getTableByName('Contacts');

  // Get table by ID
  const table = base.getTableById('tblXXXXXXXXXXXXXX');

  // List all tables
  const tables = base.tables;

  return <div>Base: {base.name}</div>;
}
```

### useRecords

Subscribe to records from a table or view.

```jsx
import { useRecords } from '@airtable/blocks/ui';

function RecordList() {
  const base = useBase();
  const table = base.getTableByName('Contacts');

  // All records
  const allRecords = useRecords(table);

  // From specific view
  const view = table.getViewByName('Active');
  const viewRecords = useRecords(view);

  // With options
  const records = useRecords(table, {
    fields: ['Name', 'Email'],  // Only load these fields
    sorts: [{ field: 'Name', direction: 'asc' }]
  });

  return (
    <ul>
      {records.map(record => (
        <li key={record.id}>
          {record.getCellValueAsString('Name')}
        </li>
      ))}
    </ul>
  );
}
```

### useRecordById

Subscribe to a single record.

```jsx
import { useRecordById } from '@airtable/blocks/ui';

function RecordDetail({ recordId }) {
  const base = useBase();
  const table = base.getTableByName('Contacts');
  const record = useRecordById(table, recordId);

  if (!record) return <div>Loading...</div>;

  return (
    <div>
      <h1>{record.getCellValueAsString('Name')}</h1>
      <p>{record.getCellValueAsString('Email')}</p>
    </div>
  );
}
```

### useGlobalConfig

Store and retrieve extension settings.

```jsx
import { useGlobalConfig } from '@airtable/blocks/ui';

function Settings() {
  const globalConfig = useGlobalConfig();

  // Read value
  const setting = globalConfig.get('mySetting');

  // Write value
  const saveSetting = async (value) => {
    await globalConfig.setAsync('mySetting', value);
  };

  // Check if can write
  const canEdit = globalConfig.hasPermissionToSet('mySetting');

  return (
    <Input
      value={setting || ''}
      onChange={e => saveSetting(e.target.value)}
      disabled={!canEdit}
    />
  );
}
```

### useSession

Get current user info.

```jsx
import { useSession } from '@airtable/blocks/ui';

function UserInfo() {
  const session = useSession();

  return (
    <div>
      Current user: {session.currentUser.name}
      Email: {session.currentUser.email}
    </div>
  );
}
```

### useViewport

Respond to extension viewport size.

```jsx
import { useViewport } from '@airtable/blocks/ui';

function ResponsiveLayout() {
  const viewport = useViewport();

  // Check size
  if (viewport.size.width < 400) {
    return <CompactView />;
  }

  return <FullView />;
}
```

### useCursor

Track selected records/fields.

```jsx
import { useCursor } from '@airtable/blocks/ui';

function SelectionInfo() {
  const cursor = useCursor();

  // Currently selected record IDs
  const selectedRecordIds = cursor.selectedRecordIds;

  // Currently selected field IDs
  const selectedFieldIds = cursor.selectedFieldIds;

  // Active table/view
  const activeTableId = cursor.activeTableId;
  const activeViewId = cursor.activeViewId;

  return (
    <div>
      {selectedRecordIds.length} records selected
    </div>
  );
}
```

## Record Methods

### Reading Values

```jsx
// Get cell value (native type)
const name = record.getCellValue('Name');
// Returns: string, number, array, object, or null depending on field type

// Get as string (for display)
const nameStr = record.getCellValueAsString('Name');
// Always returns string

// Get record ID
const id = record.id;

// Get record name (primary field)
const name = record.name;
```

### Field Type Values

| Field Type | getCellValue returns |
|------------|---------------------|
| Text | `string` or `null` |
| Number | `number` or `null` |
| Checkbox | `boolean` or `null` |
| Single Select | `{id, name, color}` or `null` |
| Multiple Select | `[{id, name, color}, ...]` or `null` |
| User | `{id, email, name}` or `null` |
| Date | `string` (ISO) or `null` |
| Linked Records | `[{id, name}, ...]` or `null` |
| Attachments | `[{id, url, filename, ...}, ...]` or `null` |

## Table Methods

### Create Records

```jsx
// Create single record
const recordId = await table.createRecordAsync({
  'Name': 'John Doe',
  'Email': 'john@example.com'
});

// Create multiple records (max 50)
const recordIds = await table.createRecordsAsync([
  { fields: { 'Name': 'John' } },
  { fields: { 'Name': 'Jane' } }
]);
```

### Update Records

```jsx
// Update single record
await table.updateRecordAsync(recordId, {
  'Status': { name: 'Complete' }
});

// Update multiple records (max 50)
await table.updateRecordsAsync([
  { id: recordId1, fields: { 'Status': { name: 'Complete' } } },
  { id: recordId2, fields: { 'Status': { name: 'Complete' } } }
]);
```

### Delete Records

```jsx
// Delete single record
await table.deleteRecordAsync(recordId);

// Delete multiple records (max 50)
await table.deleteRecordsAsync([recordId1, recordId2]);
```

### Check Permissions

```jsx
// Check if can create
const canCreate = table.hasPermissionToCreateRecord({
  'Name': 'Test'
});

// Check if can update
const canUpdate = table.hasPermissionToUpdateRecord(record, {
  'Status': { name: 'Complete' }
});

// Check if can delete
const canDelete = table.hasPermissionToDeleteRecord(record);
```

## Field Access

```jsx
const table = base.getTableByName('Contacts');

// Get field by name
const field = table.getFieldByName('Email');

// Get field by ID
const field = table.getFieldById('fldXXXXXXXXXXXXXX');

// List all fields
const fields = table.fields;

// Field properties
console.log(field.name);        // "Email"
console.log(field.type);        // "email"
console.log(field.id);          // "fldXXX..."
console.log(field.description); // Field description or null

// For select fields, get options
if (field.type === 'singleSelect') {
  const options = field.options.choices;
  // [{id, name, color}, ...]
}
```

## View Access

```jsx
const table = base.getTableByName('Contacts');

// Get view by name
const view = table.getViewByName('Active Contacts');

// Get view by ID
const view = table.getViewById('viwXXXXXXXXXXXXXX');

// List all views
const views = table.views;

// View properties
console.log(view.name);  // "Active Contacts"
console.log(view.type);  // "grid", "calendar", "kanban", etc.
```

## Async Considerations

All write operations are async:

```jsx
async function handleSave() {
  try {
    await table.createRecordAsync(fields);
    console.log('Created successfully');
  } catch (error) {
    console.error('Create failed:', error.message);
  }
}
```

**Batch limits:** 50 records per create/update/delete call.

For more than 50:
```jsx
async function batchUpdate(records) {
  const batches = [];
  for (let i = 0; i < records.length; i += 50) {
    batches.push(records.slice(i, i + 50));
  }

  for (const batch of batches) {
    await table.updateRecordsAsync(batch);
  }
}
```
