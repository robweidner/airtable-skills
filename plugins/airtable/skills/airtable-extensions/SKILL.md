---
name: airtable-extensions
description: Build custom React-based interface extensions for Airtable. Use when creating custom apps, visualizations, or interactive tools that go beyond Interface Designer capabilities.
disable-model-invocation: true
---

# Airtable Extensions Skill

Build custom interface extensions (formerly "Blocks") for Airtable using React and TypeScript.

**Note:** This skill is for developers building custom React apps. For no-code interfaces, use the main `/airtable` skill and Interface Designer.

> **This skill covers Interface Extensions**, which use the `@airtable/blocks/interface/ui` SDK. This is NOT the legacy Blocks SDK (`@airtable/blocks/ui`). The two have different import paths, different hooks, different entry point signatures, and interface extensions have no access to Airtable's built-in UI components (`Box`, `Button`, `Input`, `TablePicker`, etc.). Use plain HTML elements with Tailwind CSS for all UI.

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

```tsx
import { initializeBlock } from '@airtable/blocks/interface/ui';
import React from 'react';

function MyExtension() {
  return <div>Hello, Airtable!</div>;
}

initializeBlock({ interface: () => <MyExtension /> });
```

## Reference Files

For detailed API documentation:
- [extension-api.md](reference/extension-api.md) - SDK hooks and methods
- [component-patterns.md](reference/component-patterns.md) - Common UI patterns
- [interface-extensions-workflow.md](reference/interface-extensions-workflow.md) - Interface extensions development workflow, troubleshooting, and checklists

## Quick Patterns

### Read Records

```tsx
import { useRecords, useBase } from '@airtable/blocks/interface/ui';

function RecordList() {
  const base = useBase();
  const table = base.getTableByIdIfExists('tblXXXXXXXXXX');
  const records = table ? useRecords(table) : [];
  const nameField = table?.getFieldIfExists('fldXXXXXXXXXX');

  return (
    <ul>
      {records.map(record => (
        <li key={record.id}>
          {nameField ? record.getCellValueAsString(nameField) : record.name}
        </li>
      ))}
    </ul>
  );
}
```

### Update Record

```tsx
import { useBase } from '@airtable/blocks/interface/ui';

function UpdateButton({ recordId }: { recordId: string }) {
  const base = useBase();
  const table = base.getTableByIdIfExists('tblXXXXXXXXXX');

  const handleClick = async () => {
    if (!table) return;
    if (table.hasPermissionToUpdateRecord(recordId, { 'fldXXXXXXXXXX': undefined })) {
      await table.updateRecordAsync(recordId, {
        'fldXXXXXXXXXX': { name: 'Complete' },
      });
    }
  };

  return <button onClick={handleClick}>Mark Complete</button>;
}
```

### User Input

Interface Extensions do not include built-in UI components like `Input`, `Button`, or `Box`. Use plain HTML elements with Tailwind CSS for styling:

```tsx
import { useState } from 'react';

function SearchBox({ onSearch }: { onSearch: (q: string) => void }) {
  const [query, setQuery] = useState('');

  return (
    <div className="flex gap-2">
      <input
        className="border rounded px-3 py-1.5 flex-1 dark:bg-gray-800 dark:text-white dark:border-gray-600"
        value={query}
        onChange={e => setQuery(e.target.value)}
        placeholder="Search..."
      />
      <button
        className="px-4 py-1.5 bg-blue-500 text-white rounded hover:bg-blue-600"
        onClick={() => onSearch(query)}
      >
        Search
      </button>
    </div>
  );
}
```

### Settings/Config

Interface Extensions use `useCustomProperties` instead of `useGlobalConfig`. Custom properties let interface builders configure your extension without editing code:

```tsx
import { useCustomProperties, useBase } from '@airtable/blocks/interface/ui';
import { FieldType } from '@airtable/blocks/interface/models';

function getCustomProperties(base: Base) {
  const table = base.tables[0];
  return [
    {
      key: 'statusField',
      label: 'Status Field',
      type: 'field' as const,
      table,
      shouldFieldBeAllowed: (field: {id: string; config: {type: string}}) =>
        field.config.type === FieldType.SINGLE_SELECT,
      defaultValue: table?.fields.find(f => f.type === FieldType.SINGLE_SELECT),
    },
  ];
}

function MyExtension() {
  const { customPropertyValueByKey, errorState } = useCustomProperties(getCustomProperties);
  const statusField = customPropertyValueByKey.statusField;
  // Use statusField...
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

2. **Field access** - Use field IDs, not field names. Field names can change; IDs are stable. Use `table.getFieldIfExists(fieldId)` and always check for null.

3. **Permissions** - Extension can only do what the current user can do. Check permissions before operations.

4. **Custom properties** - Use `useCustomProperties` for builder-configurable settings, not `useGlobalConfig`.

5. **No server-side** - The extension runtime is client-only. For server operations, use Airtable Automations or external webhooks.

---

## Interface Extensions (Open Beta SDK)

Interface Extensions let you embed custom React apps directly into Airtable interfaces. Unlike regular Blocks SDK extensions (which live in a base-level sidebar), interface extensions are embedded as components within Interface Designer pages.

**Plan requirement:** Team, Business, or Enterprise.

### Key Differences from Blocks SDK

| Feature | Blocks SDK | Interface Extensions SDK |
|---------|-----------|------------------------|
| Import path | `@airtable/blocks/ui` | `@airtable/blocks/interface/ui` |
| Where it runs | Base sidebar | Inside an interface page |
| Data access | All tables/fields in base | Only tables/fields enabled in interface Data panel |
| Multiple tables | Full base access | No native multi-table; use Web API as workaround |
| Package.json | `@airtable/blocks` | `@airtable/blocks: interface-alpha` |

```jsx
// Interface Extensions SDK uses different import path!
import { initializeBlock, useBase, useRecords } from '@airtable/blocks/interface/ui';

// NOT this (old Blocks SDK):
// import { useBase, useRecords } from '@airtable/blocks/ui';
```

### Interface Extensions Data Source Configuration

**This is the #1 cause of "missing table" and "missing field" errors.** Interface extensions can ONLY access tables and fields that are explicitly added to the interface's Data panel. This is different from the Blocks SDK, where your code can see the entire base.

#### Pre-Flight Checklist (Run BEFORE coding)

Before writing any code or running `block run`, verify this configuration:

- [ ] Open the interface in **edit mode** (click "Edit" at top)
- [ ] Select the page containing your custom extension
- [ ] In the **right panel**, find the **Data** section
- [ ] Click the **gear icon** next to each table name
- [ ] **Enable every table** your extension code references (via `getTableByName` or `getTableById`)
- [ ] For **each enabled table**, click the gear icon next to **Fields**
- [ ] **Enable every field** your code reads or writes (by name or ID)
- [ ] If using linked records, enable the linked table AND its fields too
- [ ] Save the interface configuration

**The golden rule:** If your code touches it, the interface must expose it.

#### Diagnostic Pattern

Add this to your extension to see exactly what data is available:

```tsx
function DataDiagnostic() {
  const base = useBase();

  return (
    <div className="p-4">
      <h3 className="text-sm font-bold mb-2">Available Data Sources</h3>
      {base.tables.map(table => (
        <div key={table.id} className="mt-2">
          <p className="font-bold">{table.name} ({table.id})</p>
          <div className="ml-4">
            {table.fields.map(field => (
              <p key={field.id} className="text-xs text-gray-500">
                {field.name} ({field.id}) — {field.type}
              </p>
            ))}
          </div>
        </div>
      ))}
    </div>
  );
}
```

If a table or field you expect is missing from this output, it hasn't been enabled in the interface Data panel.

### Interface Extensions Troubleshooting

#### "Field does not exist" / "Table not found" Error

**Problem:** `Error: Field 'fldXXX' does not exist in table 'TableName'` or your code can't find a table that definitely exists in the base.

**Cause:** Interface Extensions can only see tables and fields that are explicitly enabled in the interface's Data panel. This is the most common development gotcha.

**Solution:**
1. Open the interface in edit mode
2. Look at the right panel → **Data** section
3. Click the gear icon next to **Table**
4. Enable all tables your extension uses
5. For each table, click the gear icon next to **Fields**
6. Enable all fields your code references (by ID or name)
7. **Save the interface** — changes don't take effect until saved

**Important:** Even if a field exists in the base, your extension can't access it unless it's enabled in the interface Data settings. This applies to ALL field access — reading, writing, and even checking if a field exists.

**Pro tip:** When adding new fields to your code, always add them to the interface Data panel FIRST, then update your code. Otherwise `block run` will crash with a confusing error.

#### "FORBIDDEN - Can only run in original base" Error

**Problem:** `You can only run your development block in the original base where it was created.`

**Cause:** The dev server (`block run`) is tied to the specific base ID stored in `.block/remote.json`.

**Solution:**
- Run the dev server while viewing the **original base** where you initialized the extension
- OR use the **published release** (click "Stop" on development panel) which works in any base
- Check `.block/remote.json` to verify the base ID matches where you're running it

#### Multiple Tables Limitation

**Problem:** Interface Extensions SDK doesn't natively support accessing multiple tables.

**Workaround:** Use the Airtable Web API to read from other tables:

```tsx
// Use fetch with the Airtable REST API for secondary tables.
// Store credentials (e.g. API token) via custom properties (string type),
// not hardcoded in source code.
const fetchLinkedRecords = async (tableId: string, apiToken: string) => {
  const response = await fetch(
    `https://api.airtable.com/v0/${baseId}/${tableId}`,
    { headers: { Authorization: `Bearer ${apiToken}` } }
  );
  return response.json();
};
```

### Development Server (`block run`) Guide

The `block run` command starts a local dev server that hot-reloads your extension code inside Airtable. It's powerful but can be fragile.

#### Starting Development

```bash
# Start the dev server
block run

# The server runs on localhost (usually port 9000)
# Airtable loads your code from this local server
```

After running `block run`:
1. Go to your Airtable interface in the browser
2. Find your extension → click "Edit extension"
3. Enter the localhost URL shown in your terminal
4. Your extension loads from your local machine

#### When `block run` Crashes

The dev server can crash during hot-reload, especially with:
- Syntax errors in JSX
- Import errors (missing modules)
- Large file saves that trigger multiple rapid reloads
- Node.js memory issues on complex extensions

**Recovery steps:**
1. `Ctrl+C` to kill the crashed server (if it's hung, `Ctrl+C` twice)
2. Check terminal output for the actual error
3. Fix the code error that caused the crash
4. Run `block run` again
5. **Refresh the Airtable browser tab** — the extension iframe may have a stale connection
6. If the extension shows "Connection error", the server isn't running — restart it

**Prevention tips:**
- Save files one at a time (avoid "save all" with many changed files)
- Keep the terminal visible to catch errors early
- If the server crashes repeatedly on startup, delete `node_modules` and reinstall:
  ```bash
  rm -rf node_modules && npm install && block run
  ```

#### Chrome/Browser Compatibility

Chrome's strict security can block the local dev server:

**Problem:** "Connection error. Please check if your local block is running" — but the server IS running.

**Cause:** Chrome's CORS policy blocks localhost connections from Airtable's domain.

**Solutions (try in order):**
1. Check `chrome://flags` → search for "localhost" → enable "Allow invalid certificates for resources loaded from localhost"
2. If that flag is hidden/expired, enable "Temporarily unexpire M130 flags" first, relaunch Chrome, then set the localhost flag
3. Try Firefox as a workaround (more permissive with localhost)
4. Last resort: launch Chrome with `--disable-web-security` using a separate profile:
   ```bash
   # macOS
   open -na "Google Chrome" --args --disable-web-security --user-data-dir=/tmp/chrome-dev
   ```
   **Only use this for development.** Never browse normally with web security disabled.

#### Alternative: Omni-Generated Extensions

For quick prototyping, you can ask Airtable's Omni AI to generate a basic custom interface extension, then click "Edit Source" to modify the code directly — bypassing the CLI setup entirely. Good for:
- Quick experiments
- Learning the API by reading generated code
- Extensions that don't need a local dev workflow

### Best Practice: Use Field IDs, Not Names

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

### Publishing Releases

```bash
# Publish with a release comment
echo "Description of changes" | npx block release

# Or interactively
npx block release
# Then enter your comment when prompted
```

After publishing, users see the released version. Stop the dev server to test the published version yourself.

#### Auto-Release on Git Push (Claude Code Hook)

If you version your extension in git, you can set up a Claude Code hook to automatically run `block release` every time you push — so your Airtable extension stays in sync with your repo.

**1. Create the hook script** at `~/.claude/hooks/post-push-block-release.sh`:

```bash
#!/bin/bash
# Post-Push Block Release Hook
# After a successful git push in an Airtable extension repo,
# automatically run `npx block release` to publish to Airtable.

# Set this to your extension repo path
EXTENSION_REPO="$HOME/path/to/your-extension"

INPUT=$(cat)
COMMAND=$(echo "$INPUT" | grep -o '"command"[[:space:]]*:[[:space:]]*"[^"]*"' | sed 's/"command"[[:space:]]*:[[:space:]]*"//' | sed 's/"$//')

# Only trigger on git push commands
if [[ "$COMMAND" != *"git push"* ]]; then
  exit 0
fi

# Check if the push was in the extension repo
if [[ "$COMMAND" != *"$EXTENSION_REPO"* ]]; then
  TOOL_CWD=$(echo "$INPUT" | grep -o '"cwd"[[:space:]]*:[[:space:]]*"[^"]*"' | sed 's/"cwd"[[:space:]]*:[[:space:]]*"//' | sed 's/"$//')
  if [[ "$TOOL_CWD" != "$EXTENSION_REPO"* ]]; then
    exit 0
  fi
fi

# Skip release if the push failed
OUTPUT=$(echo "$INPUT" | grep -o '"output"[[:space:]]*:[[:space:]]*"[^"]*"' | head -1)
if echo "$OUTPUT" | grep -qiE '(rejected|failed|fatal|error|denied)'; then
  echo "Git push failed. Skipping block release." >&2
  exit 0
fi

echo "Running block release after git push..." >&2
cd "$EXTENSION_REPO"
RELEASE_OUTPUT=$(npx block release 2>&1)
if [ $? -eq 0 ]; then
  echo "Block release successful! Extension published to Airtable."
else
  echo "Block release failed. Run 'npx block release' manually."
  echo "$RELEASE_OUTPUT" >&2
fi
exit 0
```

**2. Make it executable:**

```bash
chmod +x ~/.claude/hooks/post-push-block-release.sh
```

**3. Add the hook** to `~/.claude/settings.json`:

```json
{
  "hooks": {
    "PostToolUse": [
      {
        "matcher": "^Bash$",
        "hooks": [
          {
            "type": "command",
            "command": "~/.claude/hooks/post-push-block-release.sh",
            "timeout": 60
          }
        ]
      }
    ]
  }
}
```

Now every `git push` in your extension repo will automatically publish to Airtable — one workflow, both destinations.
