---
name: master-dev-skill
description: >
  Master orchestrator skill for taking a raw product idea from concept to fully documented, development-ready spec.
  Chains together six sub-skills in sequence: PRD (Product Requirements Document), system-design (technical architecture),
  uiux-wireframes (UI/UX wireframes and flows), feature-breakdown (granular feature tasks), frontend-design
  (production-grade UI components and visual design), and master-prompt (a master AI prompt to bootstrap the build).
  Use this skill whenever the user says things like "I have an idea", "I want to build", "help me spec out",
  "turn my idea into a plan", "I have a product concept", "let's build X", or any phrase indicating they want to
  go from idea → full product spec. Also trigger when the user explicitly invokes /master-dev-skill or asks to
  "run the full dev pipeline" on an idea. This skill is the right choice whenever multiple planning documents are
  needed together — do not run individual sub-skills separately when the user wants the full pipeline.
---

# Master Dev Skill

Transforms a raw product idea into a complete, development-ready specification package by orchestrating six sub-skills via dedicated subagents.

## What This Skill Produces

| # | Sub-Skill | Output |
|---|-----------|--------|
| 1 | `prd` | Product Requirements Document |
| 2 | `system-design` | Technical Architecture & System Design |
| 3 | `uiux-wireframes` | UI/UX Wireframes & User Flows |
| 4 | `feature-breakdown` | Granular Feature Breakdown & Task List |
| 5 | `frontend-design` | Production-Grade UI Components & Visual Design |
| 6 | `master-prompt` | Master AI Prompt to bootstrap the build |

---

## Orchestration Instructions

### Step 0 — Idea Intake

Before spawning any subagents, gather the following from the user (ask in a single message, don't bombard):

1. **What is the core idea?** (1–3 sentences)
2. **Who is the target user?**
3. **What platform?** (web, mobile, desktop, API, all)
4. **Any tech stack preferences?** (or "no preference")
5. **MVP scope or full vision?**

If the user has already provided this context, extract it from the conversation and confirm before proceeding. Do not ask for info already given.

Once confirmed, compile a **Shared Context Block** — a concise summary of the idea, users, platform, stack, and scope. Every subagent below receives this block verbatim as its starting context.

---

### Step 1 — Spawn Subagent: PRD

**Create a dedicated subagent** whose sole responsibility is the PRD skill.

**Subagent instructions:**
```
You are the PRD Subagent for the master-dev-skill pipeline.

SHARED CONTEXT:
{shared_context_block}

Your task:
1. Read `/mnt/skills/user/prd.md` if it exists. Follow its instructions exactly.
   If the file does not exist, use built-in knowledge.
2. Produce a complete Product Requirements Document containing:
   - Executive Summary
   - Problem Statement
   - Goals & Success Metrics
   - Target Users & Personas
   - Functional Requirements (must-have vs nice-to-have)
   - Non-Functional Requirements (performance, security, scalability)
   - Out of Scope
3. Save the output to `01_PRD.md`.
4. Return the full PRD content so the orchestrator can pass it to downstream subagents.
```

Wait for this subagent to complete and capture its output before proceeding. The PRD output feeds into all subsequent subagents.

---

### Step 2 — Spawn Subagent: System Design

**Create a dedicated subagent** whose sole responsibility is the system-design skill.

**Subagent instructions:**
```
You are the System Design Subagent for the master-dev-skill pipeline.

SHARED CONTEXT:
{shared_context_block}

PRD OUTPUT (from Step 1):
{prd_output}

Your task:
1. Read `/mnt/skills/user/system-design.md` if it exists. Follow its instructions exactly.
   If the file does not exist, use built-in knowledge.
2. Using the PRD above, produce a complete System Design document containing:
   - High-level architecture diagram (Mermaid or ASCII)
   - Component breakdown (frontend, backend, DB, third-party services)
   - Data models & schema outline
   - API surface (key endpoints)
   - Infrastructure & deployment strategy
   - Security considerations
3. Save the output to `02_SystemDesign.md`.
4. Return the full system design content.
```

Wait for this subagent to complete and capture its output.

---

### Step 3 — Spawn Subagent: UI/UX Wireframes

**Create a dedicated subagent** whose sole responsibility is the uiux-wireframes skill.

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
1. Read `/mnt/skills/user/uiux-wireframes.md` if it exists. Follow its instructions exactly.
   If the file does not exist, use built-in knowledge.
2. Using the PRD and system design above, produce a complete UI/UX document containing:
   - User journey / flow diagrams
   - Key screen wireframes (text-based ASCII or described layouts)
   - Navigation structure
   - Component inventory
   - Interaction notes & states (empty, loading, error, success)
3. Save the output to `03_UIUXWireframes.md`.
4. Return the full wireframes content.
```

Wait for this subagent to complete and capture its output.

---

### Step 4 — Spawn Subagent: Feature Breakdown

**Create a dedicated subagent** whose sole responsibility is the feature-breakdown skill.

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
1. Read `/mnt/skills/user/feature-breakdown.md` if it exists. Follow its instructions exactly.
   If the file does not exist, use built-in knowledge.
2. Decompose the product into a granular feature breakdown containing:
   - Epics (major feature areas)
   - User stories per epic (`As a [user], I want [X] so that [Y]`)
   - Acceptance criteria per story
   - Story point estimates (S/M/L/XL)
   - Suggested sprint groupings for MVP
3. Save the output to `04_FeatureBreakdown.md`.
4. Return the full feature breakdown content.
```

Wait for this subagent to complete and capture its output.

---

### Step 5 — Spawn Subagent: Frontend Design

**Create a dedicated subagent** whose sole responsibility is the frontend-design skill.

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
1. Read `/mnt/skills/public/frontend-design/SKILL.md` if it exists. Follow its instructions exactly.
   If the file does not exist, use built-in knowledge of production-grade frontend design.
2. Using the PRD, wireframes, and feature breakdown above, produce a complete Frontend Design document containing:
   - Design system foundations (color palette, typography scale, spacing tokens, component tokens)
   - Key UI components with detailed specs (buttons, inputs, cards, nav, modals, tables, etc.)
   - Page-level layout breakdowns for each major screen identified in the wireframes
   - Responsive behaviour notes (mobile / tablet / desktop)
   - Accessibility requirements (ARIA roles, contrast ratios, keyboard nav)
   - Animation & micro-interaction guidelines
   - Code snippets or pseudocode for the most complex components
3. Save the output to `05_FrontendDesign.md`.
4. Return the full frontend design content.
```

Wait for this subagent to complete and capture its output.

---

### Step 6 — Spawn Subagent: Master Prompt

**Create a dedicated subagent** whose sole responsibility is the master-prompt skill. This subagent runs last as it synthesises all prior outputs.

**Subagent instructions:**
```
You are the Master Prompt Subagent for the master-dev-skill pipeline.

SHARED CONTEXT:
{shared_context_block}

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

Your task:
1. Read `/mnt/skills/user/master-prompt.md` if it exists. Follow its instructions exactly.
   If the file does not exist, use built-in knowledge.
2. Synthesise ALL prior outputs into a single, comprehensive AI coding prompt that:
   - Describes the product clearly for an AI coding assistant
   - Includes tech stack, architecture decisions, and constraints from the system design
   - Lists MVP features in priority order from the feature breakdown
   - References the key screens and flows from the wireframes
   - Incorporates the design system tokens, component specs, and UI guidelines from the frontend design
   - Provides enough context to start coding immediately without needing other documents
   - Ends with: "Start by scaffolding the project structure and then implement [first feature]."
3. Save the output to `06_MasterPrompt.md`.
4. Return the full master prompt content.
```

Wait for this subagent to complete.

---

## Output & Delivery

After all six subagents are complete:

1. Present all six files to the user using `present_files`.
2. Give a brief summary table showing what was produced.
3. Provide the **How to Use** instructions below to the user verbatim.

---

## How to Use (Share with User After Completion)

> ### 🚀 How to Use Your Master Dev Package
>
> You now have 6 files that take your idea from concept to code-ready:
>
> | File | Use It To... |
> |------|-------------|
> | `01_PRD.md` | Align stakeholders, define scope, onboard team members |
> | `02_SystemDesign.md` | Guide your architecture decisions & infrastructure setup |
> | `03_UIUXWireframes.md` | Brief a designer or build your own UI flows |
> | `04_FeatureBreakdown.md` | Load into Jira/Linear/Notion as your sprint backlog |
> | `05_FrontendDesign.md` | Hand off to your frontend dev or use as a design system reference |
> | `06_MasterPrompt.md` | Paste into Claude Code, Cursor, or any AI coding assistant to start building |
>
> **Recommended Next Steps:**
> 1. Review `01_PRD.md` and confirm scope is right.
> 2. Paste `06_MasterPrompt.md` into **Claude Code** to start scaffolding immediately.
> 3. Use `04_FeatureBreakdown.md` to create your first sprint.
> 4. Use `05_FrontendDesign.md` as your living design system reference during development.
>
> **To invoke this skill again with a new idea:**
> ```
> /master-dev-skill [describe your idea here]
> ```
> Or simply say: *"I have a new idea I want to spec out"* and the skill will activate.

---

## Error Handling

- If a sub-skill file is not found at its expected path, proceed using built-in knowledge and note which sub-skill was run natively vs. from a skill file.
- If the user wants to skip a step, skip it and note it in the summary.
- If context from a prior step is ambiguous, make reasonable assumptions and flag them clearly in the output.

---

## Notes for Claude (Orchestrator)

### Subagent Execution Model
- You are the **orchestrator**. You do not produce the documents yourself — you spawn dedicated subagents to do it.
- Each subagent is a **fresh Claude instance** with a focused role. Give it its skill file path, the shared context, and all relevant outputs from prior steps. Nothing else.
- Steps 1–5 are **sequential** because each feeds into the next. Do not spawn Step 2 until Step 1's subagent has returned its output.
- Step 6 (Master Prompt) must be last — it synthesises everything.

### Passing Context Between Subagents
- After each subagent completes, extract its full document output and store it.
- Inject prior outputs into the next subagent's instructions verbatim using the `{placeholder}` slots shown above.
- Do not summarise or truncate prior outputs when passing them — subagents need full fidelity.

### Subagent Skill File Loading
- Each subagent must attempt to read its corresponding skill file first.
- Skill file paths follow the pattern: `/mnt/skills/user/{skill-name}/SKILL.md`
- If the file doesn't exist, the subagent falls back to built-in knowledge and notes it in its output.

### Output Standards
- Each subagent saves its output as a numbered `.md` file (`01_PRD.md` through `05_MasterPrompt.md`).
- All documents use **Markdown** with clear headings.
- Content must be **concrete and specific** to the user's actual idea — never generic templates.
- Each document must be **self-contained** so the user can share it independently.

### Orchestrator Behaviour
- Do not ask the user for approval between steps — run the full pipeline end-to-end.
- Announce to the user when each subagent starts and completes (e.g. "✅ PRD subagent complete — spinning up System Design subagent…").
- If a subagent fails or returns incomplete output, retry once with clarified instructions before flagging to the user.
- Present all 5 files together at the end using `present_files`.
