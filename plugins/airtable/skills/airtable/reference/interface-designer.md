# Interface Designer Reference

Guide to building interfaces in Airtable's Interface Designer.

## Views vs Interfaces

**Choose a View when:**
- Filtering/sorting data for yourself or team
- Quick data exploration
- Temporary or ad-hoc needs
- Full table editing capabilities needed

**Choose an Interface when:**
- Presenting data to specific users (stakeholders, clients)
- Creating dashboards with multiple data sources
- Building forms for data entry
- Embedding in websites or apps
- Restricting what users can see/edit

| Feature | Views | Interfaces |
|---------|-------|------------|
| Editing | Full table editing | Controlled per element |
| Layout | Grid only | Custom layouts |
| Multiple tables | One table | Multiple tables |
| Embedding | Limited | Full support |
| User restrictions | By view | By element |
| Mobile | Basic | Responsive |

## Interface Components

### Layout Elements

**Record List**
- Shows records from a table
- Customizable columns
- Can enable inline editing
- Supports filtering, sorting

**Record Detail**
- Shows single record
- Choose which fields to display
- Custom field arrangement
- Edit capabilities optional

**Form**
- Data entry for new records
- Choose fields to include
- Custom labels and help text
- Conditional logic

### Data Visualization

**Summary Bar**
- Count, sum, average, etc.
- Quick metrics at top

**Chart**
- Bar, line, pie, scatter
- Group by field
- Color by field

**Timeline**
- Date-based visualization
- Start/end dates
- Group by field

**Gallery**
- Card-based layout
- Attachment preview
- Custom card fields

### Interactive Elements

**Button**
- Trigger actions
- Link to URLs
- Open other interfaces

**Filter**
- User-controlled filtering
- Dropdown, date picker, etc.

## Building an Interface

### Step 1: Create Interface

1. Click "Interfaces" in sidebar
2. Click "Create interface"
3. Choose a starting template or blank

### Step 2: Add Data Source

1. Click "Add element"
2. Choose Record List, Detail, etc.
3. Select table
4. Configure which records to show

### Step 3: Configure Elements

**For Record List:**
```
Table: [Select]
View: [Optional - use view filters]
Fields: [Choose which to show]
Enable editing: [Yes/No per field]
Pagination: [Records per page]
```

**For Record Detail:**
```
Table: [Select]
Record: [From list selection, URL, fixed]
Layout: [Drag fields to arrange]
Edit mode: [Inline, modal, read-only]
```

### Step 4: Add Interactivity

- Link elements (list selection → detail view)
- Add filters for users
- Set up conditional visibility

### Step 5: Configure Access

1. Click "Share" or access settings
2. Choose who can access
3. Set permission levels

## Layout Patterns

### Dashboard

```
┌──────────────────────────────────────────────┐
│               Summary Bar                     │
│  [Total: 150]  [Active: 120]  [Due: 15]      │
├──────────────┬───────────────────────────────┤
│              │                               │
│   Filters    │        Chart                  │
│              │                               │
│   Status: ▼  │    ┌───────────────────┐     │
│   Owner: ▼   │    │ Bar chart by      │     │
│   Date: ▼    │    │ status/month      │     │
│              │    └───────────────────┘     │
├──────────────┴───────────────────────────────┤
│                                              │
│              Record List                     │
│  [Name]    [Status]    [Due]    [Owner]     │
│  Task 1    Active      Dec 1    John        │
│  Task 2    Pending     Dec 5    Jane        │
│                                              │
└──────────────────────────────────────────────┘
```

### Master-Detail

```
┌────────────────────┬─────────────────────────┐
│                    │                         │
│    Record List     │     Record Detail       │
│                    │                         │
│  > Contact 1       │  Name: Contact 1        │
│    Contact 2       │  Email: c1@example.com  │
│    Contact 3       │  Phone: 555-0100        │
│    Contact 4       │                         │
│                    │  [Edit] [Delete]        │
│                    │                         │
│  [+ Add Contact]   │  --- Activity ---       │
│                    │  Last contacted: Dec 1  │
│                    │                         │
└────────────────────┴─────────────────────────┘
```

### Form + Confirmation

```
┌──────────────────────────────────────────────┐
│                                              │
│              Submit Request                  │
│                                              │
│  Name: [_________________________]           │
│                                              │
│  Email: [_________________________]          │
│                                              │
│  Request Type: [Select...          ▼]       │
│                                              │
│  Description:                                │
│  [                                     ]     │
│  [                                     ]     │
│                                              │
│              [Submit Request]                │
│                                              │
└──────────────────────────────────────────────┘

After submit:
┌──────────────────────────────────────────────┐
│                                              │
│           Request Submitted!                 │
│                                              │
│  Your request #1234 has been received.       │
│                                              │
│  We'll respond within 24 hours.              │
│                                              │
│           [Submit Another]                   │
│                                              │
└──────────────────────────────────────────────┘
```

### Gallery View

```
┌──────────────────────────────────────────────┐
│  Products                     [Search...]     │
├──────────────────────────────────────────────┤
│  ┌─────────┐  ┌─────────┐  ┌─────────┐      │
│  │  [img]  │  │  [img]  │  │  [img]  │      │
│  │         │  │         │  │         │      │
│  │ Prod A  │  │ Prod B  │  │ Prod C  │      │
│  │ $99.00  │  │ $149.00 │  │ $79.00  │      │
│  └─────────┘  └─────────┘  └─────────┘      │
│                                              │
│  ┌─────────┐  ┌─────────┐  ┌─────────┐      │
│  │  [img]  │  │  [img]  │  │  [img]  │      │
│  │         │  │         │  │         │      │
│  │ Prod D  │  │ Prod E  │  │ Prod F  │      │
│  │ $199.00 │  │ $59.00  │  │ $299.00 │      │
│  └─────────┘  └─────────┘  └─────────┘      │
└──────────────────────────────────────────────┘
```

## Permissions & Sharing

### Access Levels

| Level | Can View | Can Edit | Can Configure |
|-------|----------|----------|---------------|
| Viewer | Yes | No | No |
| Commenter | Yes | Comments | No |
| Editor | Yes | Yes | No |
| Creator | Yes | Yes | Yes |

### Sharing Options

**Within Workspace:**
- Share with specific users
- Share with everyone in workspace
- Restrict by group

**External Sharing:**
- Enable public link
- Password protection
- Embed in website

**Embedding:**
```html
<iframe
  class="airtable-embed"
  src="https://airtable.com/embed/[interfaceId]"
  width="100%"
  height="533"
  style="background: transparent; border: 1px solid #ccc;">
</iframe>
```

## Tips & Best Practices

1. **Start simple** - Add elements incrementally, test as you go

2. **Think about mobile** - Preview on mobile, interfaces are responsive

3. **Limit fields** - Don't show every field. Show what's needed.

4. **Use filters wisely** - Let users filter to find what they need

5. **Test with users** - Get feedback before rolling out

6. **Version your interfaces** - Duplicate before major changes

7. **Name elements clearly** - Use descriptive names for elements (helps when linking)

8. **Consider loading time** - Many linked fields or large tables slow things down
