# Airtable Interface Extensions SDK Reference

SDK hooks and methods for building Airtable Interface Extensions.

> **Important:** This documents the Interface Extensions SDK, not the legacy Blocks SDK. The two APIs differ significantly in import paths, available hooks, and entry point signature.

## Import Paths

The Interface Extensions SDK exposes exactly two import paths:

```tsx
import {
  initializeBlock,
  useBase,
  useRecords,
  useCustomProperties,
  useColorScheme,
  useSession,
  expandRecord,
  colorUtils,
} from '@airtable/blocks/interface/ui';

import { FieldType } from '@airtable/blocks/interface/models';
```

Do NOT import UI primitives (Box, Button, Input, etc.) — they are not available in Interface Extensions. Use standard HTML elements and your own styles instead.

## Entry Point

```tsx
import { initializeBlock } from '@airtable/blocks/interface/ui';

function MyExtension() {
  return <div>Hello from Interface Extensions</div>;
}

initializeBlock({ interface: () => <MyExtension /> });
```

The `{ interface: () => ... }` wrapper is required. Do NOT use the legacy form `initializeBlock(() => ...)`.

## Hooks

### useBase

Access the current base. Only tables enabled in the interface's Data panel are visible.

```tsx
import { useBase } from '@airtable/blocks/interface/ui';

function MyComponent() {
  const base = useBase();

  // Safe lookups — return null when not found
  const tableById = base.getTableByIdIfExists('tblXXXXXXXXXXXXXX');
  const tableByName = base.getTableByNameIfExists('Contacts');

  // Only tables enabled in the interface Data panel
  const tables = base.tables;

  if (!tableByName) {
    return <div>Contacts table not found or not enabled in Data panel.</div>;
  }

  return <div>Base: {base.name}</div>;
}
```

**Do NOT use** `base.getTableByName()` or `base.getTableById()` — these throw errors when the table is missing. Always use the `IfExists` variants.

### useRecords

Subscribe to records from a table. The returned array updates reactively as records are created, edited, or deleted.

```tsx
import { useBase, useRecords } from '@airtable/blocks/interface/ui';

function RecordList() {
  const base = useBase();
  const table = base.getTableByNameIfExists('Contacts');

  // Returns all records visible through the interface Data panel
  const records = useRecords(table);

  if (!table) return <div>Table not available.</div>;

  return (
    <ul>
      {records.map((record) => (
        <li key={record.id}>
          {record.getCellValueAsString(table.getFieldIfExists('Name'))}
        </li>
      ))}
    </ul>
  );
}
```

Records may change at any time — components re-render automatically when the underlying data changes.

### useSession

Get current user information.

```tsx
import { useSession } from '@airtable/blocks/interface/ui';

function UserInfo() {
  const session = useSession();
  const user = session.currentUser;

  return (
    <div>
      <p>Name: {user.name}</p>
      <p>Email: {user.email}</p>
      <p>ID: {user.id}</p>
      {user.profilePicUrl && <img src={user.profilePicUrl} alt={user.name} />}
    </div>
  );
}
```

`currentUser` provides `email`, `id`, `name`, and an optional `profilePicUrl`.

### useColorScheme

Detect whether the interface is using dark or light mode.

```tsx
import { useColorScheme } from '@airtable/blocks/interface/ui';

function ThemedCard() {
  const { colorScheme } = useColorScheme();

  const style = {
    backgroundColor: colorScheme === 'dark' ? '#1a1a2e' : '#ffffff',
    color: colorScheme === 'dark' ? '#e0e0e0' : '#333333',
    padding: '16px',
    borderRadius: '8px',
  };

  return <div style={style}>This card responds to the interface theme.</div>;
}
```

Returns `{ colorScheme: 'dark' | 'light' }`.

### useCustomProperties

Custom properties replace `useGlobalConfig` from the Blocks SDK. They let builders configure an extension differently on each interface page.

```tsx
import { useCallback } from 'react';
import {
  useBase,
  useCustomProperties,
} from '@airtable/blocks/interface/ui';
import { FieldType } from '@airtable/blocks/interface/models';

function MyExtension() {
  const base = useBase();

  // Define properties — wrap in useCallback for stable identity
  const getCustomProperties = useCallback(
    (base) => {
      const table = base.getTableByIdIfExists('tblXXXXXXXXXXXXXX');
      return [
        {
          key: 'showArchived',
          label: 'Show archived items',
          type: 'boolean' as const,
          defaultValue: false,
        },
        {
          key: 'displayMode',
          label: 'Display mode',
          type: 'enum' as const,
          possibleValues: [
            { value: 'grid', label: 'Grid' },
            { value: 'list', label: 'List' },
          ],
          defaultValue: 'grid',
        },
        {
          key: 'nameField',
          label: 'Name field',
          type: 'field' as const,
          table: table!,
          shouldFieldBeAllowed: (field: { id: string; config: any }) =>
            field.config.type === FieldType.SINGLE_LINE_TEXT,
          defaultValue: table?.getFieldIfExists('fldXXXXXXXXXXXXXX'),
        },
        {
          key: 'sourceTable',
          label: 'Source table',
          type: 'table' as const,
          defaultValue: table,
        },
      ];
    },
    [],
  );

  const { customPropertyValueByKey, errorState } =
    useCustomProperties(getCustomProperties);

  if (errorState) {
    return <div>Configuration error — check the extension settings.</div>;
  }

  const showArchived = customPropertyValueByKey.showArchived as boolean;
  const displayMode = customPropertyValueByKey.displayMode as string;

  return (
    <div>
      <p>Display mode: {displayMode}</p>
      <p>Show archived: {showArchived ? 'Yes' : 'No'}</p>
    </div>
  );
}
```

**Property types:**

| Type | Extra fields |
|------|-------------|
| `boolean` | `defaultValue: boolean` (required) |
| `string` | `defaultValue?: string` |
| `enum` | `possibleValues: Array<{value: string; label: string}>`, `defaultValue?: string` |
| `field` | `table: Table`, `shouldFieldBeAllowed?: (field) => boolean`, `defaultValue?: Field` |
| `table` | `defaultValue?: Table` |

Full type definition:

```ts
type BlockPageElementCustomProperty = { key: string; label: string } & (
  | { type: 'boolean'; defaultValue: boolean }
  | { type: 'string'; defaultValue?: string }
  | { type: 'enum'; possibleValues: Array<{ value: string; label: string }>; defaultValue?: string }
  | { type: 'field'; table: Table; shouldFieldBeAllowed?: (field: { id: FieldId; config: FieldConfig }) => boolean; defaultValue?: Field }
  | { type: 'table'; defaultValue?: Table }
);
```

**Guidelines:**
- Use custom properties for values that vary between implementations (which field to display, which table to read from).
- Hardcode stable table/field IDs directly when the value never changes.
- Wrap `getCustomProperties` in `useCallback` or define it outside the component to maintain a stable reference.

### expandRecord

Open the built-in Record Detail page for a record.

```tsx
import { useBase, useRecords, expandRecord } from '@airtable/blocks/interface/ui';

function RecordList() {
  const base = useBase();
  const table = base.getTableByNameIfExists('Contacts');
  const records = useRecords(table);

  if (!table) return null;

  const canExpand = table.hasPermissionToExpandRecords();

  return (
    <ul>
      {records.map((record) => (
        <li key={record.id}>
          {record.getCellValueAsString(table.getFieldIfExists('Name'))}
          {canExpand && (
            <button onClick={() => expandRecord(record)}>View details</button>
          )}
        </li>
      ))}
    </ul>
  );
}
```

Check `table.hasPermissionToExpandRecords()` before rendering UI that opens record details. This is the preferred approach over building custom detail popovers.

### colorUtils

Convert Airtable color names to hex values and determine text contrast.

```tsx
import { colorUtils } from '@airtable/blocks/interface/ui';

function ColorSwatch({ color }: { color: string }) {
  const hex = colorUtils.getHexForColor(color);
  const useLightText = colorUtils.shouldUseLightTextOnColor(color);

  return (
    <div
      style={{
        backgroundColor: hex,
        color: useLightText ? '#ffffff' : '#000000',
        padding: '8px 12px',
        borderRadius: '4px',
      }}
    >
      {color}
    </div>
  );
}
```

## Record Methods

### Reading Values

```tsx
// Get the field object safely
const nameField = table.getFieldIfExists('Name');

// Get cell value (native type — string, number, array, object, or null)
const name = nameField ? record.getCellValue(nameField) : null;

// Get as string (for display — always returns a string)
const nameStr = nameField ? record.getCellValueAsString(nameField) : '';

// Record ID
const id = record.id;

// Record name (primary field value)
const displayName = record.name;
```

Always check that the field exists before reading cell values.

### Field Type Values

| Field Type | `getCellValue` returns |
|------------|----------------------|
| Text fields | `string` or `null` |
| Number / Currency / Percent | `number` or `null` |
| Checkbox | `boolean` or `null` |
| Single Select | `{id, name, color}` or `null` |
| Multiple Select | `[{id, name, color}, ...]` or `null` |
| Collaborator | `{id, email, name}` or `null` |
| Date / DateTime | `string` (ISO 8601) or `null` |
| Linked Records | `[{id, name}, ...]` or `null` |
| Attachments | `[{id, url, filename, ...}, ...]` or `null` |

**Select fields:** `getCellValue` returns an object with `id`, `name`, and `color`. Render the `name` property — do not render the object directly.

```tsx
const status = record.getCellValue(statusField);
// status = { id: 'selXXX', name: 'In Progress', color: 'greenLight2' }
// Render status.name, not status itself
```

## Table Methods

### Create Records

```tsx
// Create single record
const recordId = await table.createRecordAsync({
  Name: 'John Doe',
  Email: 'john@example.com',
});

// Create multiple records (max 50 per call)
const recordIds = await table.createRecordsAsync([
  { fields: { Name: 'John' } },
  { fields: { Name: 'Jane' } },
]);
```

### Update Records

```tsx
// Update single record
await table.updateRecordAsync(recordOrRecordId, {
  Status: { name: 'Complete' },
});

// Update multiple records (max 50 per call)
await table.updateRecordsAsync([
  { id: recordId1, fields: { Status: { name: 'Complete' } } },
  { id: recordId2, fields: { Status: { name: 'Complete' } } },
]);
```

### Delete Records

```tsx
// Delete single record
await table.deleteRecordAsync(recordOrRecordId);

// Delete multiple records (max 50 per call)
await table.deleteRecordsAsync([recordId1, recordId2]);
```

### Fetch Foreign Records

For linked record fields, fetch the records from the linked table:

```tsx
const linkedRecords = await record.fetchForeignRecordsAsync(linkedField, filterString);
// Returns { records: [{ displayName, id }, ...] }
```

### Check Permissions

Always check permissions before write operations:

```tsx
// Check create permission
if (table.hasPermissionToCreateRecords([{ fields: { Name: 'Test' } }])) {
  await table.createRecordAsync({ Name: 'Test' });
}

// Check update permission
if (table.hasPermissionToUpdateRecords([{ id: record.id, fields: { Status: { name: 'Done' } } }])) {
  await table.updateRecordAsync(record, { Status: { name: 'Done' } });
}

// Check delete permission
if (table.hasPermissionToDeleteRecords([record.id])) {
  await table.deleteRecordAsync(record);
}
```

## Field Access

```tsx
import { FieldType } from '@airtable/blocks/interface/models';

const table = base.getTableByNameIfExists('Contacts');
if (!table) return null;

// Safe field lookup — returns null if field not found
const emailField = table.getFieldIfExists('Email');

if (!emailField) {
  return <div>Email field not found.</div>;
}

// Field properties
console.log(emailField.name); // "Email"
console.log(emailField.type); // FieldType enum value
console.log(emailField.id);   // "fldXXX..."

// Compare field types using the FieldType enum — NEVER use string literals
if (emailField.type === FieldType.EMAIL) {
  console.log('This is an email field');
}

// For select fields
const statusField = table.getFieldIfExists('Status');
if (statusField && statusField.type === FieldType.SINGLE_SELECT) {
  const options = statusField.options.choices;
  // [{id, name, color}, ...]
}
```

**Do NOT use** `table.getField()`, `table.getFieldByName()`, or `table.getFieldById()` — these throw errors. Always use `table.getFieldIfExists()`.

### FieldType Enum Values

```
AI_TEXT              AUTO_NUMBER          BARCODE
BUTTON               CHECKBOX             COUNT
CREATED_BY           CREATED_TIME         CURRENCY
DATE                 DATE_TIME            DURATION
EMAIL                EXTERNAL_SYNC_SOURCE FORMULA
LAST_MODIFIED_BY     LAST_MODIFIED_TIME   MULTILINE_TEXT
MULTIPLE_ATTACHMENTS MULTIPLE_COLLABORATORS MULTIPLE_LOOKUP_VALUES
MULTIPLE_RECORD_LINKS MULTIPLE_SELECTS    NUMBER
PERCENT              PHONE_NUMBER         RATING
RICH_TEXT            ROLLUP               SINGLE_COLLABORATOR
SINGLE_LINE_TEXT     SINGLE_SELECT        URL
```

Always import `FieldType` from `@airtable/blocks/interface/models` and compare using the enum (e.g., `FieldType.SINGLE_SELECT`), never against raw strings.

## Async Considerations

All write operations are async and rate-limited to 15 calls per second.

```tsx
async function handleSave(table: any, fields: Record<string, unknown>) {
  try {
    if (!table.hasPermissionToCreateRecords()) {
      console.error('No permission to create records');
      return;
    }
    await table.createRecordAsync(fields);
    console.log('Created successfully');
  } catch (error) {
    console.error('Create failed:', (error as Error).message);
  }
}
```

**Batch limits:** 50 records per `createRecordsAsync` / `updateRecordsAsync` / `deleteRecordsAsync` call.

For more than 50 records, batch the operations:

```tsx
async function batchCreate(
  table: any,
  recordFields: Array<{ fields: Record<string, unknown> }>,
) {
  for (let i = 0; i < recordFields.length; i += 50) {
    const batch = recordFields.slice(i, i + 50);
    await table.createRecordsAsync(batch);
  }
}
```

## Hooks NOT Available in Interface Extensions

The following hooks exist only in the legacy Blocks SDK and must **not** be used:

- `useViewport` — no viewport API in Interface Extensions
- `useCursor` — no cursor/selection tracking
- `useRecordById` — not available; filter from `useRecords` instead
- `useGlobalConfig` — replaced by `useCustomProperties`
