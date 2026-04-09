---
name: master-prompt
description: >
  Master orchestrator skill that produces a comprehensive, development-ready Master Prompt Document
  by invoking five sub-skills as subagents in sequence: PRD → System Design → UI/UX Wireframes →
  Feature Breakdown → Frontend Design — then synthesising all their outputs into a single master
  prompt that any AI coding assistant can use to build the entire project.
  Trigger this skill when the user mentions: 'master prompt', 'master prompt document',
  'project prompt doc', 'AI coding instructions', 'project overview doc', 'full dev spec',
  'spec out my idea', 'turn my idea into a plan', 'I want to build [X]', 'create a master doc',
  'make a prompt doc for Claude/GPT/Cursor', 'run the full pipeline', 'I have an idea',
  'help me spec out', or any phrase indicating they want to go from idea → full product spec
  → code-ready master prompt. This skill chains all sub-skills together automatically — the user
  only needs to describe their idea and this skill handles the rest.
---

# Master Prompt Skill (Orchestrator)

This skill is the **master orchestrator**. When invoked, it runs the full development specification pipeline by spawning **five dedicated subagents** — each responsible for one skill — then **synthesises all their outputs** into a single, comprehensive Master Prompt Document.

## What This Skill Produces

| Step | Sub-Skill Invoked | Output File | Purpose |
|------|-------------------|-------------|---------|
| 1 | `prd` | `01_PRD.md` | Product Requirements Document |
| 2 | `system-design` | `02_SystemDesign.md` | Technical Architecture & System Design |
| 3 | `uiux-wireframes` | `03_UIUXWireframes.md` | UI/UX Wireframes & User Flows |
| 4 | `feature-breakdown` | `04_FeatureBreakdown.md` | Granular Feature Breakdown & Task List |
| 5 | `frontend-design` | `05_FrontendDesign.md` | Production-Grade UI Components & Visual Design |
| 6 | *(this skill)* | `06_MasterPrompt.md` | Final synthesised Master Prompt |

---

## Pipeline Flow

```
User's Idea
    │
    ▼
┌──────────┐     ┌────────────────┐     ┌──────────────────┐
│  Step 1  │────▶│    Step 2       │────▶│     Step 3        │
│   PRD    │     │ System Design   │     │ UI/UX Wireframes  │
└──────────┘     └────────────────┘     └──────────────────┘
                                                │
    ┌───────────────────────────────────────────┘
    ▼
┌──────────────────┐     ┌─────────────────┐     ┌──────────────────┐
│     Step 4        │────▶│    Step 5        │────▶│     Step 6        │
│ Feature Breakdown │     │ Frontend Design  │     │  MASTER PROMPT    │
└──────────────────┘     └─────────────────┘     │  (Synthesis)      │
                                                  └──────────────────┘
```

---

## Step 0 — Idea Intake

Before spawning any subagents, gather the following from the user in a **single conversational message** (don't bombard with separate questions):

1. **What is the core idea?** (1–3 sentences)
2. **Who is the target user?**
3. **What platform?** (web, mobile, desktop, API, all)
4. **Any tech stack preferences?** (or "no preference")
5. **MVP scope or full vision?**

If the user has already provided this context in their initial message, extract it and **confirm** before proceeding. Do not ask for info already given.

Once confirmed, compile a **Shared Context Block** — a concise summary of the idea, users, platform, stack, and scope. Every subagent below receives this block verbatim.

---

## Step 1 — Invoke Subagent: PRD

**Read the PRD skill file and spawn a dedicated subagent for it.**

**Skill file to read:**
```
prd/SKILL.md
```

**Subagent instructions:**
```
You are the PRD Subagent.

SHARED CONTEXT:
{shared_context_block}

Your task:
1. Read the PRD skill file (prd/SKILL.md). Follow its instructions exactly.
   If the file does not exist, use built-in knowledge of PRD best practices.
2. IMPORTANT: Skip the interview step — all context is provided above.
   Extract project details from the Shared Context instead of asking the user.
3. Produce a complete Product Requirements Document containing:
   - Problem Statement (what problem, why now, current pain points)
   - Target Users (primary, secondary, out-of-scope)
   - Feature List (MVP/basic features, advanced/future features, out of scope)
   - User Flow (step-by-step journey from user arrives → goal achieved)
   - Tech Preferences (if specified in context)
4. Save the output to `01_PRD.md`.
5. Return the FULL PRD content so the orchestrator can pass it downstream.
```

**Wait for this subagent to complete and capture its full output before proceeding.**
The PRD output feeds into ALL subsequent subagents.

---

## Step 2 — Invoke Subagent: System Design

**Read the System Design skill file and spawn a dedicated subagent for it.**

**Skill file to read:**
```
system-design/SKILL.md
```

**Subagent instructions:**
```
You are the System Design Subagent.

SHARED CONTEXT:
{shared_context_block}

PRD OUTPUT (from Step 1):
{prd_output}

Your task:
1. Read the System Design skill file (system-design/SKILL.md). Follow its instructions exactly.
   If the file does not exist, use built-in knowledge.
2. IMPORTANT: Skip the context gathering step — use the PRD output and Shared Context above.
3. Using the PRD above, produce a complete System Design Document covering:
   - Frontend + Backend Architecture (with architecture diagram)
   - API Structure and Flow (endpoints table, request/response flow)
   - Database Schema (entity tables, indexing strategy)
   - Authentication and Security Flow (auth strategy, registration/login flows, security checklist)
4. Save the output to `02_SystemDesign.md`.
5. Return the FULL system design content.
```

**Wait for completion. Capture output.**

---

## Step 3 — Invoke Subagent: UI/UX Wireframes

**Read the UI/UX Wireframes skill file and spawn a dedicated subagent for it.**

**Skill file to read:**
```
uiux-wireframes/SKILL.md
```

**Subagent instructions:**
```
You are the UI/UX Wireframes Subagent.

SHARED CONTEXT:
{shared_context_block}

PRD OUTPUT (from Step 1):
{prd_output}

SYSTEM DESIGN OUTPUT (from Step 2):
{system_design_output}

Your task:
1. Read the UI/UX Wireframes skill file (uiux-wireframes/SKILL.md). Follow its instructions exactly.
   If the file does not exist, use built-in knowledge.
2. Using the PRD and System Design above, produce a complete UI/UX document containing:
   - Page-by-page wireframe layouts (header, main sections, footer)
   - Navigation hierarchy and structure
   - User interaction flow (how users move through the interface)
   - Responsive behaviour (desktop, tablet, mobile layouts)
   - Component suggestions (buttons, cards, forms, modals, etc.)
   - UX best practices applied to this specific product
3. Save the output to `03_UIUXWireframes.md`.
4. Return the FULL wireframes content.
```

**Wait for completion. Capture output.**

---

## Step 4 — Invoke Subagent: Feature Breakdown

**Read the Feature Breakdown skill file and spawn a dedicated subagent for it.**

**Skill file to read:**
```
feature-breakdown/SKILL.md
```

**Subagent instructions:**
```
You are the Feature Breakdown Subagent.

SHARED CONTEXT:
{shared_context_block}

PRD OUTPUT (from Step 1):
{prd_output}

SYSTEM DESIGN OUTPUT (from Step 2):
{system_design_output}

Your task:
1. Read the Feature Breakdown skill file (feature-breakdown/SKILL.md). Follow its instructions exactly.
   If the file does not exist, use built-in knowledge.
2. Decompose the product into a granular feature breakdown containing:
   - Feature tree (top-level features → sub-features)
   - Description, implementation notes, and acceptance criteria per sub-feature
   - Feature overview table (Feature | Sub-features count | Priority | Status)
   - Dependencies between features
3. Save the output to `04_FeatureBreakdown.md`.
4. Return the FULL feature breakdown content.
```

**Wait for completion. Capture output.**

---

## Step 5 — Invoke Subagent: Frontend Design

**Read the Frontend Design skill file and spawn a dedicated subagent for it.**

**Skill file to read:**
```
frontend-design/SKILL.md
```

**Subagent instructions:**
```
You are the Frontend Design Subagent.

SHARED CONTEXT:
{shared_context_block}

PRD OUTPUT (from Step 1):
{prd_output}

UI/UX WIREFRAMES OUTPUT (from Step 3):
{uiux_output}

FEATURE BREAKDOWN OUTPUT (from Step 4):
{feature_breakdown_output}

Your task:
1. Read the Frontend Design skill file (frontend-design/SKILL.md). Follow its instructions exactly.
   If the file does not exist, use built-in knowledge of production-grade frontend design.
2. Using the PRD, wireframes, and feature breakdown above, produce a complete Frontend Design document:
   - Design system foundations (color palette, typography scale, spacing tokens, component tokens)
   - Key UI components with detailed specs (buttons, inputs, cards, nav, modals, tables)
   - Page-level layout breakdowns for each major screen identified in the wireframes
   - Responsive behaviour notes (mobile / tablet / desktop)
   - Accessibility requirements (ARIA roles, contrast ratios, keyboard nav)
   - Animation & micro-interaction guidelines
   - Code snippets or pseudocode for the most complex components
3. Save the output to `05_FrontendDesign.md`.
4. Return the FULL frontend design content.
```

**Wait for completion. Capture output.**

---

## Step 6 — Synthesise the Master Prompt (This Skill's Core Job)

Now that all 5 subagents have completed, **you** (the orchestrator) synthesise everything into the final Master Prompt.

### Inputs Available

You now have all of these:
- `{shared_context_block}` — the original idea and constraints
- `{prd_output}` — full PRD from Step 1
- `{system_design_output}` — full System Design from Step 2
- `{uiux_output}` — full UI/UX Wireframes from Step 3
- `{feature_breakdown_output}` — full Feature Breakdown from Step 4
- `{frontend_design_output}` — full Frontend Design from Step 5

### Master Prompt Document Structure

Produce `06_MasterPrompt.md` with this structure:

```
# Master Prompt: [Project Name]

> This document is a complete, self-contained brief for an AI coding assistant.
> Read it fully before writing any code.

## 1. Project Overview
   - What the product is (from PRD: Problem Statement)
   - Who it's for (from PRD: Target Users)
   - Core goals and success metrics (from PRD: Goals)
   - What's in scope / out of scope (from PRD: Scope)

## 2. Tech Stack & Architecture
   - Full tech stack table (from System Design: Section 1)
   - Architecture diagram (from System Design: Section 1.2)
   - Infrastructure & deployment (from System Design: Section 1.5)

## 3. API & Data Layer
   - API endpoints table (from System Design: Section 2)
   - Database schema summaries (from System Design: Section 3)
   - Auth strategy and token flow (from System Design: Section 4)

## 4. Screens & User Flows
   - Key screens with layout descriptions (from UI/UX Wireframes)
   - Navigation structure (from UI/UX Wireframes)
   - User journey flow (from UI/UX Wireframes)
   - Responsive behaviour rules (from UI/UX Wireframes)

## 5. Features — Priority Order
   - MVP features with sub-tasks (from Feature Breakdown)
   - Acceptance criteria per feature (from Feature Breakdown)
   - Feature dependencies (from Feature Breakdown)

## 6. Design System & UI Guidelines
   - Color palette and tokens (from Frontend Design)
   - Typography scale (from Frontend Design)
   - Component specifications (from Frontend Design)
   - Animation guidelines (from Frontend Design)
   - Accessibility requirements (from Frontend Design)

## 7. Strict Coding Rules
   (Generate these based on the tech stack and architecture decisions)
   - Language and framework rules
   - Naming conventions
   - File/folder structure
   - Import conventions
   - Error handling patterns
   - Security requirements
   - Anti-patterns to avoid

## 8. Folder Structure
   (Generate based on tech stack and architecture)
   - Root folder tree diagram
   - File naming conventions table

## 9. Getting Started
   "Start by scaffolding the project structure, then implement [first MVP feature]."
   - Step-by-step setup instructions
   - Environment variables needed
   - First feature to build
```

### Synthesis Rules

- **Be specific**: Every section must reference the actual product, not generic templates
- **Be comprehensive**: Include enough detail that someone can start coding without any other document
- **Be actionable**: End with clear "start here" instructions
- **Incorporate everything**: Don't skip outputs from any subagent — weave them all in
- **Resolve conflicts**: If subagent outputs conflict, prefer System Design for technical decisions, PRD for scope decisions
- **No placeholders**: Replace all `REPLACE_ME` or template text with actual project-specific content

---

## Output & Delivery

After all 6 steps are complete:

1. **Present all 6 files** to the user:
   - `01_PRD.md`
   - `02_SystemDesign.md`
   - `03_UIUXWireframes.md`
   - `04_FeatureBreakdown.md`
   - `05_FrontendDesign.md`
   - `06_MasterPrompt.md`

2. **Summary table**:

| # | Document | Status |
|---|----------|--------|
| 1 | PRD | ✅ Complete |
| 2 | System Design | ✅ Complete |
| 3 | UI/UX Wireframes | ✅ Complete |
| 4 | Feature Breakdown | ✅ Complete |
| 5 | Frontend Design | ✅ Complete |
| 6 | Master Prompt | ✅ Complete |

3. **How to Use** (share with user):

> ### 🚀 How to Use Your Master Dev Package
>
> | File | Use It To... |
> |------|-------------|
> | `01_PRD.md` | Align stakeholders, define scope, onboard team members |
> | `02_SystemDesign.md` | Guide architecture decisions & infrastructure setup |
> | `03_UIUXWireframes.md` | Brief a designer or build your own UI flows |
> | `04_FeatureBreakdown.md` | Load into Jira/Linear/Notion as your sprint backlog |
> | `05_FrontendDesign.md` | Hand off to frontend dev or use as design system reference |
> | `06_MasterPrompt.md` | **Paste into Claude / Cursor / GPT to start building immediately** |
>
> **Next Steps:**
> 1. Review `01_PRD.md` and confirm scope is right
> 2. Paste `06_MasterPrompt.md` into **Claude Code** to start scaffolding
> 3. Use `04_FeatureBreakdown.md` to create your first sprint

---

## Standalone Mode (Fallback)

If for some reason the sub-skill files are NOT available (files not found at their paths), this skill falls back to standalone mode:

1. Run a short interview to gather: project name, description, platform, tech stack, strict rules, code style, folder structure
2. Generate the Master Prompt Document directly with these 5 sections:
   - Project Overview
   - Strict Instructions
   - Tech Stack
   - Code Style Guidelines
   - Output Format (Files, Folders, Structure)
3. Save as `06_MasterPrompt.md`

**Note:** Standalone mode produces a lighter document since it lacks the depth of the full pipeline. Always prefer the full pipeline when sub-skills are available.

---

## Notes for Claude (Orchestrator Behaviour)

### Execution Model
- You are the **orchestrator**. You spawn dedicated subagents — you do NOT produce Steps 1–5 yourself.
- Each subagent is a **focused instance** with a single responsibility. Give it its skill file path, the shared context, and all relevant prior outputs.
- Steps 1–5 are **strictly sequential** — each feeds into the next. Do NOT spawn Step 2 until Step 1 returns.
- Step 6 (synthesis) is done by YOU, the orchestrator, after all subagents complete.

### Context Passing
- After each subagent completes, capture its **full document output** and store it.
- Inject prior outputs into the next subagent's instructions using the `{placeholder}` slots shown above.
- **Do NOT summarise or truncate** prior outputs — subagents need full fidelity.

### Skill File Resolution
- Each subagent must attempt to read its skill file first.
- Skill file paths (relative to skills root):
  - `prd/SKILL.md`
  - `system-design/SKILL.md`
  - `uiux-wireframes/SKILL.md`
  - `feature-breakdown/SKILL.md`
  - `frontend-design/SKILL.md`
- If a file doesn't exist, the subagent falls back to built-in knowledge and notes it.

### Progress Updates
- Announce to the user when each subagent starts and completes:
  - "🔄 Spawning PRD subagent..."
  - "✅ PRD complete — spawning System Design subagent..."
  - "✅ System Design complete — spawning UI/UX Wireframes subagent..."
  - etc.
- Do NOT ask the user for approval between steps — run end-to-end.

### Error Handling
- If a subagent fails or returns incomplete output, **retry once** with clarified instructions.
- If it fails again, note the gap and continue with remaining steps.
- If the user wants to **skip a step**, skip it and note it in the final summary.
- If context from a prior step is ambiguous, make reasonable assumptions and flag them.

### Output Standards
- All documents use **Markdown** with clear headings.
- Content must be **concrete and specific** to the user's actual idea — never generic templates.
- Each document must be **self-contained** so the user can share it independently.
- The final `06_MasterPrompt.md` is the crown jewel — it must be comprehensive enough to start coding from.
