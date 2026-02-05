# Airtable Skills Launch Plan

> **Product:** Airtable Skills for Claude Code
> **Author:** Rob Weidner
> **Created:** 2026-02-05
> **Status:** Ready to implement

---

## Executive Summary

Launch the Airtable Skills plugin across four platforms where Rob has presence: Airtable Community, Reddit, LinkedIn, and robweidner.com. The strategy emphasizes demo-first content (video/GIF showing the skill in action) and leverages Rob's existing reputation in the Airtable community.

**Goal:** 200 GitHub stars and 500 installs in the first month.

---

## Assets to Create

| Asset | File | Status | Priority |
|-------|------|--------|----------|
| Blog post | `docs/blog-post-draft.md` | ✅ Drafted | HIGH |
| Recording script | `docs/demo-recording-script.md` | ✅ Drafted | HIGH |
| Demo GIF (30 sec) | — | ⬜ To record | HIGH |
| Demo video (60-90 sec) | — | ⬜ To record | MEDIUM |
| Screenshots (3-4) | — | ⬜ To capture | HIGH |

### Screenshots Needed

1. **Schema creation** — Shows emoji-prefixed fields created in terminal
2. **Formula workaround** — Shows `[convert]` marker and checklist
3. **Automation diagram** — Shows ASCII diagram output
4. **Script output** — Shows working Airtable script code

---

## Launch Sequence

### Phase 1: Preparation (Before Launch Day)

- [ ] Record demo GIF (30 seconds) following `docs/demo-recording-script.md`
- [ ] Capture 3-4 screenshots for blog post
- [ ] Finalize blog post at `docs/blog-post-draft.md`
- [ ] Publish blog post to robweidner.com
- [ ] Verify GitHub repo has proper topics: `airtable`, `claude-code`, `claude-skills`, `no-code`, `ai-tools`

### Phase 2: Launch Day

| Time | Action | Platform |
|------|--------|----------|
| Morning | Publish blog post | robweidner.com |
| Morning | Submit to skill directories (see below) | Various |
| 8-10 AM | Post LinkedIn announcement | LinkedIn |
| Afternoon | Post to r/Airtable | Reddit |
| Evening | Post to r/ClaudeAI | Reddit |

### Phase 3: Days 2-7

- [ ] Monitor and respond to all comments within 2 hours
- [ ] Airtable Community comment marketing (find 3-5 relevant threads)
- [ ] Post dedicated announcement in Airtable Community "Show and Tell"
- [ ] Share blog post link in relevant LinkedIn comments

### Phase 4: Week 2

- [ ] Follow-up LinkedIn post with user feedback/testimonials
- [ ] Cross-post to r/nocode if initial posts performed well
- [ ] Consider Hacker News "Show HN" if momentum is strong

---

## Platform-Specific Content

### LinkedIn Post

```
I just open-sourced an AI assistant for Airtable.

Here's why I built it:

The Airtable API can't create formula fields.
It can't create automations.
It can't even create bases.

So every time I asked Claude for help, I'd get code that didn't work — because it didn't know these limitations.

This skill teaches Claude:
→ What the API can and can't do
→ Workarounds for each limitation
→ All 28 field types with exact parameters
→ Scripting patterns that actually work
→ How to diagram automations since you can't create them

Installation (10 seconds):
/plugin marketplace add robweidner/airtable-skills

It's MIT licensed. Link in comments.

[Attach: demo GIF]

#Airtable #ClaudeCode #NoCode #AI
```

**Posting tips:**
- Post between 8-10 AM Tuesday-Thursday
- Add blog link as first comment (not in post body)
- Engage with every comment in the first hour

---

### Reddit: r/Airtable

```
Title: I built an open-source AI skill that turns Claude into an Airtable expert

Hey everyone, I've been using Airtable for years and got tired of:
- Looking up field type parameters every time
- Writing the same script patterns over and over
- Explaining automation setups that can't be created via API

So I built an open-source Claude Code skill that knows:
✓ All 28 field types and their API parameters
✓ Scripting API patterns with experience-level adaptation
✓ Automation diagrams (since the API can't create them)
✓ Interface Designer guidance
✓ The emoji naming convention for visual field identification

Installation:
/plugin marketplace add robweidner/airtable-skills

It's free and MIT licensed. Would love feedback from fellow power users.

GitHub: https://github.com/robweidner/airtable-skills
Blog post with examples: [robweidner.com link]

[Attach: demo GIF if subreddit allows]
```

---

### Reddit: r/ClaudeAI

```
Title: Made a skill that gives Claude Airtable expertise — handles API limitations automatically

If you work with Airtable, this might be useful. The skill:
- Auto-detects when you're working with Airtable
- Knows API limitations (can't create formula fields) and works around them
- Generates automation diagrams since you can't create them via API
- Adapts explanation depth based on your experience level

The interesting technical bit: it uses a marker system to flag fields that need manual conversion, then outputs a checklist.

Free, open source: https://github.com/robweidner/airtable-skills

Install: /plugin marketplace add robweidner/airtable-skills
```

---

### Airtable Community Post

```
🎉 New open-source tool: Claude Code skill for Airtable

If you use Claude Code (Anthropic's CLI tool), this skill turns it into an Airtable expert:

• Knows all 28 field types and their API parameters
• Handles the "can't create formula fields" problem automatically
• Generates automation diagrams (since the API can't create them)
• Adds emoji naming conventions for visual field identification
• Adapts to your experience level (beginner → developer)

Install:
/plugin marketplace add robweidner/airtable-skills

GitHub: https://github.com/robweidner/airtable-skills

Would love feedback from the community! What Airtable tasks would you want Claude to handle better?
```

---

### Twitter/X (Optional)

```
I taught Claude every Airtable API limitation.

It now knows you can't create formula fields (and uses a workaround), can't create automations (and generates diagrams instead), and all 28 field types.

Free skill: /plugin marketplace add robweidner/airtable-skills

[Attach: demo GIF]
```

---

## Directory Submissions

Submit to these directories on launch day:

| Directory | URL | Notes |
|-----------|-----|-------|
| skills.sh | https://skills.sh | Vercel's directory, highest traffic |
| SkillsMP | https://skillsmp.com | Large marketplace |
| Claude Code Plugins Hub | https://claudecodeplugins.io | Searchable directory |
| claude-plugins.dev | https://claude-plugins.dev | Community registry |
| awesome-claude-skills | https://github.com/travisvn/awesome-claude-skills | Submit PR to add |

### Submission Description (use across all directories)

```
Name: Airtable Skills
Author: Rob Weidner
GitHub: https://github.com/robweidner/airtable-skills

Description:
Turn Claude into an Airtable expert. Handles schema design, scripting, automations, interfaces, and API integrations. Knows all 28 field types, API limitations (with automatic workarounds), and generates automation diagrams for workflows that can't be created via API.

Features:
- All 28 field types with exact API parameters
- Formula field workaround (creates as text with conversion checklist)
- Automation ASCII diagrams with setup instructions
- Airtable scripting SDK patterns
- Interface Designer guidance
- Experience-level adaptation (beginner → developer)
- Optional emoji field naming conventions

Install:
/plugin marketplace add robweidner/airtable-skills
```

---

## Airtable Community Comment Marketing

Find and helpfully respond to threads about:

**Search terms:**
- "scripting help"
- "automation setup"
- "API limitations"
- "formula field"
- "Claude" or "AI"

**Response template:**
```
[Genuinely helpful answer to their question]

I actually built a Claude Code skill that handles this exact pattern — it knows the Airtable scripting API and generates working code. Free and open source if you want to try it: [GitHub link]
```

**Rules:**
- Provide real value first, mention tool second
- Don't spam — max 3-5 comments in week 1
- Only comment where the skill is genuinely relevant

---

## Success Metrics

| Metric | Week 1 Target | Month 1 Target |
|--------|---------------|----------------|
| GitHub stars | 50 | 200 |
| Installs | 100 | 500 |
| Reddit upvotes | 50 total | — |
| LinkedIn likes | 50 | 100+ |
| Blog views | 500 | 2,000 |

### How to Track

- **GitHub stars:** Check repo directly
- **Installs:** Not directly trackable; estimate from stars + feedback
- **Reddit:** Check post scores
- **LinkedIn:** Post analytics
- **Blog:** Google Analytics or hosting analytics

---

## Follow-Up Content Ideas (Week 2+)

1. **"What I learned launching an AI skill"** — LinkedIn post with metrics
2. **User feedback compilation** — If you get good quotes, share them
3. **Tutorial video** — Longer walkthrough for YouTube
4. **Feature updates** — New patterns or templates added

---

## Files Reference

```
docs/
├── LAUNCH-PLAN.md          ← This file
├── blog-post-draft.md      ← Full blog post ready to publish
└── demo-recording-script.md ← Recording guide for GIF/video
```

---

## Quick Start Checklist

Copy this to your task manager:

```
AIRTABLE SKILLS LAUNCH

Prep:
[ ] Record 30-second demo GIF
[ ] Capture 3-4 screenshots
[ ] Publish blog post to robweidner.com
[ ] Add GitHub topics (airtable, claude-code, etc.)

Launch Day:
[ ] Submit to skills.sh
[ ] Submit to SkillsMP
[ ] Submit to claudecodeplugins.io
[ ] Submit to claude-plugins.dev
[ ] Submit PR to awesome-claude-skills
[ ] Post to LinkedIn (8-10 AM)
[ ] Post to r/Airtable
[ ] Post to r/ClaudeAI

Week 1:
[ ] Respond to all comments
[ ] Post in Airtable Community
[ ] Comment on 3-5 relevant Airtable threads

Week 2:
[ ] Follow-up post with feedback
[ ] Evaluate metrics vs targets
```

---

*Last updated: 2026-02-05*
