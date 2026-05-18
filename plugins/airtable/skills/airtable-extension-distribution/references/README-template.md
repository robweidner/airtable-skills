# Your Extension Name

One-line description of what this extension does.

> [!info] Status
> - Built against `@airtable/blocks` Interface Extensions SDK
> - Requires Node 22+ and the Airtable Blocks CLI
> - Tested in light + dark mode

---

## What it does

A short paragraph (2–4 sentences) describing the user-visible behavior. Screenshot helps.

---

## Required base schema

This extension expects the following tables and fields in your destination base. Field types must match — singleSelect choices are case-sensitive when present.

### Table: `Projects`
| Field | Type | Notes |
|---|---|---|
| `Name` | singleLineText | Primary field |
| `Status` | singleSelect | Choices: `Planning`, `In Progress`, `Done` |
| `Owner` | singleCollaborator | |
| `Due Date` | date | |

### Table: `Tasks`
| Field | Type | Notes |
|---|---|---|
| `Title` | singleLineText | Primary field |
| `Project` | multipleRecordLinks | Links to Projects |
| `Done` | checkbox | |

If your base uses different field names, the extension exposes table/field pickers in the properties panel — no code edits needed.

---

## Fork & run on your own base

### 1. Prerequisites

```bash
node -v                                # must be 22 or higher
npm install -g @airtable/blocks-cli
block set-api-key                      # paste a PAT with the `block:manage` scope
```

Create a PAT at <https://airtable.com/create/tokens>. Add your destination workspace under **Access**.

### 2. Get a `blockId` from your base

In the base where you want to run this extension:

1. Open **Extensions → Add an extension → Build a custom extension**
2. Pick a name. Airtable returns a `blkXXXXXXXXXXXXXX` blockId.
3. Leave the dialog open.

Also grab your **baseId** (`appXXXXXXXXXXXXXX`) from the URL.

### 3. Init from this template

```bash
block init <YOUR_BASE_ID>/<YOUR_BLOCK_ID> \
  --template=https://github.com/<owner>/<this-repo> \
  my-fork
cd my-fork
npm install
block run
```

### 4. Attach to your base

Back in Airtable, click the `</> Develop` button in the properties panel of the new extension. Accept the self-signed certificate on first run. The local block now loads inside the base.

### 5. Configure custom properties

In the Interface Designer properties panel for this extension, pick your tables and fields from the dropdowns. No code edits required.

---

## Troubleshooting

| Symptom | Likely cause |
|---|---|
| `block init` fails or hangs | Wrong Node version, PAT scope missing `block:manage`, or PAT doesn't have access to your workspace. |
| Extension renders blank | Custom properties unconfigured, OR the required fields haven't been added as data sources in the Interface Designer. |
| `getCellValue` throws / returns null | Field name mismatch with what this code expects. Either rename your fields to match the schema table above, or configure the field pickers in the properties panel. |
| `.block/remote.json` shows the wrong baseId | You cloned the repo instead of using `block init --template`. Delete `.block/`, re-run `block init <YOUR_BASE_ID>/<YOUR_BLOCK_ID>`. |
| Writes silently fail | Permission denied. Check `table.hasPermissionToCreateRecord()` — Interface Designer may have disabled editing. |

---

## License

[MIT](./LICENSE.md) — fork freely.
