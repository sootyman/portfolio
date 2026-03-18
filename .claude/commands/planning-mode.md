---
name: planning-mode
description: "Enhanced planning mode that walks through every meaningful decision one at a time, recording each as a lightweight markdown ADR in docs/adr/ADR-NNN.md with options, recommendation, and rationale. Tracks all decisions in docs/adr/decisions.md. Use for planning new projects or major features."
argument-hint: "[describe what you want to build or the feature you're planning]"
---

# Planning Mode

You are helping the user plan a project or major feature by walking them through every meaningful decision — one at a time — using lightweight markdown ADRs. You present options clearly in plain English and track every decision in `docs/adr/decisions.md` with individual ADR files for each decision.

The user's request is: **$ARGUMENTS**

**Core principles:**
- Write in plain English. Explain things like you're talking to a smart friend, not writing documentation.
- Always present exactly 4 options per decision (unless the user asks for more).
- Always include a recommendation and explain why you recommend it.
- Keep a persistent record of every decision so the user can revisit and change their mind.

---

## PHASE 1 — Understand the Request

Read `$ARGUMENTS` carefully.

If the request is clear and gives you enough to identify decision points (e.g. "I want to build a neighborhood book-sharing app where people can list books they're willing to lend, browse what's available nearby, and request to borrow them"), proceed to Phase 2.

If `$ARGUMENTS` is empty, very short, or too vague to plan around, invoke the `superpowers:brainstorming` skill to explore the user's intent, requirements, and design ideas through an interactive conversation. The goal is to understand what they want to build deeply enough to identify all the meaningful decisions. Only proceed to Phase 2 once brainstorming is complete.

Following the brainstorming, proceed to PHASE 2

---

## PHASE 2 — Identify All Decision Points

Analyze the project and list every meaningful decision the user will need to make. Group them into categories:

**Technical** — tech stack, framework, database, auth approach, hosting, API design, data modeling
**Visual/UX** — overall visual style, component design, color palette, typography, layout patterns
**Interaction** — user flows, navigation patterns, onboarding, how key actions work step by step
**Information Architecture** — what goes in the nav, content hierarchy, what's prominent vs. buried, page structure

### Ordering Rules
1. Foundational decisions first (tech stack, overall style direction) — these unlock later decisions
2. Group related decisions together when possible
3. UX/Visual decisions should be interleaved with technical ones — don't dump all technical decisions first
4. Aim for 5–10 decisions for a medium project. Fewer for simple projects, more for complex ones. Don't invent decisions that don't matter.

### Present the Roadmap

Before diving into the first decision, show the user the full list - for example:

> "Here's what we'll figure out together. I'll walk you through each one with options, visuals, and my recommendation:
>
> 1. **Frontend Framework** (Technical) — What we'll build the UI with
> 2. **Backend & Data** (Technical) — Where the data lives and how it's served
> 3. **Visual Direction** (Visual) — The overall look and feel
> 4. **Main Navigation** (IA) — How people find their way around
> 5. **Core User Flow** (Interaction) — How the main action works step by step
> 6. **Card Design** (Visual) — How individual items look in lists
> 7. **Discovery Method** (Interaction) — How users find what they're looking for
>
> Let's start with #1."

Wait for the user to acknowledge or adjust the list, then proceed to Phase 3 with decision #1.

---

## PHASE 3 — Present a Decision as Markdown

For each decision point, you will generate a self-contained Markdown file.

### Step 3a — Set Up the ADR Directory

On the first decision only, create the directory and state file:

```bash
mkdir -p adr
```

If `docs/adr/decisions.md` does not exist, create it with these headers (example row shown):

| Reference | Decision | Status | Link |
| --------- | ---------|-------| ------| 
| ADR-000 | Shortened decision description | Proposed / In Review / Confirmed | [ADR-000.md]

### Step 3b — Generate the ADR-nnnn.md

Write a self-contained Markdown file to `docs/adr/ADR-NNN.md` where NNN is a zero-padded number (001, 002, etc.) and slug is a short kebab-case summary (e.g. `frontend-framework`, `visual-direction`, `main-navigation`).


### Step 3c — Update decisions.md

Update decisions.md with a new row for the decision

### Step 3d — Tell the User

> "I've written **Decision N: [Title]** to `docs/adr/ADR-NNN.md`. Take a look at the 4 options — I've recommended Option [X] but they're all solid choices.
>
> When you're ready, tell me:
> - **'Option B'** — to go with that one
> - **'Option A but [your tweak]'** — to customize an option
> - **'More options'** — I'll add 4 more
> - Or just tell me what you're thinking and we'll figure it out"

**Wait for the user's response. Do not proceed to the next decision until this one is resolved.**

---

## PHASE 4 — Handle the User's Response

### Choosing an Option

When the user picks an option (e.g. "Option B", "B", "the second one", "Svelte Speedster"):

1. **Update the ADR file**: Mark the chosen option clearly (e.g. prepend `**CHOSEN**` to the option heading). Strike through or note all other options as not chosen.
2. **Update decisions.md**: Change the row status to `Confirmed` and note the chosen option title.
3. **Confirm plainly**:

> "Got it — going with Option B ('Svelte Speedster') for the frontend framework. That's a great pick for this project.
>
> Next up: **Decision 2 — Backend & Data**. Let me put together the options..."

Then proceed to Phase 3 for the next decision.

### "Option A but [modification]"

When the user wants a modified version:

1. **Generate a new version of that option** incorporating their modification
2. **Rewrite the ADR file** with the modified option replacing the original (keep the same letter)
3. Tell the user:

> "I've updated Option A with your change — [brief description of modification]. Take another look and let me know if that's the one, or if you want to tweak it further."

### "More Options"

When the user asks for more choices:

1. **Read the existing ADR file** to understand what options are already shown
2. **Determine the next batch of letters**: If A–D exist, next batch is E–H. If A–H exist, next is I–L. And so on.
3. **Generate 4 new options** that are meaningfully different from all existing options
4. **Append new option sections** to the ADR markdown file
5. **Extend the comparison table** with new rows for the new options
6. **Rewrite the full ADR file**
7. **Update decisions.md**: extend the options noted in the row
8. Tell the user:

> "Added Options E through H — there are now 8 options on the page. Take a look and let me know which one speaks to you."

### Changing a Past Decision

When the user says something like "for decision-001 I want Option C instead" or "I changed my mind about the frontend framework":

1. **Read the relevant ADR file and decisions.md**
2. **Update the ADR**: Move the `**CHOSEN**` marker to the new option, remove it from the old one
3. **Update decisions.md**: Change the status and chosen option in the row
4. Tell the user:

> "Done — switched Decision 1 (Frontend Framework) from Option B ('Svelte Speedster') to Option C ('Vue Versatile'). The decision page and landing page are both updated."

If the change affects downstream decisions (e.g. changing the framework might affect component design options), note this:

> "Heads up: this might affect Decision 4 (Card Design) since the component patterns are different in Vue vs Svelte. Want me to regenerate those options?"

---

## PHASE 5 — Generate Final Plan

After ALL decisions are resolved:

### Step 5a — Write the Implementation Plan

Generate a markdown summary that reads like a project brief. Save it as `docs/adr/implementation-plan.md`:

```markdown
# Implementation Plan: [Project Name]

## What We're Building
[2-3 sentence plain English summary]

## Decisions Made
| # | Decision | Choice | Category |
|---|----------|--------|----------|
| 1 | Frontend Framework | Option B: Svelte Speedster | Technical |
| 2 | Backend & Data | Option A: Supabase Simple | Technical |
| ... | ... | ... | ... |

## Implementation Steps

### 1. Project Setup
- [ ] Initialize [framework] project
- [ ] Set up [database/backend]
- [ ] Configure [hosting/deployment]

### 2. Core Structure
- [ ] Create main layout with [navigation choice]
- [ ] Set up routing for key pages
- [ ] Implement [visual direction] theme/styles

### 3. Key Features
- [ ] Build [core flow] as decided in Decision N
- [ ] Create [component] using [design choice]
- [ ] Implement [discovery method]

### 4. Polish & Launch
- [ ] Test all user flows end to end
- [ ] Responsive design pass
- [ ] Deploy to [hosting choice]

## Decision History
All decision documents are saved in the `docs/adr/` folder.
Review `docs/adr/decisions.md` for the full decision index, or open individual `docs/adr/ADR-NNN.md` files.
```

### Step 5b — Present the Plan and Ask About Execution

> "All decisions are locked in! Here's your implementation plan with [N] steps.
>
> I've saved the full plan to `docs/adr/implementation-plan.md` and your decision history is in `docs/adr/decisions.md`.
>
> How would you like to proceed?
> - **'Auto mode'** — I'll work through the task list and auto-run tools (you can still stop me anytime)
> - **'Step by step'** — I'll ask for your OK before each major action
> - **'Let me review first'** — Take a look at the plan and tell me if you want changes before we start
> - **'Just the plan'** — We're done for now, you'll implement it yourself or come back later"

Wait for the user's response and proceed accordingly.

---

## EDGE CASES

**User skips a decision:** "Skip this one" or "doesn't matter" → Mark status as `Skipped — Claude will decide` in `decisions.md`. Use your recommendation when implementing.

**User gives a custom answer not matching any option:** "I want to use Postgres with Prisma ORM" → Record it as a custom choice. Note `Custom: [their description]` in the ADR and update `decisions.md` with status `Confirmed (custom)`.

**User wants to revisit the decision list:** "What decisions have we made?" or "show me the overview" → Read `docs/adr/decisions.md` and summarize the current state.

**User wants to jump ahead:** "Let's do the navigation decision next" → Reorder and present that decision next, then continue with remaining decisions.

**Existing adr directory:** If `docs/adr/` already exists from a prior session, read `docs/adr/decisions.md` to understand what's been decided. Resume from the first pending decision. Tell the user: "I see we've already made N decisions. Picking up where we left off with Decision M: [Title]."

**User says "just decide for me":** Use your recommendation for all remaining decisions. Record them all, generate the implementation plan, and present it.

---

## IMPORTANT REMINDERS

1. **Always 4 options.** Not 3, not 5. Exactly 4. Unless the user has asked for more.
2. **Always include a recommendation.** Mark it clearly (e.g. `> **Recommended**`). Explain WHY you recommend it in the option summary.
3. **Plain English everywhere.** If you use a technical term, explain it in the same sentence. "Supabase (a hosted database that handles authentication too)" is better than just "Supabase".
4. **The comparison table is mandatory.** Every ADR must have one. Pick dimensions that actually help differentiate the options.
5. **Wait for the user.** After writing an ADR, STOP and wait. Do not proceed to the next decision until the user has made a choice.
6. **Update decisions.md after every change.** The index should always reflect the current state of all decisions.
7. **Handle decision changes gracefully.** When a user changes a past decision, update both the individual ADR and `decisions.md`. Flag any downstream decisions that might be affected.