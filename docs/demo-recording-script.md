# Demo Recording Script: Airtable Skills for Claude Code

> **Target length:** 60-90 seconds (video) or 30-45 seconds (GIF)
> **Format options:** MP4 with voiceover, silent GIF with captions, or both
> **Resolution:** 1920x1080 or 1280x720

---

## Pre-Recording Setup

### Terminal Setup
```bash
# Clean terminal history
clear

# Set a clean prompt (optional)
export PS1="$ "

# Navigate to a demo project folder
cd ~/Github/demo-project

# Make sure Claude Code is ready
claude
```

### Airtable Setup
- Have an empty or demo base ready (or create one called "Demo CRM")
- Know the base ID (starts with `app`)
- Have your PAT configured in `.secrets.env`

### Recording Settings
- **Font size:** 16-18pt in terminal (large enough to read)
- **Window size:** ~120 columns wide
- **Theme:** Dark terminal background (better contrast)
- **Hide:** Bookmarks bar, notifications, personal info

---

## Version A: Full Demo (60-90 seconds)

### Scene 1: The Problem (0:00-0:15)

**ON SCREEN:**
Terminal with Claude Code open

**ACTION:**
Type this prompt:
```
Create a formula field called "Full Name" that concatenates First Name and Last Name
```

**VOICEOVER (or caption):**
> "When you ask Claude to create a formula field in Airtable..."

**Claude responds with code that would fail**

**CAPTION:**
```
❌ This fails — Airtable API can't create formula fields
```

**PAUSE:** 2 seconds on the error explanation

---

### Scene 2: Install the Skill (0:15-0:25)

**ON SCREEN:**
Terminal

**ACTION:**
Type:
```
/plugin marketplace add robweidner/airtable-skills
```

**VOICEOVER (or caption):**
> "Install the Airtable skill in 10 seconds..."

**Show:** Installation success message

**CAPTION:**
```
✓ Skill installed — Claude now knows Airtable inside out
```

---

### Scene 3: The Solution - Formula Workaround (0:25-0:45)

**ON SCREEN:**
Terminal with Claude Code

**ACTION:**
Type the same prompt:
```
Create a formula field called "Full Name" that concatenates First Name and Last Name
```

**VOICEOVER (or caption):**
> "Now Claude knows the limitation and works around it automatically..."

**Show Claude's response:**
```
Created field: [convert]🧮 Full Name (Single line text)

⚠️ Manual conversion needed:
┌─────────────────────────────────────────────────────┐
│ [ ] [convert]🧮 Full Name                           │
│     → Convert to Formula field                      │
│     → Formula: CONCATENATE({First Name}, " ", {Last Name}) │
└─────────────────────────────────────────────────────┘
```

**CAPTION:**
```
✓ Creates as text field with formula in description
✓ Gives you a checklist for manual conversion
```

**PAUSE:** 3 seconds to read the output

---

### Scene 4: Automation Diagrams (0:45-1:05)

**ON SCREEN:**
Terminal

**ACTION:**
Type:
```
Set up an automation to send Slack when a deal closes over $10k
```

**VOICEOVER (or caption):**
> "Automations can't be created via API either. So Claude generates a diagram..."

**Show Claude's response with the ASCII diagram:**
```
AUTOMATION: High-Value Deal Notification
═══════════════════════════════════════

[1. TRIGGER]
   When record matches conditions
   Table: Deals
          │
          ▼
[2. CONDITION]
   Amount > 10000
          │
          ▼
[3. ACTION]
   Send Slack message
   Channel: #sales-wins
```

**CAPTION:**
```
✓ Visual diagram you can follow
✓ Step-by-step setup instructions included
```

---

### Scene 5: Call to Action (1:05-1:15)

**ON SCREEN:**
Terminal or title card

**ACTION:**
Show installation command prominently:
```
/plugin marketplace add robweidner/airtable-skills
```

**VOICEOVER (or caption):**
> "Free and open source. Install in 10 seconds."

**Show:**
- GitHub URL: github.com/robweidner/airtable-skills
- Your name/handle

**END**

---

## Version B: Quick GIF (30-45 seconds)

For Twitter/Reddit/LinkedIn where autoplay GIFs work better.

### Scene 1: Problem → Solution (0:00-0:20)

**Frame 1 (3 sec):**
Caption: "Claude doesn't know Airtable's API limitations"
Show: Failed formula field creation attempt

**Frame 2 (3 sec):**
Caption: "Install the Airtable skill..."
Show: `/plugin marketplace add robweidner/airtable-skills`

**Frame 3 (5 sec):**
Caption: "Now it works around them automatically"
Show: Successful formula field workaround with checklist

**Frame 4 (5 sec):**
Caption: "Can't create automations? Get diagrams instead"
Show: ASCII automation diagram

### Scene 2: CTA (0:20-0:30)

**Frame 5 (5 sec):**
Caption: "Free • Open Source • 10 seconds to install"
Show:
```
/plugin marketplace add robweidner/airtable-skills
github.com/robweidner/airtable-skills
```

**LOOP**

---

## Version C: Feature Showcase (3-4 short clips)

Create separate 15-second clips for each feature. Good for carousels or posting over multiple days.

### Clip 1: Formula Workaround
- Show the problem
- Show the solution
- End with install command

### Clip 2: Automation Diagrams
- Type automation request
- Show ASCII diagram output
- End with install command

### Clip 3: Schema Creation with Emojis
- "Create a Contacts table with name, email, company, status"
- Show the emoji-prefixed fields created
- End with install command

### Clip 4: Script Generation
- "Write a script to find overdue tasks"
- Show working Airtable scripting code
- End with install command

---

## Recording Tips

### For Smooth Recording

1. **Practice the typing** — or use a typing simulator to avoid typos
2. **Pre-write Claude's responses** — you can edit the video to show ideal outputs
3. **Use keyboard shortcuts** — `Ctrl+L` to clear, `Ctrl+C` to cancel
4. **Record in segments** — easier to edit than one long take

### For Better Visuals

1. **Increase font size** — 18pt minimum for readability on mobile
2. **Use a clean terminal theme** — Dracula, One Dark, or similar
3. **Add subtle zoom** — on key output sections in post-production
4. **Add captions** — 80% of social video is watched on mute

### Tools

| Tool | Use |
|------|-----|
| **QuickTime** (Mac) | Simple screen recording |
| **OBS Studio** | More control, free |
| **Loom** | Quick share, auto-captions |
| **Gifski** | Convert video to high-quality GIF |
| **Kap** (Mac) | Direct GIF recording |
| **CleanShot X** | Screenshots + GIFs + annotations |

---

## Text Overlays / Captions

Use these as title cards or lower-thirds:

**Opening:**
```
The Airtable API can't create formula fields.
Claude doesn't know that.
Until now.
```

**Middle:**
```
✓ All 28 field types
✓ API limitation workarounds
✓ Automation diagrams
✓ Working script patterns
```

**Closing:**
```
Free • Open Source • MIT Licensed

/plugin marketplace add robweidner/airtable-skills
```

---

## Sample Voiceover Script (Full Read)

If you're doing a voiced video, here's the full script:

---

*[0:00]*
When you ask Claude to help with Airtable, it doesn't know the API limitations. Like the fact that you can't create formula fields.

*[0:08]*
So it gives you code that fails.

*[0:12]*
I built a skill that fixes this. Install it in ten seconds.

*[0:18]*
Now when you ask for a formula field, Claude knows to create it as a text field, stores the formula in the description, and gives you a checklist for what to convert manually.

*[0:28]*
Same with automations. The API can't create them. So Claude generates a visual diagram and step-by-step setup instructions.

*[0:38]*
It knows all 28 Airtable field types, the scripting API, rate limits, and every workaround I've learned over the years.

*[0:48]*
It's free and open source. Link in description.

---

## Thumbnail Options

If posting to YouTube or anywhere that shows thumbnails:

**Option 1: Problem/Solution Split**
```
Left side: ❌ "API Error: Cannot create formula field"
Right side: ✓ Formula workaround output
```

**Option 2: Bold Text**
```
"I Taught Claude
Every Airtable
Limitation"
[Claude logo + Airtable logo]
```

**Option 3: Terminal Screenshot**
```
Clean terminal showing the automation diagram output
Text overlay: "Claude + Airtable"
```

---

## Post-Production Checklist

- [ ] Trim dead air / pauses
- [ ] Add captions (burned in for social, .srt for YouTube)
- [ ] Add subtle background music (optional, keep it low)
- [ ] Add zoom effects on key outputs
- [ ] Export at 1080p for video, 720p for GIF
- [ ] Create square crop version for Instagram/LinkedIn
- [ ] Create vertical crop for TikTok/Reels/Shorts (if using)

---

## File Naming

```
airtable-skills-demo-full-1080p.mp4
airtable-skills-demo-quick.gif
airtable-skills-clip-formula.mp4
airtable-skills-clip-automation.mp4
airtable-skills-thumbnail.png
```
