---
name: master-dev-skill
description: >
  Pipeline orchestrator skill that manages five sub-skills as subagents in sequence:
  PRD → System Design → UI/UX Wireframes → Feature Breakdown → Frontend Design.
  This skill handles the research and documentation pipeline — it gathers context,
  spawns each subagent in order, passes outputs downstream, and returns all five
  completed documents. It is designed to be invoked by the master-prompt skill,
  which handles the final synthesis step.
  Can also be called directly when the user wants to run the documentation pipeline
  without the final master prompt synthesis. Trigger phrases include:
  'run the dev pipeline', 'spec out my idea', 'I have an idea', 'I want to build',
  'help me plan', 'turn my idea into docs', 'create all the spec documents',
  or any phrase indicating they want multiple planning documents generated together.
---

# Master Dev Skill (Pipeline Orchestrator)

Orchestrates five sub-skills via dedicated subagents to transform a raw product idea into a complete documentation package. Each subagent reads its own SKILL.md, follows its instructions, and returns its full output for downstream use.

## What This Skill Produces

| Step | Sub-Skill Invoked | Skill File | Output File |
|------|-------------------|------------|-------------|
| 1 | PRD | `prd/SKILL.md` | `01_PRD.md` |
| 2 | System Design | `system-design/SKILL.md` | `02_SystemDesign.md` |
| 3 | UI/UX Wireframes | `uiux-wireframes/SKILL.md` | `03_UIUXWireframes.md` |
| 4 | Feature Breakdown | `feature-breakdown/SKILL.md` | `04_FeatureBreakdown.md` |
| 5 | Frontend Design | `frontend-design/SKILL.md` | `05_FrontendDesign.md` |

**Note:** This skill does NOT produce the final Master Prompt (Step 6). That is handled by `master-prompt/SKILL.md`, which invokes this skill and then synthesises all outputs.

---

## Pipeline Flow

```
User's Idea
    │
    ▼
[Step 0: Idea Intake — gather context]
    │
    ▼
┌──────────┐     ┌────────────────┐     ┌──────────────────┐
│  Step 1  │────▶│    Step 2       │────▶│     Step 3        │
│   PRD    │     │ System Design   │     │ UI/UX Wireframes  │
└──────────┘     └────────────────┘     └──────────────────┘
                                                │
    ┌───────────────────────────────────────────┘
    ▼
┌──────────────────┐     ┌─────────────────┐
│     Step 4        │────▶│    Step 5        │
│ Feature Breakdown │     │ Frontend Design  │
└──────────────────┘     └─────────────────┘
    │
    ▼
[Return all 5 outputs to caller]
```

---

## Step 0 — Idea Intake

If invoked directly by the user (not by master-prompt), gather the following in a **single conversational message**:

1. **What is the core idea?** (1–3 sentences)
2. **Who is the target user?**
3. **What platform?** (web, mobile, desktop, API, all)
4. **Any tech stack preferences?** (or "no preference")
5. **MVP scope or full vision?**

If invoked by `master-prompt` as a subagent, the Shared Context Block will already be provided — skip the interview.

Once the context is available, compile a **Shared Context Block** — a concise summary of the idea, users, platform, stack, and scope. Every subagent below receives this block verbatim.

---

## Step 1 — Spawn Subagent: PRD

**Read the PRD skill file and spawn a dedicated subagent.**

**Skill file to read:**
```
prd/SKILL.md
```

**Subagent instructions:**
```
You are the PRD Subagent for the master-dev-skill pipeline.

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
Announce: "🔄 Spawning PRD subagent..."
On completion: "✅ PRD complete."

---

## Step 2 — Spawn Subagent: System Design

**Read the System Design skill file and spawn a dedicated subagent.**

**Skill file to read:**
```
system-design/SKILL.md
```

**Subagent instructions:**
```
You are the System Design Subagent for the master-dev-skill pipeline.

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
Announce: "🔄 Spawning System Design subagent..."
On completion: "✅ System Design complete."

---

## Step 3 — Spawn Subagent: UI/UX Wireframes

**Read the UI/UX Wireframes skill file and spawn a dedicated subagent.**

**Skill file to read:**
```
uiux-wireframes/SKILL.md
```

**Subagent instructions:**
```
You are the UI/UX Wireframes Subagent for the master-dev-skill pipeline.

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
Announce: "🔄 Spawning UI/UX Wireframes subagent..."
On completion: "✅ UI/UX Wireframes complete."

---

## Step 4 — Spawn Subagent: Feature Breakdown

**Read the Feature Breakdown skill file and spawn a dedicated subagent.**

**Skill file to read:**
```
feature-breakdown/SKILL.md
```

**Subagent instructions:**
```
You are the Feature Breakdown Subagent for the master-dev-skill pipeline.

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
Announce: "🔄 Spawning Feature Breakdown subagent..."
On completion: "✅ Feature Breakdown complete."

---

## Step 5 — Spawn Subagent: Frontend Design

**Read the Frontend Design skill file and spawn a dedicated subagent.**

**Skill file to read:**
```
frontend-design/SKILL.md
```

**Subagent instructions:**
```
You are the Frontend Design Subagent for the master-dev-skill pipeline.

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
Announce: "🔄 Spawning Frontend Design subagent..."
On completion: "✅ Frontend Design complete."

---

## Final Output — Return to Caller

After all 5 subagents complete, compile and return ALL outputs:

```
PIPELINE COMPLETE. Returning all outputs:

PRD OUTPUT:
{prd_output}

SYSTEM DESIGN OUTPUT:
{system_design_output}

UI/UX WIREFRAMES OUTPUT:
{uiux_output}

FEATURE BREAKDOWN OUTPUT:
{feature_breakdown_output}

FRONTEND DESIGN OUTPUT:
{frontend_design_output}
```

If invoked directly by the user (not by master-prompt), also present the 5 files and suggest:
> "💡 Want a synthesised Master Prompt from all of this? Run the `master-prompt` skill — it will take these 5 documents and weave them into a single AI coding prompt."

---

## Relationship to Other Skills

```
master-prompt/SKILL.md  (Entry Point — user calls this)
    │
    ├── Invokes: master-dev-skill/SKILL.md  (THIS FILE — Pipeline Orchestrator)
    │       │
    │       ├── Spawns: prd/SKILL.md
    │       ├── Spawns: system-design/SKILL.md
    │       ├── Spawns: uiux-wireframes/SKILL.md
    │       ├── Spawns: feature-breakdown/SKILL.md
    │       └── Spawns: frontend-design/SKILL.md
    │
    └── Synthesises all outputs → 06_MasterPrompt.md
```

---

## Notes for Claude (Orchestrator Behaviour)

### Execution Model
- You are the **pipeline orchestrator**. You spawn dedicated subagents — you do NOT produce the documents yourself.
- Each subagent is a **focused instance** with a single responsibility.
- Steps 1–5 are **strictly sequential** — each feeds into the next. Do NOT spawn Step 2 until Step 1 returns.

### Context Passing
- After each subagent completes, capture its **full document output**.
- Inject prior outputs into the next subagent using the `{placeholder}` slots.
- **Do NOT summarise or truncate** prior outputs — subagents need full fidelity.

### Skill File Resolution
- Each subagent must attempt to read its skill file first.
- Skill files (relative to skills root):
  - `prd/SKILL.md`
  - `system-design/SKILL.md`
  - `uiux-wireframes/SKILL.md`
  - `feature-breakdown/SKILL.md`
  - `frontend-design/SKILL.md`
- If a file doesn't exist, the subagent falls back to built-in knowledge and notes it.

### Error Handling
- If a subagent fails, **retry once** with clarified instructions.
- If it fails again, note the gap and continue with remaining steps.
- If the user wants to skip a step, skip it and note it in the final output.
- Do NOT ask the user for approval between steps — run end-to-end.
