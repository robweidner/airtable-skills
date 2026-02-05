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
