---
name: airtable-extension-distribution
description: Fork, publish, and dev-mode Airtable Interface Extensions across bases. Use when sharing a custom extension on GitHub, forking someone else's extension into a new base, or troubleshooting the "original base" FORBIDDEN error in dev mode.
---

# Airtable Extension Distribution & Forking

This skill is the companion to **`airtable-extensions`** (the SDK reference). That skill teaches you to build a block; this one teaches you to ship it so someone else can fork and rebuild it against their own base.

> [!tip] Mental model
> A custom extension always lives inside a specific base. Your repo is just the source code — the binding to a base is one tiny file (`.block/remote.json`) plus the table/field names your code expects. Forking is the act of swapping the binding to a different base and (often) adapting field references.

---

## The 4-Stage Workflow

```
Build  →  Publish  →  Fork  →  Reconnect
 (you)    (GitHub)  (them)   (their base)
```

Most "it doesn't work after fork" failures happen at **Reconnect**. They have three root causes:

1. The publisher committed `.block/remote.json` (or `production.remote.json`) and the forker inherited a baseId/blockId they don't own.
2. The code hardcodes `tblXXX` / `fldXXX` IDs that only exist in the original base.
3. The README doesn't tell the forker which tables and fields they need to create.

The publisher prevents #1 and #2. Either side can fix #3 with documentation.

---

## `block run` vs `block release` vs the Develop Button — Mental Model

This trips up almost everyone. There are three independent things going on, and the error messages get confusing fast unless you understand which mode you're in.

| State | What's serving the JS | What the user sees |
|---|---|---|
| `block release` was run | Airtable's CDN (the bundle you uploaded) | The released version. No localhost needed. |
| `block run` is running + Develop button **ON** + correct port | Your laptop at `https://localhost:<port>` | Your local code, hot-reloading. |
| Neither | nothing | *"This extension does not have any releases yet"* |

Important behaviors:

- **The Develop button is the toggle.** When OFF, Airtable shows the released bundle (or "no releases yet" if you've never run `block release`). When ON, Airtable tries to load `https://localhost:<port>` — that's where your `block run` server lives.
- **Port matching is manual.** `block run` defaults to port 9000 but may shift to 9002 if 9000 is in use. The port shown in your terminal MUST match the port entered in the Develop dialog.
- **First run requires accepting a self-signed cert.** Open `https://localhost:9000` (or whatever port) in a browser, click through the warning, then return to Airtable.
- **`block release` ≠ `block run`.** Release uploads a bundle to Airtable's servers. After release, you can turn Develop OFF and the extension still works for everyone in the base — that's the production path.

---

## Managing `block run` Dev Servers — Start, Find, Stop, Kill All

After a few hours of development, your laptop can have 3-5 orphaned dev servers running. They hold ports, sometimes serve stale bundles, and silently outlive the terminal windows that launched them. Here's the toolkit.

### `block run` allocates TWO consecutive ports

Each `block run` instance claims the main HTTPS port AND `port+1` (the internal bundler proxy). Verified with `lsof`:

```
node  PID  ... TCP *:9000 (LISTEN)   ← main HTTPS server
node  PID  ... TCP *:9001 (LISTEN)   ← bundler proxy (NOT free!)
```

So `block run` default is 9000+9001. `block run --port=9002` uses 9002+9003. **If you want three dev servers running, use 9000 / 9002 / 9004** — the odd ports are owned by their neighbors. Trying to start a fourth server on 9001 will error out.

### Start

```bash
# Default — uses 9000+9001
block run

# Explicit port — uses 9002+9003
block run --port=9002

# Common pattern (many Airtable templates ship with this in package.json):
npm run dev:9002
```

### Find what's running

```bash
# Which ports are taken in the dev range?
lsof -nP -i TCP:9000-9010 -sTCP:LISTEN

# All Airtable CLI processes
ps aux | grep -E "blocks-cli|block run" | grep -v grep

# Which folder is a specific PID running from? (tells you which blockId it serves)
lsof -p <PID> | awk '$4=="cwd"{print $NF}'
```

### Stop one server

```bash
# By port — kills whoever holds port 9002 (and freed 9003 follows automatically)
lsof -ti :9002 | xargs kill

# By PID directly
kill <PID>

# In the terminal where it's running:
# Ctrl-C
```

### Kill ALL dev servers (nuclear)

```bash
pkill -f "block run"
# Or, more aggressive — also kills the bundler/esbuild subprocesses:
pkill -f "@airtable/blocks-cli"
```

### Routine cleanup

Before starting a new session, scan for stragglers:

```bash
lsof -nP -i TCP:9000-9010 -sTCP:LISTEN
```

Anything you don't recognize, kill. They're free to leak across terminal-close events if backgrounded with `&` or `run_in_background`, and they survive until reboot otherwise.

### `block` CLI command reference (verified from `block --help`)

```
add-remote      [Beta] Add a new remote configuration
init            Initialize an Airtable extension project
list-remotes    [Beta] List remote configurations
release         Release a build to an Airtable base
remove-remote   [Beta] Remove a remote configuration
run             Run the extension locally
set-api-key     Set a personal access token (with block:manage scope)
submit          Submit extension for review for the Airtable Marketplace
```

The `add-remote` / `list-remotes` / `remove-remote` trio is officially marked Beta. They're the multi-base support path for one folder.

Notable CLI quirks:
- `block release` accepts only `--help` and `--remote`. **No `--comment` flag** — older docs and tutorials reference one. Passing it produces: `Error: ❌ This block cannot be released with a comment. Run block release without --comment. Code: releaseCommandBlock1CommentUnsupported`
- `block run --port=N` explicitly documents: *"The server will listen for HTTP on PORT + 1"* — official confirmation of the dual-port allocation.
- No verbose / debug flag for `block release`. If you need to diagnose where a release actually went, you have to inspect Airtable's Releases tab manually.

### Knowing which dev server serves which blockId

Each `block run` process is bound to the folder it was started in, which means its `.block/remote.json` determines the blockId. From the `cwd` of the process:

```bash
for pid in $(pgrep -f "block run"); do
  cwd=$(lsof -p $pid 2>/dev/null | awk '$4=="cwd"{print $NF; exit}')
  echo "PID $pid → cwd $cwd"
  [ -n "$cwd" ] && [ -f "$cwd/.block/remote.json" ] && \
    echo "  serves: $(cat "$cwd/.block/remote.json" | python3 -c 'import json,sys; d=json.load(sys.stdin); print(d.get("blockId"), "in base", d.get("baseId"))')"
done
```

This is the cleanest way to confirm "Server URL `https://localhost:9002/` in Airtable → which extension am I actually developing?"

---

## PUBLISHER — Prep Your Block to be Forked

### 1. Get `.gitignore` right

`.block/remote.json` pins your block to YOUR baseId and blockId. It must NOT travel to forkers.

Canonical `.gitignore` for a shareable block:

```gitignore
/node_modules
/.airtableblocksrc.json
/build
.env
.env.*
.DS_Store
.tmp/
.block/
```

The `.block/` line is the load-bearing one. If you've already committed `.block/remote.json` or `.block/production.remote.json` to history, run `git rm -r --cached .block/` and recommit.

See `references/gitignore-template` for a drop-in.

### 2. Leave `block.json` alone

`block.json` itself contains no base-specific data — just `version` and `frontendEntry`. It's already portable. Don't touch it.

### 3. Don't hardcode `tblXXX` / `fldXXX` IDs

Table and field IDs are unique per base. A forker's base will have different IDs. Three approaches, in increasing order of forker-friendliness:

| Approach | What it looks like | Fork friction |
|---|---|---|
| Hardcoded IDs | `table.getField('fldZ7X9...')` | **High** — every ID has to be replaced by hand |
| Named lookups | `table.getFieldByNameIfExists('Status')` | **Medium** — forker must use exact field names |
| Custom properties | `useCustomProperties` so builders pick fields in the sidebar | **Low** — forker picks fields via UI, no code edits |

Strongly prefer custom properties for fork-ready blocks. See the *Custom Properties for Builders* section in the `airtable-extensions` skill for the full pattern.

### 4. Document the required schema in your README

Forkers can't run your block until their base has the tables and fields it expects. Spell this out. Minimum content:

- Each required table by name
- Each required field per table, with type
- For singleSelect / multipleSelect fields: the exact choice names if your code branches on them
- Any required permissions (Editor on the table, etc.)

A drop-in template lives at `references/README-template.md`.

### 5. Provide a one-line `block init` invocation in the README

Make it copy-pasteable. Example:

```bash
block init <YOUR_BASE_ID>/<YOUR_BLOCK_ID> --template=https://github.com/<you>/<repo> <dest-dir>
```

Replace placeholders with their actual IDs. The exact CLI install + auth steps are in the *CONSUMER* section below — link to it from your README.

### 6. License + attribution

If you want forks to be allowed, add a permissive license (MIT, Apache-2.0). Mention the original repo in the README if you'd like attribution preserved.

---

## Cross-Organization / Cross-Workspace Forks (the hardest case)

Custom extensions live inside a specific *workspace*, not just a base. **A blockId from workspace A cannot be developed against in workspace B**, full stop. You'll get this error verbatim:

> *"You can only run your development block in the original base where it was created."*

This usually surfaces when you're trying to hand a block to a collaborator in a different org (e.g., your repo lives against your client's workspace, your contractor is in their own workspace).

### Who can create extensions

To create a new custom extension, you need **creator-level access on the workspace**. Read-only collaborators and guest collaborators on a base **cannot** create extensions in that workspace, even if they can fully edit the base.

If the forker doesn't have that access, they can't fix the error on their own. The workspace owner has to step in.

### The collaborator-handoff workflow (when the forker can't create extensions)

If your forker is a guest in a workspace where they can't create extensions, the workspace owner does the creation:

1. **Workspace owner** opens the destination base in the target workspace
2. **Owner** → Extensions → Add an extension → Build a custom extension
3. **Owner** names the extension and copies the new `blkXXXXXXXXXXXXXX` ID
4. **Owner** adds the forker as a collaborator on the new extension
5. **Forker** receives the new blockId from the owner
6. **Forker** updates `.block/remote.json` with `{"baseId": "appXXX...", "blockId": "blkXXX..."}` — that's the only file that needs to change for cross-org
7. **Forker** creates a new custom interface page in the destination base, adds the new extension to it
8. **Forker** runs `block run` from the correctly-bound folder

### The alternative — duplicate the base into your own workspace

If owner-driven creation isn't an option (slow client, no access), the forker can duplicate the base into their *own* workspace, then create their own extension there, then remap the data later. Heavier, but unblocks development.

### When you want one folder to serve multiple bases

`.block/remote.json` only binds to one base. If you legitimately need one codebase that runs against multiple bases (e.g., a template you actively develop and ship to several clients), use the undocumented:

```bash
block add remote
```

This command is in beta and not on Airtable's docs site — visible only via `block --help`. It adds an additional named remote to your block folder. You can then `block run --remote=<name>` to target a different base from the same source tree.

Trade-offs:
- ✅ One source of truth across bases
- ✅ No need to maintain multiple forks
- ⚠️ Undocumented; behavior may change
- ⚠️ Each remote still needs its own `blkXXX` created in its base

---

## CONSUMER — Fork & Connect to Your Base

### 1. Install the CLI and set your PAT

```bash
npm install -g @airtable/blocks-cli       # Node 22+ required
block set-api-key                          # paste a PAT with block:manage scope
```

Create the PAT at `airtable.com/create/tokens`. Required scope: **`block:manage`**. Add your destination workspace under "Access".

### 2. Get a `blockId` — Builder Hub is the canonical path for Interface Extensions

The official docs (<https://airtable.com/developers/interface-extensions/guides/hello-world-tutorial>) say:

> Head to the Builder Hub and click Create new extension. If prompted to choose an extension type, select Interface.

> [!warning] Two creation paths exist — make sure you pick the right one
>
> Airtable has a *second* path inside each base: **Tools → Extensions → Add an extension → "Build a custom extension"** tab. **That path creates the legacy Apps SDK** (`@airtable/blocks/ui`), which lives in Dashboards, not Interface Designer pages. It uses `apps-hello-world` template, has its own list of starter templates ("To-do list", "Print records", etc.), AND a **"Remix from GitHub"** option that the Builder Hub flow lacks.
>
> If you want a modern Interface Extension that imports `@airtable/blocks/interface/ui`, **use Builder Hub**, not the in-base "Build an extension" dialog.

The canonical Builder Hub path:

1. Click your **account icon** in the bottom-left of any Airtable page
2. **Builder Hub** → **Developers** → **Custom extensions** → **Create new extension**
3. If prompted to choose an extension type, select **Interface**
4. Pick a starter template: Hello world (JS), Hello world (TypeScript), Heatmap, Sliding Bar Chart (and a couple others Airtable has added — Map, Org chart)
5. Fill in **Extension name** and **Tagline** (both required) and click **Create extension**

**Fast path**: navigate directly to `https://airtable.com/create/extensions/new`. Skips the menu navigation. (Note: the direct URL may bypass the "extension type" picker — you'll just land on the template selector. If you have any doubt about which kind you're creating, go through the click-through path to see the type picker.)

### 2a. Builder Hub flow (creates an Interface Extension)

To create a new custom extension you can develop against:

**Fast path:** navigate directly to `https://airtable.com/create/extensions/new`. Skips the menu navigation entirely.

**Click-through path** (if you prefer the UI):

1. Click your **account icon** in the bottom-left corner of any Airtable page
2. Select **Builder Hub**
3. In the left sidebar, go to **Developers** → **Custom extensions**
4. Click **Create new extension**
5. **Pick a template** — Airtable forces this; you can't start truly blank. Pick **Hello world** (JS) if you plan to override it with your own code. The 6 options as of late 2025:
   - Hello world (JS) / Hello world (TypeScript) — bare-bones starters
   - Heatmap / Sliding bar chart / Map / Org chart — fuller example apps
6. Fill in **Extension name** and **Tagline** (both required)
7. Click **Create extension**
8. Airtable returns the new `blkXXXXXXXXXXXXXX`. Copy it — you'll need it for `block init`.

Also grab the **baseId** (`appXXXXXXXXXXXXXX`) of the base where you want the extension to run, from that base's URL.

> [!tip] You aren't married to the template
> The template choice only determines the initial files when you `block init`. Once init is done, you can overwrite every file with your own code — the blockId persists, the template content doesn't matter. This is the "pick a template, then override" pattern.

### 3. Use `block init --template` — the canonical fork command

This is THE command most people miss when forking. It clones the repo AND writes the correct `.block/remote.json` for your base in one step:

```bash
block init <YOUR_BASE_ID>/<YOUR_BLOCK_ID> \
  --template=https://github.com/<author>/<repo> \
  my-fork-dir
cd my-fork-dir
npm install
block run
```

> [!info] What Airtable's UI generates for you
> When you create an extension via the Builder Hub flow, the setup page literally shows the exact command, with `NONE` as the baseId placeholder:
>
> ```
> block init NONE/<blkXXX> \
>   --template=https://github.com/Airtable/interface-extensions-hello-world \
>   <directory_name>
> ```
>
> Three things worth knowing:
> - `NONE` is the canonical placeholder when you haven't bound to a base yet. The CLI replaces it on first `block run` or you can update `.block/remote.json` manually.
> - `<directory_name>` is auto-derived from the extension name: spaces → underscores, parens stripped, lowercased. `Performance Attribution (AG dev)` becomes `performance_attribution_ag_dev`.
> - The Hello world JS template repo is `https://github.com/Airtable/interface-extensions-hello-world` (use this for "scaffold then override" — see below).
> - The setup page also shows: `npm install -g @airtable/blocks-cli`, `block set-api-key`, `block run`, `block release`. That's the full official workflow.

### 3a. "Remix from GitHub" — the docs-prescribed fork workflow

This is the official workflow Airtable documents at <https://airtable.com/developers/interface-extensions/guides/remix-from-github>. It's just like the Hello world flow with a different template URL:

```bash
# 1. Create a NEW extension shell in Builder Hub (returns <YOUR_blkXXX>)
#    → https://airtable.com/create/extensions/new

# 2. Init with the source repo as the template:
block init NONE/<YOUR_blkXXX> \
  --template=https://github.com/<author>/<source-repo> \
  my-remix
cd my-remix

# 3. Run + add to a base + Develop — same as Hello world tutorial
block run
```

The key insight: **the template URL can be ANY public GitHub repo containing an Interface Extension**, not just Airtable's templates. Pointing `--template=` at your own repo (or someone else's) is the canonical fork mechanism.

Airtable maintains a list of remixable Interface Extensions at <https://github.com/Airtable/> (filter for `interface-extensions-*`). Notable ones:
- `interface-extensions-hello-world` (JS) / `interface-extensions-hello-world-typescript` (TS) — bare scaffolds
- `interface-extensions-map` — Mapbox + Tailwind reference
- `interface-extensions-word-cloud-typescript` — D3 + dark mode reference
- `interface-extensions-sliding-bar-chart` — MUI + sliders
- `interface-extensions-heatmap` — data viz
- `interface-extensions-embed` — iframe embeds + custom properties

### 3b. "Pick a template, then override" workflow

When Airtable's Builder Hub forces you to pick a template at creation time, you don't have to keep the template's code. The blockId is what matters; the template files are disposable.

```bash
# 1. Init with Hello world (gets you a fresh .block/remote.json + scaffold)
block init NONE/<YOUR_blkXXX> \
  --template=https://github.com/Airtable/interface-extensions-hello-world \
  <directory>
cd <directory>

# 2. Copy your real code in, overwriting the scaffold
cp -R /path/to/your-real-block/frontend ./frontend
cp /path/to/your-real-block/package.json ./
cp /path/to/your-real-block/block.json ./
# (and any other files: tailwind.config.js, eslint.config.mjs, etc.)
# DO NOT copy .block/ — keep the one block init created.

# 3. Install + run
npm install
block run
```

This is the cleanest way to bind YOUR existing code to a freshly-created blockId in a new base.

### 4. Alternative path — when you've already cloned the repo

If you forked via GitHub UI and ran `git clone`, the `.block/` directory may or may not exist. Either way:

```bash
rm -rf .block/                                # drop any inherited binding
block init <YOUR_BASE_ID>/<YOUR_BLOCK_ID>     # writes a fresh .block/remote.json
npm install
block run
```

### 5. Add the extension to an interface page

A blockId by itself doesn't render anywhere — you have to attach it to an interface page or dashboard.

1. Open the base, go to **Interfaces**
2. Open the page where you want the extension (or create a new one — for full-bleed extensions, the "Custom layout" option works well)
3. Add a new element → **Custom** → pick your extension (by name, from "My custom elements") from the list
4. **Pick the source table.** Airtable forces you to choose one at add-time. Pick the table that holds the **majority of the data your extension reads** — it determines the default data scope, permissions, and which fields show up in `useCustomProperties` field pickers.
5. You can change the source later via the right-hand sidebar: select the extension element → look under the **Data** heading → change **Source**.

### 6. Hit Develop, point it at your local dev server

Select the extension element. In the right sidebar, under **Development**:

1. On first add you'll see: *"This extension does not have any releases yet."* That's normal — you haven't run `block release` yet, and Develop is off. The extension can't render anything until one of those is true.
2. Set **Server** to the URL your `block run` is using — e.g., `https://localhost:9002/` if you started it with `--port=9002`. **Don't trust the default** (`https://localhost:9000/`) — it'll point at the wrong dev server if you're running multiple.
3. Click `</> Develop`.

If you see *"You can only run your development block in the original base where it was created"* here, 90% of the time the cause is a port mismatch — the Server field points at a dev server that's serving a *different* blockId's bundle. Check that the port matches the one `block run` printed for *this* extension's folder.

### 7. Configure custom properties

If the author followed the best practices in the PUBLISHER section, open the extension's properties panel in Interface Designer and pick your tables and fields from the dropdowns. The block should come alive.

If the author hardcoded names instead, your fields must match those names exactly (case-sensitive) — see the troubleshooting section.

---

## Troubleshooting Decision Tree

> [!warning] Start here — verbatim error matching
> Below are the actual error strings the Airtable CLI and UI produce. Match yours against these *first* before debugging code.

### The two-SDK / two-render-context architecture (critical to understand)

After multi-hour live testing, the picture is:

| Where the extension renders | Which SDK runs there | How it was created |
|---|---|---|
| **Interface Designer pages** (full page Custom layout, Dashboard layout on an Interface) | Interface Extensions SDK (`@airtable/blocks/interface/ui`) | Builder Hub at `https://airtable.com/create/extensions/new` |
| **In-base Dashboards** (the right-sidebar panel in the data view) | Apps SDK / legacy Blocks SDK (`@airtable/blocks/ui`) | In-base **Tools → Extensions → Add an extension → "Build a custom extension" tab** |

**The two are not interchangeable.** Bundling Interface Extension code (`@airtable/blocks/interface/ui` imports) and serving it to an in-base Dashboard context produces this runtime error in the extension viewport:

```
Error: Unexpected import when running block in base
    at node_modules/@airtable/blocks/dist/esm/interface/assert_r...
    at __init (https://localhost:<port>/__runFrame/bundle.js:...)
    at runBlock (https://localhost:<port>/__runFrame/bundle.js:...)
```

The SDK asserts the runtime context at load time and refuses to run if it doesn't match.

### Dev-mode behavior by combo (verified May 2026)

| Created via | Installed on | Dev mode works? |
|---|---|---|
| Builder Hub (Interface Extension) | Interface Designer Custom Layout page | ❌ Fires `FORBIDDEN: You can only run your development block in the original base where it was created`. Unresolved — see "Open mystery" below. |
| Builder Hub (Interface Extension) | In-base Dashboard | ❌ Fires `Unexpected import when running block in base` (SDK mismatch — Interface code in Apps context) |
| In-base "Build an extension" (Remix from GitHub, points at `interface-extensions-*` repo) | In-base Dashboard | ❌ Same `Unexpected import` error. Init produces non-NONE baseId which fixes the "original base" error, BUT the SDK mismatch wall replaces it. |
| In-base "Build an extension" (Hello world / Remix from `apps-*` repo) | In-base Dashboard | ✅ **Works.** Apps SDK code in Apps context. Init uses non-NONE baseId. `block run` → Edit Extension dialog → paste `https://localhost:<port>` → Start editing extension. |

### SDK category is locked in at creation flow — NOT by template URL

A critical and non-obvious finding: when you create an extension via Airtable's UI, the **category is set by which flow you used** (Builder Hub vs. in-base), not by the template repo you point at.

- Builder Hub → categorized as **Interface Extension** → appears in Interface Designer's *Custom Layout → Add a custom element* picker → does NOT appear in the in-base Dashboard's *Add extension* search.
- In-base "Build an extension" → categorized as **Apps SDK** → appears in in-base Marketplace search → does NOT appear in Interface Designer's Custom Layout picker.

**Even if you use Remix from GitHub in the in-base flow and point it at `interface-extensions-hello-world`** (the new SDK template), Airtable still categorizes the result as Apps SDK. The bundle will use Interface SDK imports but the install context is Apps SDK → runtime throws `Unexpected import when running block in base`.

There is no UI path to create an extension that is **both** categorized as Interface Extension **and** has a non-NONE baseId in its `block init` command. That combination is what would unlock dev mode for the modern Interface Extensions stack — and it doesn't exist as of testing.

### Practical implication

If your code uses `@airtable/blocks/interface/ui` (i.e., it's an Interface Extension — including any block created with `interface-extensions-hello-world` template, including your own fork of an Interface Extension repo), you have:

1. ✅ **Production**: `block release` works, the released bundle renders correctly on Interface Designer pages.
2. ❌ **Dev mode**: no verified working path as of testing. Builder Hub creates the right *kind* of extension but its dev-mode-against-arbitrary-base is broken. In-base creates the wrong *kind*.

If your code uses `@airtable/blocks/ui` (i.e., it's an Apps SDK / legacy Blocks app — including any block from the `apps-*` template repos):

1. ✅ **Production**: `block release` works, renders in in-base Dashboards.
2. ✅ **Dev mode**: in-base "Build an extension" flow, then `block run`, paste localhost URL into Edit Extension dialog.

### THE FIX for V2 Interface Extension dev mode: `"baseId": "NONE"`

**This was the wall for 12 test iterations. The fix is one character.**

For V2 / Interface Extensions, `.block/remote.json` should look like this:

```json
{
    "blockId": "blkXXXXXXXXXXXXXX",
    "baseId": "NONE"
}
```

**The string `"NONE"`** — not your real baseId. Not empty. Not a placeholder you fill in later. The literal string `NONE`.

The reason: the local `block run` server has an endpoint at `/registerBlockInstallationMetadata`. Airtable's iframe POSTs to it with `{applicationId, blockId, blockInstallationId}`. For V2 blocks, Airtable's iframe sends `applicationId: "NONE"` because V2 blocks are base-agnostic. The local server then checks:

```js
if (req.body.applicationId !== remoteConfig.baseId) {
    res.status(403).send({error: 'FORBIDDEN', message: 'You can only run your development block in the original base where it was created.'});
}
```

If you patched `.block/remote.json` to have your real baseId (e.g., `apphMObIrt3jO3C8F`), the strings don't match → FORBIDDEN.

The fix: **leave it as `"NONE"`**. `block init NONE/<blkXXX>` writes it correctly. Don't "fix" it.

### Why dev mode appears broken for Interface Extensions — source-code-level root cause

After reading the `blocks-cli` source (`/Users/rob/.nvm/versions/node/<version>/lib/node_modules/@airtable/blocks-cli/lib/`), the architecture becomes clear:

**`block init` is entirely local** — it does NOT call Airtable's API. It just:
1. Downloads the template tarball from GitHub
2. Extracts + runs `npm install`
3. Writes `.block/remote.json` with `{blockId, baseId}` from your command args

That's it. No binding registration, no Airtable backend writes. So the baseId you pass in the init command is ONLY used locally (and as a switch in `block release`).

**`block run` also makes zero API calls** — it just starts an HTTPS server on a port. Nothing is registered anywhere.

**`block release` is the only command that hits Airtable's API.** And it branches on baseId:

```js
// from release.js
if (baseId === V2_BLOCKS_BASE_ID) {  // V2_BLOCKS_BASE_ID = 'NONE'
    airtableApi = new AirtableBlockV2Api({ blockId, apiKey, ... });
} else {
    airtableApi = new AirtableLegacyBlockApi({ baseId, blockId, apiKey, ... });
}
```

**`NONE` is the official sentinel value for V2 Blocks = Interface Extensions.** Not a placeholder — a deliberate marker.

The V2 API endpoints all live under `/v2/blocksV2/<blockId>/...` (no baseId in URL):
- `/v2/blocksV2/<blockId>/builds/start`
- `/v2/blocksV2/<blockId>/releases/create`
- `/v2/blocksV2/<blockId>/codeUpload/create`

V2 blocks are **base-agnostic by design**. They don't have an "original base" in the data model.

### The actual root cause of the FORBIDDEN error

Putting it together:
1. Builder Hub correctly creates V2 (Interface Extension) blockIds
2. `block init NONE/<blkXXX>` correctly marks them as V2
3. `block release` correctly uses V2 API endpoints (no baseId required)
4. BUT — Airtable's iframe-side dev-mode authorization check is V1-style: it expects an "original base where the block was created" and rejects if it doesn't match the current base.
5. V2 blocks don't have an original base (by design). The check resolves against null/undefined.
6. **Therefore the V1-style check fails for every V2 block in every base.**

The error message *"You can only run your development block in the original base where it was created"* is V1 language fired against V2 blocks where the concept doesn't exist. This is either an undocumented precondition we're missing (a backend flag that enables V2 dev mode), or a genuine product bug between the CLI's V2 release path and the iframe's V1-style dev check.

### Open mystery: dev mode fires "original base" error against Builder Hub-created extensions

**Status**: as of May 2026, after multi-hour live testing, we have **not found a way to make `block run` dev mode work for an Interface Extension created via Builder Hub** when developing against an arbitrary base. The error fires regardless of which combinations we tried.

> [!warning] What we've ruled out — none of these fix the error
> - ❌ AG-code-copy contamination (tested with pure Hello world template, no code modifications)
> - ❌ Port mismatch (set Server URL to the actual `block run` port)
> - ❌ Audience = "Collaborators only" (changed to "Everyone in specific workspaces" + selected the workspace containing the base — verified via the All Workspaces page that the base lives in the workspace we picked)
> - ❌ Stale cert (system-keychain Always Trust applied)
> - ❌ Stale bundle (verified `curl -k https://localhost:<port>/bundle.js` returns 200 + multi-MB bundle)
> - ❌ `.block/remote.json` baseId (manually patched from `NONE` to the actual baseId)
> - ❌ Running `block release` first to "warm up" the binding (released successfully, no change)
> - ❌ **Using `block init <BASE_ID>/<blkXXX>` instead of `block init NONE/<blkXXX>`** — for a Builder Hub-created blockId, even when the init command uses a real baseId (matching the in-base flow's syntax), dev mode still throws FORBIDDEN. The `block init` command doesn't establish the binding; the binding is established by the *creation flow on Airtable's backend* (Builder Hub vs in-base), and Builder Hub demonstrably does not establish it for any base.

> [!info] What the official template repo reveals
> Airtable's [official Hello World repo](https://github.com/Airtable/interface-extensions-hello-world) has `.block/remote.json` **committed** with `{"blockId": "", "baseId": ""}` (empty strings, not `NONE`). They also don't put `.block/` in `.gitignore`. This contradicts the publisher-side advice in this skill for OFFICIAL Airtable templates only — Airtable's templates ship with empty `.block/remote.json` placeholders that `block init` overwrites. For YOUR fork-friendly repos, still gitignore `.block/` — empty placeholders don't help anyone and your own `.block/remote.json` will leak your baseId/blockId to forkers.

> [!tip] Bonus gotcha discovered: incomplete in-base extension creation
> If you start the in-base **"Build a custom extension"** flow and cancel out of the *"Install the Blocks CLI"* setup wizard, the extension's blockId is **reserved in Airtable's backend** but the extension itself doesn't appear in that base's marketplace search. The CLI thinks it exists (`block init <BASE>/<blkXXX>` works), and Builder Hub lists it, but you can't add it to the dashboard via the normal Add an Extension flow. **Always run the wizard to completion** — even if you intend to overwrite the template files later.

**What the docs claim**: <https://airtable.com/developers/interface-extensions/guides/hello-world-tutorial> describes a simple path — Builder Hub → block init → block run → add to interface → click Develop. The error we hit isn't documented in the FAQ or any other guide page.

**Hypothesis for future testing**:
- The extension may need to be RELEASED via the in-base "Build a custom extension" → Remix from GitHub UI flow (which generates `block init <BASE_ID>/<blkXXX>` with a non-NONE baseId, suggesting that's the dev-binding mechanism). The Builder Hub path's `NONE` placeholder may genuinely never resolve to a base for dev purposes.
- OR: there's a workspace/account-level setting (e.g., "Enable Developer Mode") that we couldn't find in the UI.
- OR: this is a known platform bug being actively addressed by Airtable.

**Pragmatic workarounds (untested but plausible)**:
- **Use the legacy Apps SDK path** via in-base "Build an extension" → Hello world template. Loses Interface Designer integration, but Apps SDK dev mode works against the base where created.
- **Find someone with a working setup** and reverse-engineer it.
- **File a support ticket** referencing the verbatim error and this skill's documented attempts.
- **Continue using `block release`** as the only path to ship changes — release after each iteration, accept the slow loop until dev mode is unlocked.

---

### `"You can only run your development block in the original base where it was created"`

Airtable surfaces this in the extension viewport (not as a banner) when the Develop toggle is ON. The exact JSON shape:

```json
{
  "error": "FORBIDDEN",
  "message": "You can only run your development block in the original base where it was created."
}
```

The error has *"Copy error"* and *"Reload extension"* buttons beneath it. Reloading won't help — the cause is the blockId, not the extension code. The blockId in your `.block/remote.json` was created in a different base than the one you're trying to develop against. This is the #1 cross-org / fork error.

**Three ways out, in order of effort:**

1. **`cd` to the original base's folder.** If you have a different folder/repo bound to the base where the block was actually created, run `block run` from there instead.
2. **Create a new extension in the target base** (or have the workspace owner do it — see *Cross-Organization Workflow* above). Update `.block/remote.json` with the new `appXXX`/`blkXXX`. Recreate the custom interface page in the new base.
3. **Use `block add remote`** if you intend to actively maintain the codebase against multiple bases.

### `"This extension does not have any releases yet"`

Develop button is OFF and you've never run `block release` against this blockId. Two fixes:

- **Run `block release` once** to push a production bundle, OR
- **Toggle the `</> Develop` button ON** and enter the port your local `block run` is using (and accept the self-signed cert at `https://localhost:<port>`).

### `"Valid block CLI API key"` / 401 / 403 from the CLI

PAT expired, revoked, or scoped to the wrong workspace. Re-run `block set-api-key` with a fresh token (PAT with `block:manage` scope, scoped to the destination workspace).

### `block init` fails or hangs

- **Node version** — must be Node 22+. Check with `node -v`.
- **PAT scope** — must include `block:manage`. Re-run `block set-api-key`.
- **PAT workspace access** — the PAT must list your destination workspace under "Access". Tokens scoped to a different workspace fail silently.
- **Network** — the `--template=` URL must be reachable. If GitHub is blocked behind SSO/IP allowlists in your org, the clone step fails partway.

### `block run` succeeds but Airtable shows a different error than you expected

You're probably running from the **wrong folder**. Coding agents (Cursor, Claude Code, Windsurf) sometimes spin up duplicate or sibling folders that look like your block source but bind to a different blockId. Symptoms:

- Errors that don't match the code you just edited
- "Different error" appearing after the agent's last refactor
- Two folders in your sidebar with similar names

**Triage in this order:**

1. In the terminal where `block run` is running, check `pwd` — is it the directory you think?
2. `cat .block/remote.json` — does the `blockId` match the extension you're trying to develop?
3. If you have multiple folders, `cd` to the intended one and re-run `block run`.

### Local server is up but the Develop dialog says "block is not running"

Port mismatch. `block run` prints something like `Server listening at https://localhost:9000` — that port number must match what you enter in the Develop dialog. The default 9000 can shift to 9002, 9003, etc. if 9000 is in use. Always read the actual port from terminal output, don't assume.

Also check that you accepted the self-signed cert at `https://localhost:<port>` in a browser tab first.

> [!tip] Running two block servers in parallel
> Need to develop two extensions at once (e.g., during a migration)? `block run --port=9002` forces a non-default port. The CLI also accepts a copy-paste `dev:9002` npm script pattern that many of Airtable's example templates ship with. Once both are running, set each extension's *Server* field in Airtable's Develop dialog to its respective port. Don't forget to update the Develop dialog — Airtable doesn't auto-detect a port change.

### Cursor's preview pane is a dead end for Airtable — use a real browser

Two independent failures stack inside Cursor's webview (and VS Code's Simple Browser, Windsurf, etc.):

1. **Cert layer (fixable):** Cursor's built-in *Browser Tab* doesn't even render Chrome's cert interstitial. Navigating to `https://localhost:9000` produces a hard-fail page with just `Connection Failed`, `ERR_CERT_AUTHORITY_INVALID`, and a "Restart Browser" button — no Advanced, no Proceed, no link to bypass. Typing `thisisunsafe` doesn't work either; the page has no input target for it. **Fixable** with macOS Keychain trust (option 2 below).
2. **App layer (NOT fixable):** Even with the cert trusted, Airtable itself detects Cursor's User-Agent and shows: *"The browser you're currently using is not supported. Update your browser to the latest version, or try another browser."* This isn't a cert issue — it's Airtable refusing to render in non-mainstream browsers. There's no setting on your end to override this.

The honest answer: **don't try to use Cursor's preview pane for Airtable interface development.** It's two-strikes-out, and only the first strike has a fix.

Same dead-end appears with browser automation: the Chrome cert warning lives at `chrome-error://chromewebdata/`, an internal page that extensions like Claude-in-Chrome and Playwright can't attach to. You'll see *"Cannot attach to this target"*. Manual click-through works in a real Chrome window, but breaks for any tooling driving the browser.

**1. Cursor for code editing, real browser for Airtable (zero setup; the actual right answer)**

This is the workflow that works. Open Airtable in Chrome or Safari, do the cert click-through there once, edit your block code in Cursor. The two stay in sync because `block run` is hot-reloading — your code changes appear immediately in the Airtable iframe in your real browser.

**Pro tip for Chrome:** you don't have to click Advanced → Proceed. Type `thisisunsafe` anywhere on the warning page — Chrome treats those exact keystrokes as a bypass shortcut.

**2. macOS Keychain *Always Trust* — the only fix that makes Cursor work**

This makes the cert trusted at the OS level. Cursor's webview, VS Code's Simple Browser, the Claude-in-Chrome extension, and any other Chromium-based view read from the system trust store and **never show the warning in the first place** — and since there's no warning, the click-through-that-doesn't-exist-in-Cursor stops being a blocker.

```bash
# The cert's path depends on your Node version + npm global root:
CERT="$(npm root -g)/@airtable/blocks-cli/keys/server.crt"
echo "$CERT"   # confirm the file exists before continuing

# Add to System keychain with always-trust (sudo + Touch ID/password):
sudo security add-trusted-cert -d -r trustRoot \
  -k /Library/Keychains/System.keychain "$CERT"
```

GUI alternative: open **Keychain Access** → drag `server.crt` into the **System** keychain → double-click the cert → expand **Trust** → set *"When using this certificate"* to **Always Trust**.

**After running this:** reload the Airtable interface in Cursor's webview. The iframe pointed at `https://localhost:<port>` should load with no warning. No click-through, no failure.

> [!warning] The cert regenerates — this is why "it worked before" stops working
> The Airtable CLI rotates `server.crt` periodically. Look for `.bak-YYYYMMDD` files in the keys directory — those are previous certs you may have trusted before. After a rotation, the **new** `server.crt` is untrusted again, and Cursor's Browser Tab will start failing with `ERR_CERT_AUTHORITY_INVALID` even though you "fixed this once already." Re-run the trust step on the current `server.crt`. Same applies after a Node version upgrade (the path includes `vXX.XX.XX`). After trusting, **restart Cursor** (or close + reopen the Browser Tab) — Electron caches cert validation results in-memory.

**3. Replace the CLI's cert with one from `mkcert` (most setup, most durable)**

If the cert keeps rotating and you don't want to re-trust every few weeks, install `mkcert` (`brew install mkcert nss && mkcert -install`), generate a locally-trusted cert for `localhost`, and swap it into the CLI's `keys/` directory. Cursor's webview, all browsers, and any subprocess will trust it without prompting. Trade-off: you'll need to re-apply this swap any time the `blocks-cli` package updates.

### `block run` starts but the extension renders blank

- **Custom properties not configured** — open the extension's properties panel and pick your tables/fields.
- **Field not added as a data source** — Interface Extensions only see fields that the *builder* has explicitly added to the layout. Even fields that exist on the table are invisible until added. See the *Debug Panel Pattern* section in the `airtable-extensions` skill.
- **No records visible** — check view filters in Interface Designer; the extension only sees records Designer exposes.

### `getCellValue` throws or returns null after fork

- **Field name mismatch** — author's code uses `getFieldByNameIfExists('Status')`, your base has a field called `Stage`. Either rename your field, or open a PR to the original to replace name lookups with `useCustomProperties`.
- **Field not exposed in Designer** — same as the blank-render case. The field exists on the table but isn't visible to the extension.

### Extension installs but writes nothing / silently fails

- **Permissions** — call `table.hasPermissionToCreateRecord()` and friends. Interface Designer can disable editing per-field even when you're a base admin.
- **Array overwrite trap** — see *Array Field Append Pattern* in the `airtable-extensions` skill. Especially common for linked records, attachments, multi-select.

### `.block/remote.json` keeps showing the wrong baseId

The author committed it. Delete `.block/`, re-run `block init <YOUR_BASE_ID>/<YOUR_BLOCK_ID>`, and open an issue on the upstream repo asking them to add `.block/` to `.gitignore`.

### `account_inactive` / `401` / `403` from the CLI

PAT expired, revoked, or scoped to the wrong workspace. Re-run `block set-api-key` with a fresh token that includes the destination workspace.

### Forker can run the block but `block release` fails for them

Expected — `block release` writes a bundle to the block's hosted record, which only the OWNER of that block (in the forker's base) can do. The PAT identity must match the workspace that owns the block. This isn't related to the fork; it's a PAT/workspace mismatch.

---

## Pre-Flight Checklists

### Publisher — before pushing to GitHub

> [!todo]
> - [ ] `.block/` is in `.gitignore` and not in git history
> - [ ] No `tblXXX` / `fldXXX` IDs in source (grep your code: `grep -RE 'tbl[A-Za-z0-9]{14}|fld[A-Za-z0-9]{14}' frontend/`)
> - [ ] Tables and fields are exposed via `useCustomProperties` OR via `getTableByName` / `getFieldByNameIfExists`
> - [ ] README documents required tables, fields, types, and singleSelect choices
> - [ ] README has a copy-pasteable `block init --template=...` command
> - [ ] README explains the cross-org case: if the forker is in a different workspace, point them to "Cross-Organization Workflow"
> - [ ] LICENSE file exists
> - [ ] `block run` works from a fresh clone of your own repo (test it!)
> - [ ] **Bonus end-to-end test:** create a new base in a different workspace, create a new blockId there, init a fresh clone of your repo against it, confirm it runs without code edits

### Consumer — before opening an issue

> [!todo]
> - [ ] Node 22+ confirmed (`node -v`)
> - [ ] PAT created with `block:manage` scope and access to destination workspace
> - [ ] You can create extensions in the destination workspace (not just edit the base). If not, get the owner to create the extension and add you as a collaborator — see *Cross-Organization Workflow*.
> - [ ] You used `block init <BASE>/<BLOCK> --template=<URL> <DIR>` (not `git clone` alone)
> - [ ] `.block/remote.json` shows YOUR baseId AND YOUR blockId, not the author's
> - [ ] `pwd` confirms you're in the folder you think (`block run` is path-sensitive — agent-spawned sibling folders cause silent misroutes)
> - [ ] Your base has the tables and fields the README requires
> - [ ] Custom properties are configured in the Designer sidebar
> - [ ] You attached the running block via the `</> Develop` button, entered the actual port from terminal output, and accepted the self-signed cert

---

## Related Skills

- **`airtable-extensions`** — the SDK reference. Especially: *Custom Properties for Builders*, *Debug Panel Pattern*, *Array Field Append Pattern*, *FieldType Cell Value Reference*.
- **`obsidian:obsidian-markdown`** — for writing the README using Obsidian-style callouts (matches Rob's vault conventions if you're documenting blocks alongside project notes).

## References in this skill

- `references/README-template.md` — drop-in README for a fork-friendly block repo
- `references/gitignore-template` — canonical `.gitignore`
