# Blog Post Draft: Airtable Skills for Claude Code Launch

> **Target URL:** robweidner.com/blog/airtable-claude-code-skill
> **Target length:** 1,500-2,000 words
> **Primary keyword:** "Airtable Claude Code skill"
> **Secondary keywords:** "Airtable AI assistant", "Airtable scripting", "Claude Code plugins"

---

# I Taught Claude Every Airtable API Limitation (And Built a Free Skill to Share It)

If you've ever asked Claude to help with Airtable, you've probably hit this wall:

```
Claude: "Here's the code to create a formula field..."
You: *tries it*
Airtable API: "Cannot create formula fields via API"
You: 😐
```

The Airtable API has quirks. Lots of them. And unless you explicitly tell Claude about each one, it'll confidently generate code that doesn't work.

So I built a Claude Code skill that teaches Claude everything I know about Airtable—the good, the bad, and the "why can't I create a formula field via API?"

It's free, open source, and takes 10 seconds to install.

---

## The Problem: Claude Doesn't Know What Airtable Can't Do

Claude is brilliant at code. But it doesn't know that:

- **You can't create formula fields via API** (or rollups, or lookups)
- **You can't create automations via API** (at all)
- **You can't create bases via API** (you have to make them in the UI first)
- **Field names are case-sensitive** (so `Status` and `status` are different fields)
- **Rate limits are strict** (5 requests/second, 10 records per batch)

Every time I asked Claude for Airtable help, I'd get code that looked perfect—then failed on these limitations. I'd have to explain the workaround, Claude would apologize, and we'd start over.

After the tenth time explaining the formula field workaround, I thought: *What if Claude just... knew this already?*

---

## The Solution: A Skill That Knows Airtable Inside Out

Claude Code has a feature called **skills**—essentially instruction sets you can install that give Claude domain-specific knowledge.

I built one for Airtable. It knows:

| Capability | What It Knows |
|------------|---------------|
| **All 28 field types** | Exact API parameters, options, and gotchas |
| **API limitations** | What can't be done + automatic workarounds |
| **Scripting patterns** | Airtable's scripting SDK with working examples |
| **Automation design** | ASCII diagrams since the API can't create them |
| **Interface guidance** | Interface Designer patterns and best practices |
| **Emoji conventions** | Visual field naming system (optional) |

When you ask Claude to "create a formula field," it doesn't fail anymore—it creates a text field with a `[convert]` marker and puts the formula in the description, then gives you a checklist of what to convert manually.

---

## Installation (10 Seconds)

If you have Claude Code installed:

```bash
/plugin marketplace add robweidner/airtable-skills
```

That's it. The skill auto-activates whenever you're working with Airtable.

For manual installation:
```bash
cd ~/.claude/plugins
git clone https://github.com/robweidner/airtable-skills.git airtable
```

---

## What It Actually Does (With Examples)

### Example 1: Creating a Schema

**You say:**
> "Create a Contacts table with name, email, company, and status fields"

**Without the skill**, Claude might create fields without considering:
- Whether you want audit fields (Created Time, Last Modified By)
- How to handle the status field options
- Field naming conventions

**With the skill**, Claude:
1. Suggests standard audit fields (you can decline)
2. Asks about your emoji preference for visual field identification
3. Creates all fields with proper API parameters
4. Outputs a summary of what was created

```
Created fields in "Contacts":
├── 📝 Name (Single line text)
├── 📧 Email (Email)
├── 🔗 Company (Link to Companies)
├── 📊 Status (Single select: New, Contacted, Qualified, Closed)
├── 📅 Created Time (Created time)
└── 👤 Last Modified By (Last modified by)
```

### Example 2: The Formula Field Workaround

**You say:**
> "Add a Full Name field that concatenates First Name and Last Name"

**Without the skill**, Claude tries to create a formula field via API. It fails.

**With the skill**, Claude knows the limitation and works around it:

```
Created field: [convert]🧮 Full Name (Single line text)

⚠️ Manual conversion needed:
┌─────────────────────────────────────────────────────┐
│ [ ] [convert]🧮 Full Name                           │
│     → Convert to Formula field                      │
│     → Formula: CONCATENATE({First Name}, " ", {Last Name}) │
└─────────────────────────────────────────────────────┘

The formula is stored in the field description for reference.
```

### Example 3: Automation Diagrams

**You say:**
> "Set up an automation that sends a Slack message when a deal closes over $10,000"

**Without the skill**, Claude might try to create this via API (impossible) or give you vague instructions.

**With the skill**, Claude generates a visual diagram and step-by-step setup:

```
AUTOMATION: High-Value Deal Notification
═══════════════════════════════════════

[1. TRIGGER]
   Type: When record matches conditions
   Table: Deals
   Condition: Status = "Closed Won"
          │
          ▼
[2. CONDITION]
   Amount > 10000
          │
          ▼
[3. ACTION]
   Send Slack message
   Channel: #sales-wins
   Message: "🎉 {Deal Name} closed for ${Amount}!"

────────────────────────────────────────
SETUP STEPS:
1. Go to Automations tab in your base
2. Click "+ Create automation"
3. Name it: "High-Value Deal Notification"
4. Set trigger: "When record matches conditions"
5. Select table: Deals
6. Add condition: Status = "Closed Won"
7. Add another condition: Amount > 10000
8. Add action: "Send a Slack message"
9. Configure channel and message as shown
10. Turn on automation
```

### Example 4: Scripts That Actually Work

**You say:**
> "Write a script to find all overdue tasks and update their status to Urgent"

**With the skill**, Claude generates working Airtable scripting code:

```javascript
// Find overdue tasks and mark as Urgent
let tasksTable = base.getTable("Tasks");
let today = new Date();
today.setHours(0, 0, 0, 0);

let query = await tasksTable.selectRecordsAsync({
    fields: ["Name", "Due Date", "Status"]
});

let overdueRecords = query.records.filter(record => {
    let dueDate = record.getCellValue("Due Date");
    if (!dueDate) return false;
    return new Date(dueDate) < today;
});

output.markdown(`Found **${overdueRecords.length}** overdue tasks`);

for (let record of overdueRecords) {
    await tasksTable.updateRecordAsync(record.id, {
        "Status": { name: "Urgent" }
    });
    output.text(`Updated: ${record.getCellValue("Name")}`);
}

output.markdown("✅ All overdue tasks marked as Urgent");
```

The skill adapts the code style based on your experience level—beginners get more comments and explanations, developers get terse, pattern-focused code.

---

## Preferences You Can Set

The skill asks for your preferences on first run, or you can add them to your `CLAUDE.md`:

```markdown
## Airtable Preferences
airtable_experience: developer
airtable_emoji_mode: auto
airtable_script_style: minimal
airtable_use_field_ids: true
```

| Preference | Options | What It Does |
|------------|---------|--------------|
| `experience` | beginner / power-user / developer | Controls explanation depth |
| `emoji_mode` | auto / new-only / ask / none | Field name emoji prefixes |
| `script_style` | minimal / comprehensive | Output-focused vs. all-in-one scripts |
| `use_field_ids` | true / false | Store field IDs in descriptions |

### Why Field IDs Matter

Field names change. Field IDs don't.

If you enable `use_field_ids`, the skill stores field IDs in field descriptions after creation:

```
Description: Customer's primary contact email
Field ID: fldABC123XYZ789
```

Then scripts use IDs instead of names—so your code survives field renames.

---

## The Emoji Naming Convention

This is optional, but I find it useful. The skill can prefix field names with type emojis for visual scanning:

```
📝 Name           → text field
📧 Email          → email field
🔢 Quantity       → number field
💰 Total          → currency field
📅 Due Date       → date field
🔗 Company        → linked record
📊 Status         → single select
🧮 Full Name      → formula
```

When you're scanning a table with 30+ fields, these help you instantly identify field types.

---

## What's NOT Possible (Even With This Skill)

Let's be clear about limitations:

| Can't Do | Why | Workaround |
|----------|-----|------------|
| Create bases | API limitation | Create in UI, give Claude the base ID |
| Create automations | API limitation | Skill generates diagrams + setup steps |
| Create formula/rollup/lookup fields | API limitation | Creates as text with conversion checklist |
| Modify field types | API limitation | Delete and recreate |

The skill doesn't magically bypass Airtable's API—it just knows the limitations and works around them intelligently.

---

## Why I Built This

I'm a power user who got tired of being Claude's Airtable tutor.

Every session started the same way:
1. Ask Claude for help
2. Get code that doesn't work
3. Explain why it doesn't work
4. Get better code
5. Repeat for the next limitation

Skills solve this by front-loading the knowledge. Now Claude knows what I know, and we can skip straight to productive work.

If you work with Airtable regularly and use Claude Code, this should save you hours of back-and-forth.

---

## Get Started

**Install:**
```bash
/plugin marketplace add robweidner/airtable-skills
```

**GitHub:** [github.com/robweidner/airtable-skills](https://github.com/robweidner/airtable-skills)

**Contribute:** It's MIT licensed. PRs welcome for new scripting patterns, automation templates, or documentation updates.

---

## Questions?

Drop a comment below or open an issue on GitHub. I'd especially love to hear:
- What Airtable tasks do you wish Claude handled better?
- Are there patterns or workflows I should add?

---

*Rob Weidner builds tools at the intersection of no-code and AI. Follow along at [robweidner.com](https://robweidner.com).*

---

## SEO Metadata

**Title tag:** I Taught Claude Every Airtable API Limitation | Free Skill
**Meta description:** A free Claude Code skill that turns Claude into an Airtable expert. Handles API limitations, generates working scripts, and creates automation diagrams.
**URL slug:** /blog/airtable-claude-code-skill

**Schema markup suggestions:**
- HowTo schema for installation steps
- SoftwareApplication schema for the skill itself

---

## Social Sharing Snippets

**Twitter/X:**
```
I taught Claude every Airtable API limitation.

It now knows you can't create formula fields (and uses a workaround), can't create automations (and generates diagrams instead), and all 28 field types.

Free skill: /plugin marketplace add robweidner/airtable-skills
```

**LinkedIn:**
```
I just open-sourced an AI assistant for Airtable.

The Airtable API can't create formula fields. It can't create automations. It can't even create bases.

So I built a Claude Code skill that knows all the limitations—and works around them automatically.

Full blog post: [link]
```

**Reddit (r/Airtable):**
```
Title: I built an open-source AI skill that turns Claude into an Airtable expert

[First paragraph of blog post, then link]
```
