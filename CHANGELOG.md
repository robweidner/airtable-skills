# Changelog

All notable changes to the Airtable Skills plugin.

## [1.6.0] - 2026-02-24

### Added

- **CLAUDE.md** — Root-level guidance file for Claude Code instances working in this repo
- **AI field type** (🪄) added to `field-types.md` for cross-file consistency with `emoji-conventions.md`
- **Barcode emoji** (📱) restored to both `field-types.md` and `emoji-conventions.md`

### Changed

- **MCP server package fix** — Corrected non-existent `@airtable/mcp-server` to the real `airtable-mcp-server` npm package; mapped `AIRTABLE_API_KEY` env var correctly *(contributed by Andrew Mitchell)*
- **Emoji system overhaul** — Replaced bracket notation (`[formula]`, `[check]`, `[wip]`, etc.) with actual Unicode emojis that render correctly in Airtable field names *(contributed by Andrew Mitchell)*
- **Extensions SDK rewrite** — Rewrote `airtable-extensions` skill and all three reference docs for the Interface Extensions SDK, replacing deprecated Blocks SDK patterns *(contributed by Andrew Mitchell)*
  - Updated import paths from `@airtable/blocks/ui` to `@airtable/blocks/interface/ui`
  - Replaced `initializeBlock(() => ...)` with `initializeBlock({ interface: () => ... })`
  - Removed Blocks SDK UI components (`Box`, `Button`, `Input`, `TablePicker`, etc.) in favor of HTML + Tailwind CSS
  - Replaced `useGlobalConfig` with `useCustomProperties`
  - Added `useColorScheme` for dark mode support
  - Added `FieldType` enum usage, `expandRecord`, `colorUtils`
  - Added permission checking patterns throughout
  - Documented hooks NOT available in Interface Extensions (`useViewport`, `useCursor`, `useRecordById`)
- **Automation-involved emoji** changed from 🤖 to ⚡ to avoid collision with Formula/Rollup type emoji

### Fixed

- **React Rules of Hooks** — Fixed conditional `useRecords` calls in code examples that violated React hook rules
- **`useColorScheme` return type** — Aligned docs to consistent object return shape across all reference files
- **`useCustomProperties` API shape** — Reconciled three conflicting API signatures to one consistent pattern
- **Undefined function parameters** — Added missing `apiToken` and `baseId` params to fetch examples
- **Null safety** — Added proper field null checks in `useRecords` code examples
- **Missing trailing newline** in `scripting-patterns.md`
- **Automation script `return` limitation** — Documented that `return` outside a function is only supported in Extension scripts, not Automation scripts

## [1.5.0] - 2026-02-08

### Added

- **Interface Extensions deep-dive** - Major expansion of the `airtable-extensions` skill for interface extension development
  - Data source configuration checklist: tables and fields must be explicitly enabled in the interface Data panel
  - Diagnostic component pattern to verify what data your extension can access
  - `block run` crash recovery workflow and common crash scenarios
  - Chrome CORS troubleshooting for localhost dev server
  - Multiple tables workaround using the REST API
  - Omni as alternative entry point for quick prototyping
  - Comprehensive error reference table with causes and fixes
  - Full development checklist (setup → coding → release)
- **New reference doc: `interface-extensions-workflow.md`** - Complete development lifecycle guide for interface extensions

### Changed

- Updated Interface Extensions section from "Alpha SDK" to "Open Beta SDK"
- Added comparison table: Blocks SDK vs Interface Extensions SDK
- Expanded troubleshooting from 2 errors to 10+ common issues

### Fixed

- **Moved `plugin.json` to correct location** — now at `plugins/airtable/.claude-plugin/plugin.json` per the [official plugin spec](https://code.claude.com/docs/en/plugins). Previously at `plugins/airtable/plugin.json` which may have prevented marketplace installation.
- **Rewrote README Quick Start** — step-by-step numbered guide (Install → Token → Env Var → Verify → Preferences) replaces the previous abbreviated setup
- **Fixed skill invocation syntax** — all slash commands now show correct namespaced format (`/airtable:airtable-extensions` instead of `/airtable-extensions`)
- **Added prerequisites section** — Claude Code v1.0.33+, Node.js 18+ for MCP server
- **Added verification step** — how to confirm the plugin is working after installation
- **Added troubleshooting section** — common installation issues and fixes
- **Replaced manual clone instructions** — now uses `--plugin-dir` flag for development/testing
- **Clarified token setup** — explicit instructions for shell profile environment variable instead of ambiguous `.secrets.env`

## [1.4.0] - 2026-02-08

### Added

- **Auto-release on git push** - New Claude Code hook recipe in the `airtable-extensions` skill that automatically runs `block release` after every `git push`, keeping your Airtable extension in sync with your GitHub repo
  - PostToolUse hook detects `git push` commands scoped to your extension repo
  - Skips release if the push failed
  - Step-by-step setup instructions included in the skill

## [1.3.0] - 2026-02-05

### Added

- **New skill: `/airtable-base-audit`** - Comprehensive base health analysis
  - Field Health (~35%): Unused fields, low fill rates, "(copy)" fields, uniqueness analysis
  - Performance (~35%): Record count vs plan limits, TODAY() formula detection, complex formulas
  - Schema Quality (~30%): Repeated data patterns, missing relationships, denormalization
  - Letter grade (A-F) scoring with percentage breakdown
  - Three output modes: apply safe fixes, export report, detailed recommendations
  - Plan-aware limits (Teams/Business/Enterprise)

## [1.2.0] - 2026-02-05

### Added

- **Claude Cowork compatibility** - Plugin now works in both Claude Code CLI and Claude Cowork web
- **Bundled MCP configuration** - `.mcp.json` auto-configures Airtable MCP server on install
- Added `cowork` keyword for marketplace discovery
- Added `homepage` and `repository` fields to plugin manifest

### Documentation

- Updated README with Cowork installation instructions
- Added airtable-field-audit skill to README skills table

## [1.1.0] - 2026-02-05

### Added

- **New skill: `/airtable-field-audit`** - Analyze and clean up field naming issues
  - Detects whitespace (leading/trailing spaces)
  - Finds near-duplicates (typos, hyphen variations)
  - Flags similar/confusing names (Status vs Status 2)
  - Identifies inconsistent casing (firstName vs FirstName)
  - Three remediation modes: auto-fix, review each, export report
  - Respects emoji prefix conventions (won't flag `[email] Email` as duplicate)

### Documentation

- Added brainstorm: `docs/brainstorms/2026-02-05-field-audit-brainstorm.md`
- Added implementation plan: `docs/plans/2026-02-05-feat-airtable-field-audit-skill-plan.md`

## [1.0.0] - 2026-02-04

### Added

- Initial release
- **Skill: `airtable`** - Comprehensive Airtable assistant for schema design, API interactions, scripting, interfaces, and automations
- **Skill: `airtable-extensions`** - Custom React extensions development
- Reference documentation for field types, API patterns, MCP patterns, scripting, automations, and more
