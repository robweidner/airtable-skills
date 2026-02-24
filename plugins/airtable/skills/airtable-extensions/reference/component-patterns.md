# Interface Extension Component Patterns

Common UI patterns for Airtable interface extensions. Interface extensions use standard HTML elements styled with Tailwind CSS -- they do NOT have access to Airtable's built-in UI components (`Box`, `Button`, `Input`, `Select`, `Text`, `Heading`, `FormField`, `TablePicker`, `FieldPicker`, `RecordPicker`, `RecordCard`, `Loader`, `Dialog`, etc.). Those are Blocks SDK only.

All imports come from `@airtable/blocks/interface/ui` or `@airtable/blocks/interface/models`. Tailwind CSS is preconfigured in the build and requires no import.

## Extension Entry Point

Every interface extension must call `initializeBlock` with an `interface` key:

```tsx
import React from 'react';
import { initializeBlock } from '@airtable/blocks/interface/ui';

function MyExtension() {
    return (
        <div className="w-full h-full p-4">
            <h1 className="text-lg font-semibold">My Extension</h1>
        </div>
    );
}

initializeBlock({ interface: () => <MyExtension /> });
```

Note the `{ interface: () => ... }` signature -- this is different from the Blocks SDK which uses `initializeBlock(() => ...)` directly.

Use `w-full h-full` on the root element so the extension fills its container.

## Layout Patterns

Use Tailwind utility classes on standard HTML elements for all layout.

### Flex Row

```tsx
function FlexRow() {
    return (
        <div className="flex items-center gap-2 p-4">
            <input
                className="flex-1 rounded border border-gray-300 px-3 py-1.5 text-sm dark:border-gray-600 dark:bg-gray-800 dark:text-gray-100"
                placeholder="Enter value..."
            />
            <button className="rounded bg-blue-600 px-4 py-1.5 text-sm font-medium text-white hover:bg-blue-700">
                Submit
            </button>
        </div>
    );
}
```

### Flex Column

```tsx
function FlexColumn() {
    return (
        <div className="flex flex-col gap-3 p-4">
            <p className="text-sm text-gray-700 dark:text-gray-300">Item 1</p>
            <p className="text-sm text-gray-700 dark:text-gray-300">Item 2</p>
            <p className="text-sm text-gray-700 dark:text-gray-300">Item 3</p>
        </div>
    );
}
```

### Card Container

```tsx
function Card({ children }: { children: React.ReactNode }) {
    return (
        <div className="rounded-lg border border-gray-200 bg-white p-4 shadow-sm dark:border-gray-700 dark:bg-gray-900">
            {children}
        </div>
    );
}
```

## Form Pattern

Use standard `<input>`, `<select>`, and `<button>` elements with Tailwind styling. Include `dark:` prefixes for dark mode support.

```tsx
import React, { useState } from 'react';

function ContactForm({ onSubmit }: { onSubmit: (data: { name: string; email: string }) => void }) {
    const [name, setName] = useState('');
    const [email, setEmail] = useState('');

    return (
        <form
            className="flex flex-col gap-4 p-4"
            onSubmit={(e) => {
                e.preventDefault();
                onSubmit({ name, email });
            }}
        >
            <div>
                <label className="mb-1 block text-sm font-medium text-gray-700 dark:text-gray-300">
                    Name
                </label>
                <input
                    type="text"
                    value={name}
                    onChange={(e) => setName(e.target.value)}
                    placeholder="Enter name"
                    className="w-full rounded-md border border-gray-300 px-3 py-2 text-sm shadow-sm
                               focus:border-blue-500 focus:outline-none focus:ring-1 focus:ring-blue-500
                               dark:border-gray-600 dark:bg-gray-800 dark:text-gray-100
                               dark:focus:border-blue-400 dark:focus:ring-blue-400"
                />
            </div>

            <div>
                <label className="mb-1 block text-sm font-medium text-gray-700 dark:text-gray-300">
                    Email
                </label>
                <input
                    type="email"
                    value={email}
                    onChange={(e) => setEmail(e.target.value)}
                    placeholder="Enter email"
                    className="w-full rounded-md border border-gray-300 px-3 py-2 text-sm shadow-sm
                               focus:border-blue-500 focus:outline-none focus:ring-1 focus:ring-blue-500
                               dark:border-gray-600 dark:bg-gray-800 dark:text-gray-100
                               dark:focus:border-blue-400 dark:focus:ring-blue-400"
                />
            </div>

            <div>
                <button
                    type="submit"
                    disabled={!name || !email}
                    className="rounded-md bg-blue-600 px-4 py-2 text-sm font-medium text-white
                               hover:bg-blue-700 disabled:cursor-not-allowed disabled:opacity-50
                               dark:bg-blue-500 dark:hover:bg-blue-600"
                >
                    Save
                </button>
            </div>
        </form>
    );
}
```

## Custom Properties Configuration Pattern

Interface extensions use `useCustomProperties` instead of `GlobalConfig` or settings panels. This lets interface editors configure the extension through a native Airtable sidebar UI (table pickers, field pickers, text inputs, etc.) without any custom settings code.

### Key Rules

1. The `properties` function must be defined **outside** the component so it has a stable identity across renders.
2. Import `FieldType` from `@airtable/blocks/interface/models` for field filtering.
3. When required properties are not yet configured, render setup instructions instead of the main UI.

```tsx
import React from 'react';
import {
    initializeBlock,
    useBase,
    useRecords,
    useCustomProperties,
} from '@airtable/blocks/interface/ui';
import { FieldType } from '@airtable/blocks/interface/models';

// Define properties OUTSIDE the component (stable identity required)
const properties = (base: any) => ({
    sourceTable: {
        type: 'table' as const,
        label: 'Source table',
        description: 'The table to read records from',
    },
    nameField: {
        type: 'field' as const,
        label: 'Name field',
        description: 'The field to use as the display name',
        parentTable: 'sourceTable',
        shouldFieldBeAllowed: (field: any) =>
            field.type === FieldType.SINGLE_LINE_TEXT ||
            field.type === FieldType.FORMULA,
    },
    statusField: {
        type: 'field' as const,
        label: 'Status field',
        description: 'A single-select field for record status',
        parentTable: 'sourceTable',
        shouldFieldBeAllowed: (field: any) =>
            field.type === FieldType.SINGLE_SELECT,
    },
    pageSize: {
        type: 'number' as const,
        label: 'Records per page',
        description: 'How many records to show at once',
    },
});

function MyExtension() {
    const base = useBase();
    const customProperties = useCustomProperties(properties);

    const { sourceTable, nameField, statusField, pageSize } = customProperties;

    // Show setup instructions when required properties are not configured
    if (!sourceTable || !nameField) {
        return (
            <div className="flex h-full w-full items-center justify-center p-8">
                <div className="text-center">
                    <h2 className="mb-2 text-lg font-semibold text-gray-800 dark:text-gray-200">
                        Configure this extension
                    </h2>
                    <p className="text-sm text-gray-500 dark:text-gray-400">
                        Click the settings icon in the extension toolbar and select a
                        source table and name field to get started.
                    </p>
                </div>
            </div>
        );
    }

    const records = useRecords(sourceTable, {
        fields: [nameField, statusField].filter(Boolean),
    });

    return (
        <div className="w-full h-full p-4">
            <h1 className="mb-3 text-lg font-semibold dark:text-gray-100">
                {sourceTable.name}
            </h1>
            <ul className="space-y-1">
                {records.slice(0, pageSize || 25).map((record) => (
                    <li key={record.id} className="text-sm dark:text-gray-300">
                        {record.getCellValueAsString(nameField)}
                        {statusField && ` - ${record.getCellValueAsString(statusField)}`}
                    </li>
                ))}
            </ul>
        </div>
    );
}

initializeBlock({ interface: () => <MyExtension /> });
```

### Available Property Types

| Type | Airtable Renders | Value Returned |
|------|-----------------|----------------|
| `'table'` | Table picker dropdown | `Table` object or `null` |
| `'field'` | Field picker (scoped to `parentTable`) | `Field` object or `null` |
| `'view'` | View picker (scoped to `parentTable`) | `View` object or `null` |
| `'number'` | Numeric input | `number` or `null` |
| `'text'` | Text input | `string` or `null` |
| `'boolean'` | Toggle switch | `boolean` |

For `'field'` properties, use `parentTable` to scope the picker to a table property and `shouldFieldBeAllowed` to filter by `FieldType`.

## Select Pill Rendering

Use `colorUtils` from `@airtable/blocks/interface/ui` to render select option pills that match Airtable's color palette:

```tsx
import { colorUtils } from '@airtable/blocks/interface/ui';

interface SelectValue {
    name: string;
    color: string;
}

function SelectPill({ value }: { value: SelectValue }) {
    const bgColor = colorUtils.getHexForColor(value.color);

    return (
        <span
            className="inline-flex items-center rounded-full px-2.5 py-0.5 text-xs font-medium"
            style={{
                backgroundColor: bgColor,
                color: colorUtils.shouldUseLightTextOnColor(value.color)
                    ? '#fff'
                    : '#000',
            }}
        >
            {value.name}
        </span>
    );
}

// Usage with a single-select cell value
function StatusBadge({ record, statusField }: { record: any; statusField: any }) {
    const value = record.getCellValue(statusField) as SelectValue | null;
    if (!value) return null;
    return <SelectPill value={value} />;
}

// Usage with a multi-select cell value
function TagList({ record, tagsField }: { record: any; tagsField: any }) {
    const values = (record.getCellValue(tagsField) as SelectValue[] | null) ?? [];

    return (
        <div className="flex flex-wrap gap-1">
            {values.map((v) => (
                <SelectPill key={v.name} value={v} />
            ))}
        </div>
    );
}
```

## Record Detail (Expand Record)

Interface extensions cannot render `RecordCard`. Instead, use `expandRecord` to open Airtable's native record detail modal:

```tsx
import { expandRecord, useBase, useRecords } from '@airtable/blocks/interface/ui';

function RecordList({ table }: { table: any }) {
    const records = useRecords(table);
    const canExpand = table.hasPermissionToExpandRecords();

    return (
        <ul className="divide-y divide-gray-200 dark:divide-gray-700">
            {records.map((record) => (
                <li
                    key={record.id}
                    className="flex items-center justify-between px-4 py-3"
                >
                    <span className="text-sm dark:text-gray-200">
                        {record.name}
                    </span>
                    {canExpand && (
                        <button
                            onClick={() => expandRecord(record)}
                            className="rounded px-2 py-1 text-xs text-blue-600 hover:bg-blue-50
                                       dark:text-blue-400 dark:hover:bg-gray-800"
                        >
                            View details
                        </button>
                    )}
                </li>
            ))}
        </ul>
    );
}
```

Always check `table.hasPermissionToExpandRecords()` before rendering the expand button. Not all interface configurations allow record expansion.

## Loading State

Use a Tailwind CSS spinner or skeleton instead of the Blocks SDK `<Loader />`:

### Spinner

```tsx
function Spinner() {
    return (
        <div className="flex h-full w-full items-center justify-center">
            <div className="h-8 w-8 animate-spin rounded-full border-4 border-gray-200 border-t-blue-600 dark:border-gray-700 dark:border-t-blue-400" />
        </div>
    );
}
```

### Skeleton Rows

```tsx
function SkeletonRows({ count = 5 }: { count?: number }) {
    return (
        <div className="w-full space-y-3 p-4">
            {Array.from({ length: count }).map((_, i) => (
                <div key={i} className="flex animate-pulse gap-3">
                    <div className="h-4 w-1/4 rounded bg-gray-200 dark:bg-gray-700" />
                    <div className="h-4 w-1/2 rounded bg-gray-200 dark:bg-gray-700" />
                    <div className="h-4 w-1/6 rounded bg-gray-200 dark:bg-gray-700" />
                </div>
            ))}
        </div>
    );
}
```

### Conditional Loading

```tsx
import { useBase, useRecords } from '@airtable/blocks/interface/ui';

function DataLoader() {
    const base = useBase();
    const table = base.getTableByIdIfExists('tblXXXXXXXXXX');
    const records = table ? useRecords(table) : [];

    if (!records || records.length === 0) {
        return <Spinner />;
    }

    return <DataContent records={records} />;
}
```

## Data Table Pattern

Use standard HTML `<table>` or divs with Tailwind for tabular data. Read cell values with `record.getCellValueAsString(field)`.

```tsx
import { useBase, useRecords } from '@airtable/blocks/interface/ui';
import type { Field, Record } from '@airtable/blocks/interface/models';

function DataTable({ records, fields }: { records: Record[]; fields: Field[] }) {
    return (
        <div className="w-full overflow-x-auto">
            <table className="w-full text-left text-sm">
                <thead>
                    <tr className="border-b border-gray-200 bg-gray-50 dark:border-gray-700 dark:bg-gray-800">
                        {fields.map((field) => (
                            <th
                                key={field.id}
                                className="px-4 py-2 text-xs font-semibold uppercase tracking-wide text-gray-500 dark:text-gray-400"
                            >
                                {field.name}
                            </th>
                        ))}
                    </tr>
                </thead>
                <tbody className="divide-y divide-gray-200 dark:divide-gray-700">
                    {records.map((record) => (
                        <tr
                            key={record.id}
                            className="hover:bg-gray-50 dark:hover:bg-gray-800/50"
                        >
                            {fields.map((field) => (
                                <td
                                    key={field.id}
                                    className="px-4 py-2 text-gray-700 dark:text-gray-300"
                                >
                                    {record.getCellValueAsString(field)}
                                </td>
                            ))}
                        </tr>
                    ))}
                </tbody>
            </table>
        </div>
    );
}
```

## Search / Filter Pattern

Standard input with filter logic and Tailwind styling:

```tsx
import React, { useState, useMemo } from 'react';
import { useBase, useRecords } from '@airtable/blocks/interface/ui';
import type { Field } from '@airtable/blocks/interface/models';

function SearchableList({ table, nameField }: { table: any; nameField: Field }) {
    const [search, setSearch] = useState('');
    const records = useRecords(table);

    const filtered = useMemo(
        () =>
            records.filter((record) =>
                record
                    .getCellValueAsString(nameField)
                    .toLowerCase()
                    .includes(search.toLowerCase()),
            ),
        [records, search, nameField],
    );

    return (
        <div className="flex h-full w-full flex-col">
            {/* Search bar */}
            <div className="border-b border-gray-200 p-3 dark:border-gray-700">
                <input
                    type="text"
                    value={search}
                    onChange={(e) => setSearch(e.target.value)}
                    placeholder="Search by name..."
                    className="w-full rounded-md border border-gray-300 px-3 py-2 text-sm
                               focus:border-blue-500 focus:outline-none focus:ring-1 focus:ring-blue-500
                               dark:border-gray-600 dark:bg-gray-800 dark:text-gray-100"
                />
            </div>

            {/* Results */}
            <div className="flex-1 overflow-y-auto">
                {filtered.length === 0 ? (
                    <p className="p-4 text-center text-sm text-gray-400 dark:text-gray-500">
                        No results found
                    </p>
                ) : (
                    <ul className="divide-y divide-gray-100 dark:divide-gray-800">
                        {filtered.map((record) => (
                            <li
                                key={record.id}
                                className="px-4 py-2.5 text-sm text-gray-700 dark:text-gray-300"
                            >
                                {record.getCellValueAsString(nameField)}
                            </li>
                        ))}
                    </ul>
                )}
            </div>
        </div>
    );
}
```

## Error Handling Pattern

Standard try/catch with a Tailwind-styled error banner:

```tsx
import React, { useState } from 'react';

function SafeComponent({ table }: { table: any }) {
    const [error, setError] = useState<string | null>(null);
    const [saving, setSaving] = useState(false);

    const handleAction = async () => {
        try {
            setError(null);
            setSaving(true);
            await table.createRecordAsync({ Name: 'New Record' });
        } catch (err: any) {
            setError(err.message ?? 'An unexpected error occurred');
        } finally {
            setSaving(false);
        }
    };

    return (
        <div className="p-4">
            {error && (
                <div className="mb-3 rounded-md border border-red-200 bg-red-50 px-4 py-3 dark:border-red-800 dark:bg-red-900/30">
                    <p className="text-sm text-red-700 dark:text-red-300">
                        Error: {error}
                    </p>
                </div>
            )}

            <button
                onClick={handleAction}
                disabled={saving}
                className="rounded-md bg-blue-600 px-4 py-2 text-sm font-medium text-white
                           hover:bg-blue-700 disabled:cursor-not-allowed disabled:opacity-50
                           dark:bg-blue-500 dark:hover:bg-blue-600"
            >
                {saving ? 'Saving...' : 'Create Record'}
            </button>
        </div>
    );
}
```

## Dark Mode Support

Interface extensions support dark mode through two mechanisms:

### 1. Tailwind `dark:` Prefix (CSS-level)

Tailwind's `dark:` variant responds to the user's `prefers-color-scheme` media query automatically:

```tsx
function DarkModeCard() {
    return (
        <div className="rounded-lg border border-gray-200 bg-white p-4 text-gray-900
                        dark:border-gray-700 dark:bg-gray-900 dark:text-gray-100">
            <h2 className="text-lg font-semibold">Title</h2>
            <p className="mt-1 text-sm text-gray-500 dark:text-gray-400">
                This card adapts to light and dark mode automatically.
            </p>
        </div>
    );
}
```

### 2. `useColorScheme` Hook (JS-level)

When you need to branch logic (not just styling) based on the color scheme:

```tsx
import { useColorScheme } from '@airtable/blocks/interface/ui';

function ChartWrapper() {
    const colorScheme = useColorScheme();
    const isDark = colorScheme === 'dark';

    // Use for dynamic values like chart colors, inline styles, or third-party libs
    const chartColors = isDark
        ? { background: '#1f2937', text: '#e5e7eb', grid: '#374151' }
        : { background: '#ffffff', text: '#111827', grid: '#e5e7eb' };

    return (
        <div
            className="h-64 w-full rounded-lg"
            style={{ backgroundColor: chartColors.background }}
        >
            {/* Pass chartColors to a chart library, canvas rendering, etc. */}
        </div>
    );
}
```

### Common Dark Mode Pairs

| Element | Light | Dark |
|---------|-------|------|
| Page background | `bg-white` | `dark:bg-gray-900` |
| Card background | `bg-white` | `dark:bg-gray-800` |
| Primary text | `text-gray-900` | `dark:text-gray-100` |
| Secondary text | `text-gray-500` | `dark:text-gray-400` |
| Border | `border-gray-200` | `dark:border-gray-700` |
| Input background | (default) | `dark:bg-gray-800` |
| Hover state | `hover:bg-gray-50` | `dark:hover:bg-gray-800` |

## Permission Check Pattern

Always check permissions before rendering action buttons. This avoids showing controls that will fail when clicked.

```tsx
import React, { useState } from 'react';
import { useBase, useRecords, expandRecord } from '@airtable/blocks/interface/ui';

function RecordManager({ table }: { table: any }) {
    const records = useRecords(table);
    const [selectedId, setSelectedId] = useState<string | null>(null);

    const canCreate = table.hasPermissionToCreateRecords();
    const canExpand = table.hasPermissionToExpandRecords();

    const handleCreate = async () => {
        if (!canCreate) return;
        try {
            await table.createRecordAsync({});
        } catch (err: any) {
            console.error('Create failed:', err.message);
        }
    };

    const handleDelete = async (recordId: string) => {
        // Check delete permission for a specific record
        const record = records.find((r) => r.id === recordId);
        if (!record || !table.hasPermissionToDeleteRecord(record)) return;
        try {
            await table.deleteRecordAsync(recordId);
        } catch (err: any) {
            console.error('Delete failed:', err.message);
        }
    };

    const handleUpdate = async (recordId: string, fields: Record<string, any>) => {
        const record = records.find((r) => r.id === recordId);
        if (!record || !table.hasPermissionToUpdateRecord(record, fields)) return;
        try {
            await table.updateRecordAsync(recordId, fields);
        } catch (err: any) {
            console.error('Update failed:', err.message);
        }
    };

    return (
        <div className="flex h-full w-full flex-col p-4">
            {/* Toolbar */}
            <div className="mb-3 flex items-center gap-2">
                {canCreate && (
                    <button
                        onClick={handleCreate}
                        className="rounded-md bg-green-600 px-3 py-1.5 text-sm font-medium text-white
                                   hover:bg-green-700 dark:bg-green-500 dark:hover:bg-green-600"
                    >
                        + New Record
                    </button>
                )}
            </div>

            {/* Record list */}
            <ul className="flex-1 divide-y divide-gray-200 overflow-y-auto dark:divide-gray-700">
                {records.map((record) => (
                    <li
                        key={record.id}
                        className="flex items-center justify-between px-2 py-2.5"
                    >
                        <span className="text-sm dark:text-gray-200">
                            {record.name}
                        </span>
                        <div className="flex items-center gap-2">
                            {canExpand && (
                                <button
                                    onClick={() => expandRecord(record)}
                                    className="rounded px-2 py-1 text-xs text-blue-600 hover:bg-blue-50
                                               dark:text-blue-400 dark:hover:bg-gray-800"
                                >
                                    View
                                </button>
                            )}
                            {table.hasPermissionToDeleteRecord(record) && (
                                <button
                                    onClick={() => handleDelete(record.id)}
                                    className="rounded px-2 py-1 text-xs text-red-600 hover:bg-red-50
                                               dark:text-red-400 dark:hover:bg-gray-800"
                                >
                                    Delete
                                </button>
                            )}
                        </div>
                    </li>
                ))}
            </ul>
        </div>
    );
}
```

### Permission Methods Reference

| Method | Checks |
|--------|--------|
| `table.hasPermissionToCreateRecords()` | Can create any new record |
| `table.hasPermissionToCreateRecord(fields)` | Can create a record with specific field values |
| `table.hasPermissionToUpdateRecord(record, fields)` | Can update specific fields on a record |
| `table.hasPermissionToUpdateRecords()` | Can update any record |
| `table.hasPermissionToDeleteRecord(record)` | Can delete a specific record |
| `table.hasPermissionToDeleteRecords()` | Can delete any record |
| `table.hasPermissionToExpandRecords()` | Can open the record detail modal |
