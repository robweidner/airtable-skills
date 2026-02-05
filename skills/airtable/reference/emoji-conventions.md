# Emoji Naming Conventions

A visual system for identifying field types and statuses at a glance.

## Why Use Emojis?

1. **Visual identification** - Instantly see what type of field you're looking at
2. **Signal to others** - Emoji-labeled fields indicate "don't modify without approval"
3. **Error detection** - Easier to spot when a field has the wrong type
4. **Maintenance** - Quickly understand complex bases during maintenance

**Example:** In a base with 50 fields, finding all formula fields is instant when they're prefixed with `[formula]`.

## Emoji Preference Options

Users can set their preference in CLAUDE.md:

```markdown
airtable_emoji_mode: auto | new-only | ask | none
```

| Mode | Behavior |
|------|----------|
| `auto` | Add type emojis to all fields automatically |
| `new-only` | Only add emojis to newly created fields |
| `ask` | Ask before adding emojis each time |
| `none` | Never add emojis |

## Field Type Emojis

Add these at the **start** of field names to indicate type.

| Field Type | Emoji | Example |
|------------|-------|---------|
| Link to another record | [link] | `[link] Company` |
| Single line text | (none) | `Name` (basic, no emoji needed) |
| Long text | [text] | `[text] Description` |
| Attachment | [attach] | `[attach] Documents` |
| Checkbox | [check] | `[check] Completed` |
| Multiple select | [multi] | `[multi] Tags` |
| Single select | [select] | `[select] Status` |
| User | [user] | `[user] Assignee` |
| Date | [date] | `[date] Due Date` |
| Phone number | [phone] | `[phone] Mobile` |
| Email | [email] | `[email] Work Email` |
| URL | [url] | `[url] Website` |
| Number | [num] | `[num] Quantity` |
| Currency | [money] | `[money] Price` |
| Percent | [pct] | `[pct] Progress` |
| Duration | [time] | `[time] Time Spent` |
| Rating | [star] | `[star] Priority` |
| Formula | [formula] | `[formula] Full Name` |
| Rollup | [rollup] | `[rollup] Total Orders` |
| Count | [count] | `[count] Items` |
| Lookup | [lookup] | `[lookup] Customer Email` |
| Created time | [created] | `[created] Created At` |
| Last modified time | [modified] | `[modified] Updated At` |
| Created by | [user] | `[user] Created By` |
| Last modified by | [user] | `[user] Modified By` |
| Autonumber | [auto] | `[auto] ID` |
| Barcode | [barcode] | `[barcode] Product Code` |
| Button | [button] | `[button] Open Link` |

## Purpose/Status Emojis

These indicate the **purpose** or **status** of a field, layered ON TOP OF type emojis.

| Purpose | Emoji | When to Use |
|---------|-------|-------------|
| Automation-involved | [auto-field] | Fields read/written by automations |
| Permissions | [lock] | Access control fields |
| Tested/Working | [tested] | Verified automation is working |
| Work in progress | [wip] | Still being built |
| Emergency/backup | [sos] | Just-in-case workflows, not daily use |
| Archived | [archive] | Should be removed eventually |
| Needs review | [review] | Attention required |
| Helper tool | [helper] | Utility field |
| Trigger | [trigger] | Starts an action (checkbox that triggers automation) |
| Synced | [sync] | Data synced from another source |
| AI-powered | [ai] | AI field |
| External integration | [ext] | Depends on Make/Zapier/external API |
| Convert needed | [convert] | Field needs manual type conversion |

## Emoji Stacking

When multiple emojis apply, **stack them** with status/purpose first, then type.

**Pattern:** `[status][type] Field Name`

### Examples

| Scenario | Field Name |
|----------|------------|
| WIP formula field | `[wip][formula] Sales Commission` |
| Synced lookup | `[sync][lookup] Customer Status` |
| Tested automation trigger | `[tested][trigger][check] Send Welcome Email` |
| External integration number | `[ext][num] Stripe Balance` |
| Field needing conversion | `[convert][formula] Full Name` |

## Conversion Marker

The `[convert]` emoji is special - it indicates a field that was created as a placeholder and needs manual conversion.

**Why?** Airtable's API cannot create formula, rollup, or lookup fields. The workaround is:
1. Create as Single Line Text
2. Mark with `[convert]`
3. Put the formula/config in the field description
4. After API creation, manually convert the field type

**Example workflow:**

API creates:
```
Field: [convert][formula] Full Name
Type: Single Line Text
Description: CONCATENATE({First Name}, " ", {Last Name})
```

User manually:
1. Opens field settings
2. Changes type to Formula
3. Pastes formula from description
4. Removes `[convert]` from name

## Best Practices

### When to Emoji

- **Always:** Formula, Rollup, Lookup, Link fields (they have special behavior)
- **Recommended:** Currency, Date, User fields (clarifies expected data)
- **Optional:** Text fields (basic, often don't need marking)
- **Always:** Fields involved in automations (signals "don't change without checking")

### When NOT to Emoji

- Very simple bases (< 10 fields)
- Temporary/throwaway bases
- When the team prefers clean names

### Naming After the Emoji

Keep names descriptive and consistent:
- `[link] Company` not `[link] Comp` or `[link] company`
- `[date] Due Date` not `[date] Due` or `[date] DueDate`
- Use Title Case after emoji

### Team Agreement

If working with a team, agree on:
1. Which emojis to use (maybe subset of this list)
2. Whether to use emojis at all
3. Stacking order preference

Document in your project's CLAUDE.md or Airtable base documentation.
