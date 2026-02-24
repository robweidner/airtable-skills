# Interface Extensions Development Workflow

Complete guide to building, debugging, and deploying custom interface extensions in Airtable.

## Overview

Interface extensions are custom React apps embedded in Airtable Interface Designer pages. They use a variant of the Blocks SDK with a different import path and a more restricted data access model.

**Key difference from Blocks SDK:** Interface extensions can only access tables and fields that are explicitly enabled in the interface's Data panel. Regular blocks see the entire base.

## Development Lifecycle

### Phase 1: Setup

```bash
# Install CLI globally
npm install -g @airtable/blocks-cli

# Create extension project
block init my-interface-extension
cd my-interface-extension
```

Update `package.json` to use the interface SDK:
```json
{
  "dependencies": {
    "@airtable/blocks": "interface-alpha"
  }
}
```

Update imports in your code:
```jsx
// Use interface path — NOT the regular blocks path
import { initializeBlock, useBase, useRecords } from '@airtable/blocks/interface/ui';
```

### Phase 2: Configure Interface Data Sources

**Do this BEFORE writing any data-access code.**

1. Open your Airtable interface in **edit mode**
2. Add or select the page where your extension will live
3. Add a **Custom Extension** element to the page
4. In the **right panel** → **Data** section:
   - Enable every table your code will reference
   - For each table, enable every field your code will read or write
   - If using linked records, enable the linked table AND its fields
5. **Save the interface**

### Phase 3: Local Development

```bash
block run
```

Then in Airtable:
1. Navigate to your interface page
2. Click the custom extension element → "Edit extension"
3. Enter the localhost URL from your terminal (usually `https://localhost:9000`)
4. Extension loads from your local machine with hot-reload

### Phase 4: Testing & Debugging

Use browser DevTools (F12) to:
- Check console for errors
- Monitor network requests
- Debug React component state

Add the diagnostic component (see below) to verify data access.

### Phase 5: Release

```bash
# Publish to Airtable
echo "v1.0 - Initial release" | npx block release
```

After release, stop your dev server. Users see the published version.

---

## Data Source Configuration (Critical)

This is the #1 source of bugs in interface extension development. The interface's Data panel acts as an allowlist — your extension code is sandboxed to only what's explicitly enabled.

### What Gets Blocked

| Code pattern | Requires in Data panel |
|-------------|----------------------|
| `base.getTableByName('Tasks')` | "Tasks" table must be enabled |
| `base.getTableById('tblXXX')` | Table with that ID must be enabled |
| `record.getCellValue('Status')` | "Status" field must be enabled on that table |
| `record.getCellValue('fldXXX')` | Field with that ID must be enabled |
| `table.fields` | Only returns enabled fields |
| `base.tables` | Only returns enabled tables |

### Diagnostic Component

Drop this into your extension to see exactly what's available:

```tsx
import React from 'react';
import { useBase } from '@airtable/blocks/interface/ui';

function DataDiagnostic() {
  const base = useBase();

  return (
    <div className="p-4">
      <h3 className="text-sm font-bold mb-1">Data Sources Available to Extension</h3>
      <p className="text-xs text-gray-400 mb-3">
        Tables: {base.tables.length} | If something is missing, enable it in the interface Data panel
      </p>
      {base.tables.map(table => (
        <div key={table.id} className="mt-2 p-2 border rounded">
          <p className="font-bold">{table.name}</p>
          <p className="text-xs text-gray-400">{table.id}</p>
          <div className="ml-4 mt-1">
            {table.fields.map(field => (
              <p key={field.id} className="text-xs">
                {field.name} ({field.type}) — {field.id}
              </p>
            ))}
          </div>
        </div>
      ))}
    </div>
  );
}

export default DataDiagnostic;
```

### Adding New Fields to an Existing Extension

When your code needs a new field:

1. **First:** Add the field to the interface Data panel and save
2. **Then:** Update your code to reference the field
3. **Then:** Refresh the browser tab

If you do this in reverse order (code first, data panel second), `block run` will throw errors and may crash.

---

## `block run` Server Management

### Normal Startup

```bash
$ block run
✔ Extension running at https://localhost:9000
```

The server watches your source files and hot-reloads on save.

### Common Crash Scenarios

#### 1. Syntax Error in JSX
**Symptom:** Server stops, terminal shows parse error
**Fix:** Correct the syntax, then re-run `block run`

#### 2. Missing Module Import
**Symptom:** `Module not found: Error: Can't resolve 'some-package'`
**Fix:** `npm install some-package` then re-run `block run`

#### 3. Rapid Multiple Saves
**Symptom:** Server hangs or crashes during hot-reload
**Fix:** `Ctrl+C` (twice if needed), then `block run` again

#### 4. Stale Node Modules
**Symptom:** Random errors after updating `@airtable/blocks` version
**Fix:**
```bash
rm -rf node_modules package-lock.json
npm install
block run
```

#### 5. Port Already in Use
**Symptom:** `Error: listen EADDRINUSE :::9000`
**Fix:**
```bash
# Find and kill the old process
lsof -ti:9000 | xargs kill -9
block run
```

### After Restarting `block run`

The Airtable browser tab may have a stale WebSocket connection. Always:
1. Restart `block run`
2. **Refresh the Airtable browser tab** (Cmd+R / F5)
3. If the extension still shows "Connection error", hard-refresh (Cmd+Shift+R)

---

## Browser Compatibility

### Chrome CORS Issues

Chrome enforces strict CORS policies that can block the localhost dev server.

**Symptom:** "Connection error. Please check if your local block is running" — but the server IS running.

**Fix (try in order):**

1. **Chrome flags** — go to `chrome://flags`, search "localhost":
   - Enable "Allow invalid certificates for resources loaded from localhost"
   - If this flag isn't visible, first enable "Temporarily unexpire M130 flags", relaunch, then set the localhost flag

2. **Firefox** — use Firefox for extension development (more permissive with localhost CORS)

3. **Chrome with relaxed security** (dev only):
   ```bash
   # macOS
   open -na "Google Chrome" --args --disable-web-security --user-data-dir=/tmp/chrome-dev

   # Linux
   google-chrome --disable-web-security --user-data-dir=/tmp/chrome-dev
   ```
   Never browse normally with this flag. Use a separate profile.

### Safari
Not officially supported for extension development. Use Chrome or Firefox.

---

## Multiple Tables Workaround

The Interface Extensions SDK does not natively support multiple tables. If your extension needs data from tables not in the Data panel, use the Airtable REST API:

```jsx
const AIRTABLE_API = 'https://api.airtable.com/v0';

async function fetchFromTable(baseId, tableIdOrName, options = {}) {
  const url = new URL(`${AIRTABLE_API}/${baseId}/${tableIdOrName}`);
  if (options.filterByFormula) {
    url.searchParams.set('filterByFormula', options.filterByFormula);
  }
  if (options.fields) {
    options.fields.forEach(f => url.searchParams.append('fields[]', f));
  }

  const response = await fetch(url.toString(), {
    headers: { Authorization: `Bearer ${apiToken}` }
  });

  if (!response.ok) throw new Error(`API error: ${response.status}`);
  return response.json();
}
```

**Note:** Store the API token via `useCustomProperties` (define a string-type custom property in your extension config) rather than `globalConfig`, which is not available in interface extensions. The SDK's built-in hooks only access interface-enabled data.

---

## Omni as Alternative Entry Point

Airtable's Omni AI can generate basic interface extensions without CLI setup:

1. In an interface, ask Omni to "create a custom interface extension that [does X]"
2. Omni generates the code and adds it to your interface
3. Click "Edit Source" (below "Share Interface") to modify the generated code
4. Edit directly in the browser — no `block run` needed

**Good for:** Quick experiments, learning the API, simple one-off tools
**Not good for:** Complex extensions, version control, team development

---

## Error Reference

| Error Message | Cause | Fix |
|--------------|-------|-----|
| `Field 'fldXXX' does not exist in table` | Field not enabled in interface Data panel | Enable field in Data panel, save interface |
| `Table 'TableName' not found` | Table not enabled in interface Data panel | Enable table in Data panel, save interface |
| `FORBIDDEN - Can only run in original base` | `block run` tied to different base | Check `.block/remote.json` base ID matches |
| `Connection error. Please check if your local block is running` | CORS blocking localhost OR server not running | Check Chrome flags, verify `block run` is active |
| `blockId mismatch` | Extension ID mismatch between local and remote | Re-run `block init` or fix `.block/remote.json` |
| `Module not found` | Missing npm dependency | `npm install <package>` |
| `EADDRINUSE :::9000` | Port in use from previous crashed server | Kill process on port 9000, restart |
| `Cannot read property of undefined` in yaml | Node.js version incompatibility | Use Node 18+ (avoid Node 12-14) |
| `export = is not supported` | Babel/TypeScript config issue | Reinstall from clean `block init` project |

---

## Development Checklist

Use this checklist when starting a new interface extension:

### Initial Setup
- [ ] Node.js 18+ installed
- [ ] `@airtable/blocks-cli` installed globally
- [ ] Airtable Team/Business/Enterprise plan
- [ ] Extension initialized with `block init`
- [ ] `package.json` updated with `interface-alpha` dependency
- [ ] Imports use `@airtable/blocks/interface/ui` path

### Before Coding
- [ ] Interface page created in Interface Designer
- [ ] Custom Extension element added to page
- [ ] ALL tables enabled in interface Data panel
- [ ] ALL fields enabled for each table
- [ ] Linked tables and their fields also enabled
- [ ] Interface saved after data panel changes

### During Development
- [ ] `block run` active in terminal
- [ ] Localhost URL entered in extension edit panel
- [ ] Chrome localhost flags configured (or using Firefox)
- [ ] Diagnostic component confirms data access
- [ ] Browser DevTools open for error monitoring

### Before Release
- [ ] Remove diagnostic component
- [ ] Test all CRUD operations
- [ ] Verify permissions handling (read-only users)
- [ ] Run `block release` with descriptive comment
- [ ] Stop dev server and test published version
- [ ] Verify extension works for other base users
