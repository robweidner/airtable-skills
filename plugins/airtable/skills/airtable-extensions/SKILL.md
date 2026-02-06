---
name: airtable-extensions
description: Build custom React-based interface extensions for Airtable. Use when creating custom apps, visualizations, or interactive tools that go beyond Interface Designer capabilities.
disable-model-invocation: true
---

# Airtable Extensions Skill

Build custom interface extensions (formerly "Blocks") for Airtable using React.

**Note:** This skill is for developers building custom React apps. For no-code interfaces, use the main `/airtable` skill and Interface Designer.

## Prerequisites

Before building extensions, ensure you have:

- [ ] Node.js 16+ installed
- [ ] npm or yarn
- [ ] An Airtable Pro/Enterprise plan (extensions require paid plan)
- [ ] Airtable developer API access

## Getting Started

### 1. Install the CLI

```bash
npm install -g @airtable/blocks-cli
```

### 2. Create New Extension

```bash
block init my-extension
cd my-extension
```

This creates a new extension project with:
```
my-extension/
├── frontend/
│   └── index.js       # Main React component
├── package.json
└── block.json         # Extension config
```

### 3. Development Server

```bash
block run
```

Opens extension in development mode. Changes hot-reload.

### 4. Deploy

```bash
block release
```

Packages and uploads to Airtable.

## Core Concepts

### The Extension Environment

Extensions run in a sandboxed iframe inside Airtable. They can:
- Read/write records in the current base
- Display custom UI
- Respond to user interactions
- Access global config

They cannot:
- Make arbitrary network requests (use Airtable proxy)
- Access the DOM outside their iframe
- Run code when the extension isn't visible

### SDK Entry Point

```jsx
import { initializeBlock } from '@airtable/blocks/ui';
import React from 'react';

function MyExtension() {
  return <div>Hello, Airtable!</div>;
}

initializeBlock(() => <MyExtension />);
```

## Reference Files

For detailed API documentation:
- [extension-api.md](reference/extension-api.md) - SDK hooks and methods
- [component-patterns.md](reference/component-patterns.md) - Common UI patterns

## Quick Patterns

### Read Records

```jsx
import { useRecords, useBase } from '@airtable/blocks/ui';

function RecordList() {
  const base = useBase();
  const table = base.getTableByName('Contacts');
  const records = useRecords(table);

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

### Update Record

```jsx
import { useBase } from '@airtable/blocks/ui';

function UpdateButton({ recordId }) {
  const base = useBase();
  const table = base.getTableByName('Contacts');

  const handleClick = async () => {
    await table.updateRecordAsync(recordId, {
      'Status': { name: 'Complete' }
    });
  };

  return <button onClick={handleClick}>Mark Complete</button>;
}
```

### User Input

```jsx
import { Input, Button, Box } from '@airtable/blocks/ui';
import { useState } from 'react';

function SearchBox({ onSearch }) {
  const [query, setQuery] = useState('');

  return (
    <Box display="flex" gap={2}>
      <Input
        value={query}
        onChange={e => setQuery(e.target.value)}
        placeholder="Search..."
      />
      <Button onClick={() => onSearch(query)}>
        Search
      </Button>
    </Box>
  );
}
```

### Settings/Config

```jsx
import { useGlobalConfig } from '@airtable/blocks/ui';

function Settings() {
  const globalConfig = useGlobalConfig();
  const apiKey = globalConfig.get('apiKey');

  const saveKey = (key) => {
    globalConfig.setAsync('apiKey', key);
  };

  return (
    <Input
      value={apiKey || ''}
      onChange={e => saveKey(e.target.value)}
      placeholder="Enter API key"
    />
  );
}
```

## When to Build an Extension

**Build an extension when:**
- Interface Designer can't do what you need
- You need complex interactivity
- External API integration required
- Custom visualizations (maps, graphs)
- Bulk operations with custom UI

**Use Interface Designer when:**
- Standard layouts work
- No coding preferred
- Quick turnaround needed
- Simple data display/entry

## Deployment Options

### 1. Base-Specific
Deploy to a single base. Only users of that base can use it.

### 2. Workspace-Wide
Share across workspace bases. Requires admin approval.

### 3. Marketplace
Public distribution. Requires Airtable review.

## Debugging

```jsx
// Console.log works - check browser devtools
console.log('Debug:', record.getCellValue('Name'));

// For async issues
try {
  await table.updateRecordAsync(recordId, fields);
} catch (error) {
  console.error('Update failed:', error);
}
```

## Common Gotchas

1. **Records are reactive** - `useRecords` updates automatically when data changes. Don't manually poll.

2. **Field access** - Use field names as strings, not field IDs (unless using `getFieldById`).

3. **Permissions** - Extension can only do what the current user can do. Check permissions before operations.

4. **Global config limits** - 50KB total. Don't store large data in global config.

5. **No server-side** - Extensions are client-only. For server operations, use Airtable Automations or external webhooks.

---

## Interface Extensions (Alpha SDK)

The newer Interface Extensions SDK has different patterns. Use `@airtable/blocks: interface-alpha` in package.json.

### Key Differences from Blocks SDK

```jsx
// Interface Extensions SDK uses different import path!
import { initializeBlock, useBase, useRecords } from '@airtable/blocks/interface/ui';

// NOT this (old Blocks SDK):
// import { useBase, useRecords } from '@airtable/blocks/ui';
```

### Interface Extensions Troubleshooting

#### "Field does not exist" Error

**Problem:** `Error: Field 'fldXXX' does not exist in table 'TableName'`

**Cause:** Interface Extensions require you to explicitly enable tables AND fields in the interface Data settings.

**Solution:**
1. Open the interface in edit mode
2. Look at the right panel → **Data** section
3. Click the gear icon next to **Table**
4. Enable all tables your extension uses
5. For each table, click the gear icon next to **Fields**
6. Enable all fields your code references (by ID or name)

**Important:** Even if a field exists in the base, your extension can't access it unless it's enabled in the interface Data settings.

#### "FORBIDDEN - Can only run in original base" Error

**Problem:** `You can only run your development block in the original base where it was created.`

**Cause:** The dev server (`block run`) is tied to the specific base ID in `.block/remote.json`.

**Solution:**
- Run the dev server while viewing the **original base** where you initialized the extension
- OR use the **published release** (click "Stop" on development panel) which works in any base

#### Best Practice: Use Field IDs, Not Names

Field names can change. Field IDs are stable. Always use IDs in production code:

```javascript
// constants.js - Store field IDs
export const FIELD_IDS = {
  EVENTS: {
    NAME: 'fldb43P9bv3dqKPVO',
    STATUS: 'fldyXuvmxMH8av5J9',
  },
};

// helpers.js - Use IDs in data access
export function mapRecordToEvent(record) {
  return {
    id: record.id,
    name: record.getCellValueAsString(FIELD_IDS.EVENTS.NAME),
    status: record.getCellValue(FIELD_IDS.EVENTS.STATUS)?.name ?? 'Upcoming',
  };
}
```

#### Publishing Releases

```bash
# Publish with a release comment
echo "Description of changes" | npx block release

# Or interactively
npx block release
# Then enter your comment when prompted
```

After publishing, users see the released version. Stop the dev server to test the published version yourself.
