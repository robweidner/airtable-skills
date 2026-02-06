# Airtable Automations Reference

Guide to setting up Airtable automations, including ASCII diagram templates.

## Important Limitation

**Automations cannot be created via API or MCP.** They must be set up manually in the Airtable UI.

This skill provides:
1. ASCII diagrams showing the automation flow
2. Step-by-step setup instructions
3. Script templates for script blocks

## Automation Components

### Triggers
What starts the automation.

| Trigger | When it fires |
|---------|---------------|
| When record created | New record added to table |
| When record updated | Field value changes |
| When record matches conditions | Record enters a filtered view |
| When record enters view | Record appears in a view |
| When form submitted | Form response received |
| At scheduled time | Cron schedule (daily, weekly, etc.) |
| When webhook received | External HTTP POST |
| When button clicked | Manual trigger via button field |

### Conditions
Filter which records trigger actions.

### Actions
What happens when triggered.

| Action | What it does |
|--------|--------------|
| Update record | Change field values |
| Create record | Add new record |
| Send email | Send via Airtable |
| Send Slack message | Post to channel |
| Run script | Execute JavaScript |
| Call webhook | HTTP request to URL |
| Find records | Query for records |
| Repeat for each | Loop over records |

### Advanced Actions (Embedded)
- **Conditional logic** - If/then/else branches
- **Repeat step** - Loop (only ONE per automation)

## ASCII Diagram Templates

### Template 1: Simple Record Update

```
============================================================
AUTOMATION: [Name]
============================================================

┌─────────────────────────────────────────────┐
│ TRIGGER: When record updated                │
│ Table: [Table Name]                         │
│ Watch field: [Field Name]                   │
└─────────────────────┬───────────────────────┘
                      │
                      ▼
┌─────────────────────────────────────────────┐
│ ACTION: Update record                       │
│ Record: Trigger record                      │
│ Fields:                                     │
│   [Field] = [Value]                         │
└─────────────────────────────────────────────┘
```

### Template 2: With Condition

```
============================================================
AUTOMATION: [Name]
============================================================

┌─────────────────────────────────────────────┐
│ TRIGGER: When record created                │
│ Table: [Table Name]                         │
└─────────────────────┬───────────────────────┘
                      │
                      ▼
┌─────────────────────────────────────────────┐
│ CONDITION                                   │
│ Only continue if:                           │
│   [Field] is not empty                      │
│   AND [Field] = [Value]                     │
└─────────────────────┬───────────────────────┘
                      │
                      ▼
┌─────────────────────────────────────────────┐
│ ACTION: [Action Type]                       │
│ [Action details...]                         │
└─────────────────────────────────────────────┘
```

### Template 3: Send Notification

```
============================================================
AUTOMATION: Welcome Email
============================================================

┌─────────────────────────────────────────────┐
│ TRIGGER: When record created                │
│ Table: Contacts                             │
└─────────────────────┬───────────────────────┘
                      │
                      ▼
┌─────────────────────────────────────────────┐
│ CONDITION                                   │
│ Email is not empty                          │
└─────────────────────┬───────────────────────┘
                      │
                      ▼
┌─────────────────────────────────────────────┐
│ ACTION: Send email                          │
│ To: {Email}                                 │
│ Subject: Welcome, {Name}!                   │
│ Body:                                       │
│   Hi {Name},                                │
│                                             │
│   Welcome to our platform...                │
└─────────────────────────────────────────────┘
```

### Template 4: With Script Block

```
============================================================
AUTOMATION: Calculate Commission
============================================================

┌─────────────────────────────────────────────┐
│ TRIGGER: When record updated                │
│ Table: Sales                                │
│ Watch: Amount                               │
└─────────────────────┬───────────────────────┘
                      │
                      ▼
┌─────────────────────────────────────────────┐
│ ACTION: Run script                          │
│ Input: recordId = Trigger record ID         │
│                                             │
│ ┌─────────────────────────────────────────┐ │
│ │ let config = input.config();            │ │
│ │ let table = base.getTable("Sales");     │ │
│ │ let record = await table               │ │
│ │   .selectRecordAsync(config.recordId);  │ │
│ │ let amount = record                     │ │
│ │   .getCellValue("Amount");              │ │
│ │ let commission = amount * 0.1;          │ │
│ │ output.set("commission", commission);   │ │
│ └─────────────────────────────────────────┘ │
│ Output: commission                          │
└─────────────────────┬───────────────────────┘
                      │
                      ▼
┌─────────────────────────────────────────────┐
│ ACTION: Update record                       │
│ Record: Trigger record                      │
│ Fields:                                     │
│   Commission = {commission from script}     │
└─────────────────────────────────────────────┘
```

### Template 5: Conditional Branching

```
============================================================
AUTOMATION: Route by Priority
============================================================

┌─────────────────────────────────────────────┐
│ TRIGGER: When record created                │
│ Table: Tickets                              │
└─────────────────────┬───────────────────────┘
                      │
                      ▼
┌─────────────────────────────────────────────┐
│ CONDITIONAL                                 │
│ If Priority = "High"                        │
├─────────────────────┬───────────────────────┤
│         YES         │          NO           │
└──────────┬──────────┴──────────┬────────────┘
           │                     │
           ▼                     ▼
┌─────────────────────┐ ┌─────────────────────┐
│ Send Slack          │ │ Send email          │
│ Channel: #urgent    │ │ To: support@...     │
│ Message: New high   │ │ Subject: New ticket │
│ priority ticket!    │ │                     │
└─────────────────────┘ └─────────────────────┘
```

### Template 6: Repeat Loop

```
============================================================
AUTOMATION: Weekly Digest
============================================================

┌─────────────────────────────────────────────┐
│ TRIGGER: At scheduled time                  │
│ Every Monday at 9:00 AM                     │
└─────────────────────┬───────────────────────┘
                      │
                      ▼
┌─────────────────────────────────────────────┐
│ ACTION: Find records                        │
│ Table: Tasks                                │
│ Condition: Due Date is within past 7 days   │
│            AND Status != Done               │
└─────────────────────┬───────────────────────┘
                      │
                      ▼
┌─────────────────────────────────────────────┐
│ REPEAT: For each record found               │
│ ┌─────────────────────────────────────────┐ │
│ │ ACTION: Send email                      │ │
│ │ To: {Assignee Email}                    │ │
│ │ Subject: Overdue: {Task Name}           │ │
│ │ Body: This task is overdue...           │ │
│ └─────────────────────────────────────────┘ │
│ (Limit: 1 repeat step per automation)       │
└─────────────────────────────────────────────┘
```

## Step-by-Step Setup

### Creating an Automation

1. **Open Automations**
   - Click "Automations" in the top-left menu
   - Or press `Cmd/Ctrl + Shift + A`

2. **Create New**
   - Click "+ Create automation"
   - Name it descriptively

3. **Add Trigger**
   - Click "Add trigger"
   - Select trigger type
   - Configure trigger options
   - Test trigger to verify

4. **Add Conditions (Optional)**
   - Click "Add condition" below trigger
   - Set field conditions
   - Use AND/OR logic

5. **Add Actions**
   - Click "Add action"
   - Select action type
   - Configure with dynamic values from trigger

6. **Test**
   - Click "Test" on each step
   - Click "Test automation" for full run
   - Check run history for errors

7. **Turn On**
   - Toggle automation ON
   - Confirm "Turn on"

### Dynamic Values

Use curly braces to insert field values:
- `{Field Name}` - Value from trigger record
- `{Step Name.fieldName}` - Value from previous step
- For script outputs: `{Script Name.outputName}`

## Limitations

| Limitation | Workaround |
|------------|------------|
| Only 1 repeat step per automation | Use script to loop internally |
| Can't call external APIs directly | Use "Call webhook" or script |
| No complex date math | Use script for calculations |
| Limited nested conditions | Use script for complex logic |

## Common Gotchas

1. **Trigger loops** - Automation A updates record → triggers Automation B → which updates same record → triggers Automation A. Add conditions to prevent.

2. **Test vs Live** - Test mode uses sample data. Always test with real records before turning on.

3. **Rate limits** - Automations can trigger rapidly. Add delays or conditions to prevent overwhelming external services.

4. **Field changes only** - "When record updated" only fires when the WATCHED field changes, not any field.

5. **Script timeouts** - Scripts have a 30-second timeout. For long operations, break into multiple automations.
