# Emoji Naming Conventions

A visual system for identifying field types and statuses at a glance.

## Why Use Emojis?

1. **Visual identification** - Instantly see what type of field you're looking at
2. **Signal to others** - Emoji-labeled fields indicate "don't modify without approval"
3. **Error detection** - Easier to spot when a field has the wrong type
4. **Maintenance** - Quickly understand complex bases during maintenance

**Example:** In a base with 50 fields, finding all formula fields is instant when they're prefixed with 🤖.

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
| Link to another record | 🔗 | `🔗 Company` |
| Single line text | (none) | `Name` (basic, no emoji needed) |
| Long text | 📝 | `📝 Description` |
| Attachment | 📎 | `📎 Documents` |
| Checkbox | ☑️ | `☑️ Completed` |
| Multiple select | 🔢 | `🔢 Tags` |
| Single select | 1️⃣ | `1️⃣ Status` |
| User | 👤 | `👤 Assignee` |
| Date | 📅 | `📅 Due Date` |
| Phone number | 📞 | `📞 Mobile` |
| Email | 📧 | `📧 Work Email` |
| URL | 🌍 | `🌍 Website` |
| Number | #️⃣ | `#️⃣ Quantity` |
| Currency | 💰 | `💰 Price` |
| Percent | 📊 | `📊 Progress` |
| Duration | ⏱️ | `⏱️ Time Spent` |
| Rating | ⭐ | `⭐ Priority` |
| Formula | 🤖 | `🤖 Full Name` |
| Rollup | 🤖 | `🤖 Total Orders` |
| Count | 🧮 | `🧮 Items` |
| Lookup | 🔎 | `🔎 Customer Email` |
| Created time | 📅 | `📅 Created At` |
| Last modified time | 📅 | `📅 Updated At` |
| Created by | 👤 | `👤 Created By` |
| Last modified by | 👤 | `👤 Modified By` |
| Autonumber | 🆔 | `🆔 ID` |
| Barcode | 📱 | `📱 Product Code` |
| Button | 👊 | `👊 Open Link` |
| AI | 🪄 | `🪄 Summary` |

## Purpose/Status Emojis

These indicate the **purpose** or **status** of a field, layered ON TOP OF type emojis.

| Purpose | Emoji | When to Use |
|---------|-------|-------------|
| Automation-involved | ⚡ | Fields read/written by automations |
| Permissions | 🔒 | Access control fields |
| Tested/Working | ✅ | Verified automation is working |
| Work in progress | 👷 | Still being built |
| Emergency/backup | 🆘 | Just-in-case workflows, not daily use |
| Archived | 💀 | Should be removed eventually |
| Needs review | 👀 | Attention required |
| Helper tool | 🛟 | Utility field |
| Trigger | 🚀 | Starts an automation or action |
| Synced | 🔄 | Data synced from another source |
| External integration | 🔌 | Depends on Make/Zapier/external API |
| Convert needed | 🔧 | Field needs manual type conversion |

## Emoji Stacking

When multiple emojis apply, **stack them** with status/purpose first, then type.

**Pattern:** `<status> <type> Field Name`

### Examples

| Scenario | Field Name |
|----------|------------|
| WIP formula field | `👷🤖 Sales Commission` |
| Synced lookup | `🔄🔎 Customer Status` |
| Tested automation trigger | `✅🚀☑️ Send Welcome Email` |
| External integration number | `🔌#️⃣ Stripe Balance` |
| Field needing conversion | `🔧🤖 Full Name` |

## Conversion Marker

The 🔧 emoji is special - it indicates a field that was created as a placeholder and needs manual conversion.

**Why?** Airtable's API cannot create formula, rollup, or lookup fields. The workaround is:
1. Create as Single Line Text
2. Mark with 🔧
3. Put the formula/config in the field description
4. After API creation, manually convert the field type

**Example workflow:**

API creates:
```
Field: 🔧🤖 Full Name
Type: Single Line Text
Description: CONCATENATE({First Name}, " ", {Last Name})
```

User manually:
1. Opens field settings
2. Changes type to Formula
3. Pastes formula from description
4. Removes 🔧 from name

## Best Practices

### When to Emoji

- **Always:** Formula, Rollup, Lookup, Link fields (they have special behavior)
- **Recommended:** Currency, Date, User fields (clarifies expected data)
- **Optional:** Text fields (basic, often don't need marking)
- **Always:** Fields involved in automations (signals "don't change without checking")

### When NOT to Emoji

- Very simple tables (< 10 fields)
- Temporary/throwaway tables
- When the team prefers clean names

### Naming After the Emoji

Keep names descriptive and consistent:
- `🔗 Company` not `🔗 Comp` or `🔗 company`
- `📅 Due Date` not `📅 Due` or `📅 DueDate`
- Use Title Case after emoji

### Team Agreement

If working with a team, agree on:
1. Which emojis to use (maybe subset of this list)
2. Whether to use emojis at all
3. Stacking order preference

Document in your project's CLAUDE.md or Airtable base documentation.
