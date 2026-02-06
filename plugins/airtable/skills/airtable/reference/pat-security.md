# Personal Access Token (PAT) Security Guide

Best practices for creating and managing Airtable Personal Access Tokens.

## Why Scoped Tokens Matter

**Create a separate PAT for each project.** This ensures:

1. **Isolation** - Claude can only access bases for the current project
2. **Damage control** - If a token is compromised, only one project is affected
3. **Auditability** - Know which token accessed what
4. **Easy revocation** - Revoke one project's access without affecting others

**Anti-pattern:** One "master" PAT with access to all bases. Don't do this.

## Creating a Scoped PAT

### Step 1: Go to Token Settings

Visit [airtable.com/create/tokens](https://airtable.com/create/tokens)

Or: Account menu → Developer hub → Personal access tokens

### Step 2: Create New Token

Click "Create new token"

### Step 3: Name It Descriptively

Use a clear name that identifies:
- Project name
- Purpose
- Who/what uses it

**Examples:**
- `CRM Project - Claude Code`
- `Inventory App - Production`
- `HR System - Read Only`

### Step 4: Select Bases (IMPORTANT)

**Add only the base(s) needed for this project.**

Click "Add a base" and select from your workspace.

| Selection | Risk Level | Use Case |
|-----------|------------|----------|
| Single base | Low | Single-purpose project |
| Few specific bases | Medium | Related bases in one project |
| All bases | HIGH | Almost never appropriate |

### Step 5: Add Scopes

Select minimum required permissions:

| Scope | Permission | When Needed |
|-------|------------|-------------|
| `data.records:read` | Read records | Always |
| `data.records:write` | Create/update/delete records | If modifying data |
| `schema.bases:read` | Read table/field structure | For schema operations |
| `schema.bases:write` | Create/modify tables/fields | If building schema |
| `webhook:manage` | Manage webhooks | If setting up webhooks |

**Recommended combinations:**

**Read-only access:**
- `data.records:read`
- `schema.bases:read`

**Full CRUD:**
- `data.records:read`
- `data.records:write`
- `schema.bases:read`

**Schema management:**
- `data.records:read`
- `data.records:write`
- `schema.bases:read`
- `schema.bases:write`

### Step 6: Create and Copy

1. Click "Create token"
2. **Copy the token immediately** - it won't be shown again
3. Store it securely (see below)

## Storing Tokens Safely

### Option 1: Project .secrets.env (Recommended)

Create a `.secrets.env` file in your project root:

```bash
# .secrets.env
AIRTABLE_PAT=patXXXXXXXXXXXXXX
AIRTABLE_BASE_ID=appXXXXXXXXXXXXXX
```

**Add to .gitignore:**
```
.secrets.env
```

**Load in scripts:**
```bash
source .secrets.env
```

### Option 2: Environment Variables

Set in your shell profile (`~/.zshrc`, `~/.bashrc`):

```bash
export AIRTABLE_PAT_PROJECTNAME=patXXXXXXXXXXXXXX
```

Use project-specific variable names to avoid conflicts.

### Option 3: Secret Manager

For production use, consider:
- macOS Keychain
- 1Password CLI
- AWS Secrets Manager
- HashiCorp Vault

### Never Do

- Commit tokens to git (even in private repos)
- Put tokens in CLAUDE.md or config files
- Share tokens via Slack/email
- Use tokens in client-side code

## Token Lifecycle

### Rotation

Rotate tokens periodically:
1. Create new token with same scopes
2. Update all places using old token
3. Test with new token
4. Delete old token

**Recommended rotation:** Every 90 days, or after team member leaves.

### Revocation

Revoke immediately if:
- Token may be compromised
- Project is complete
- Team member with access leaves
- You see unexpected API activity

**How to revoke:**
1. Go to token settings
2. Find the token
3. Click "Delete"
4. Confirm deletion

### Monitoring

Check your tokens periodically:
- Last used date
- Which bases are accessible
- Scope permissions

Remove unused tokens.

## Troubleshooting

### 401 Unauthorized

Token is invalid or expired.
- Check token was copied correctly (starts with `pat`)
- Verify token exists in your account
- Create new token if needed

### 403 Forbidden

Token lacks required permissions.
- Check scopes in token settings
- Add missing scopes
- Or create new token with correct scopes

### 404 Not Found

Base/table not accessible with this token.
- Verify base is added to token
- Check base wasn't deleted or renamed
- Confirm you have workspace access

## MCP Configuration

If using the Airtable MCP, configure the token:

```json
{
  "mcpServers": {
    "airtable": {
      "command": "npx",
      "args": ["@anthropic/airtable-mcp"],
      "env": {
        "AIRTABLE_API_KEY": "patXXXXXXXXXXXXXX"
      }
    }
  }
}
```

**Note:** MCP config files should also be gitignored.

## Checklist for New Projects

- [ ] Create new PAT (don't reuse existing)
- [ ] Name it clearly (project + purpose)
- [ ] Add only needed bases
- [ ] Add minimum scopes
- [ ] Store in .secrets.env
- [ ] Add .secrets.env to .gitignore
- [ ] Test token works
- [ ] Document in project README (without the actual token)
