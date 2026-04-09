---
name: master-prompt
description: >
  Master entry-point skill that produces a comprehensive, development-ready Master Prompt Document.
  When invoked, it first calls the master-dev-skill (pipeline orchestrator) which spawns five
  sub-skills as subagents in sequence: PRD → System Design → UI/UX Wireframes → Feature Breakdown
  → Frontend Design. Once all five are complete, this skill synthesises their outputs into a single
  master prompt that any AI coding assistant can use to build the entire project.
  This skill is the top of the skill hierarchy — all 7 skills are wired together through it.
  Trigger this skill when the user mentions: 'master prompt', 'master prompt document',
  'project prompt doc', 'AI coding instructions', 'full dev spec', 'spec out my idea',
  'turn my idea into a plan', 'I want to build [X]', 'create a master doc', 'I have an idea',
  'help me spec out', 'run the full pipeline', 'make a prompt doc for Claude/GPT/Cursor',
  or any phrase indicating they want to go from idea → full product spec → code-ready prompt.
---

# Master Prompt Skill (Entry Point)

This skill is the **top-level entry point** for the entire development specification pipeline. When invoked, it:

1. Gathers the user's idea
2. Calls **master-dev-skill** (the pipeline orchestrator) which spawns 5 subagents
3. Receives all 5 completed documents
4. Synthesises everything into the final **Master Prompt Document**

## Complete Skill Hierarchy

```
┌─────────────────────────────────────────────────────────┐
│  master-prompt/SKILL.md  ← YOU ARE HERE (Entry Point)   │
│                                                          │
│  Invokes:                                                │
│  ┌─────────────────────────────────────────────────────┐ │
│  │  master-dev-skill/SKILL.md  (Pipeline Orchestrator) │ │
│  │                                                      │ │
│  │  Spawns 5 subagents:                                 │ │
│  │  ┌──────────────────────────────────────────┐        │ │
│  │  │ Step 1: prd/SKILL.md                     │        │ │
│  │  │ Step 2: system-design/SKILL.md           │        │ │
│  │  │ Step 3: uiux-wireframes/SKILL.md         │        │ │
│  │  │ Step 4: feature-breakdown/SKILL.md       │        │ │
│  │  │ Step 5: frontend-design/SKILL.md         │        │ │
│  │  └──────────────────────────────────────────┘        │ │
│  │                                                      │ │
│  │  Returns: 5 complete documents                       │ │
│  └─────────────────────────────────────────────────────┘ │
│                                                          │
│  Step 6: Synthesise → 06_MasterPrompt.md                 │
└─────────────────────────────────────────────────────────┘
```

## All 7 Skills and Their Roles

| # | Skill | File | Role |
|---|-------|------|------|
| 1 | **master-prompt** | `master-prompt/SKILL.md` | 🎯 Entry point — user calls this |
| 2 | **master-dev-skill** | `master-dev-skill/SKILL.md` | 🔧 Pipeline orchestrator — manages Steps 1–5 |
| 3 | **prd** | `prd/SKILL.md` | 📋 Worker — produces PRD |
| 4 | **system-design** | `system-design/SKILL.md` | 🏗️ Worker — produces System Design |
| 5 | **uiux-wireframes** | `uiux-wireframes/SKILL.md` | 🎨 Worker — produces UI/UX Wireframes |
| 6 | **feature-breakdown** | `feature-breakdown/SKILL.md` | 📦 Worker — produces Feature Breakdown |
| 7 | **frontend-design** | `frontend-design/SKILL.md` | 🖥️ Worker — produces Frontend Design |

## What This Skill Produces (Full Pipeline)

| Step | Handled By | Output File | Purpose |
|------|-----------|-------------|---------|
| 0 | master-prompt | *(context)* | Idea intake |
| 1 | master-dev-skill → prd | `01_PRD.md` | Product Requirements Document |
| 2 | master-dev-skill → system-design | `02_SystemDesign.md` | Technical Architecture & System Design |
| 3 | master-dev-skill → uiux-wireframes | `03_UIUXWireframes.md` | UI/UX Wireframes & User Flows |
| 4 | master-dev-skill → feature-breakdown | `04_FeatureBreakdown.md` | Granular Feature Breakdown & Task List |
| 5 | master-dev-skill → frontend-design | `05_FrontendDesign.md` | Production-Grade UI Components |
| 6 | master-prompt | `06_MasterPrompt.md` | Final synthesised Master Prompt |

---

## Step 0 — Idea Intake

Before invoking the pipeline, gather the following from the user in a **single conversational message** (don't bombard with separate questions):

1. **What is the core idea?** (1–3 sentences)
2. **Who is the target user?**
3. **What platform?** (web, mobile, desktop, API, all)
4. **Any tech stack preferences?** (or "no preference")
5. **MVP scope or full vision?**

If the user has already provided this context in their initial message, extract it and **confirm** before proceeding. Do not ask for info already given.

Once confirmed, compile a **Shared Context Block** — a concise summary of the idea, users, platform, stack, and scope. This block is passed to master-dev-skill.

---

## Step 1 — Invoke master-dev-skill (Pipeline Orchestrator)

**This is the key wiring step.** Read the master-dev-skill skill file and invoke it as a subagent.

**Skill file to read:**
```
master-dev-skill/SKILL.md
```

**Subagent instructions:**
```
You are the Pipeline Orchestrator Subagent (master-dev-skill).

SHARED CONTEXT:
{shared_context_block}

Your task:
1. Read the master-dev-skill skill file (master-dev-skill/SKILL.md). Follow its instructions exactly.
2. The Shared Context above contains the user's idea — skip the Idea Intake step (Step 0).
3. Execute the FULL pipeline — Steps 1 through 5:
   - Step 1: Spawn a PRD subagent (reads prd/SKILL.md) → produce 01_PRD.md
   - Step 2: Spawn a System Design subagent (reads system-design/SKILL.md) → produce 02_SystemDesign.md
   - Step 3: Spawn a UI/UX Wireframes subagent (reads uiux-wireframes/SKILL.md) → produce 03_UIUXWireframes.md
   - Step 4: Spawn a Feature Breakdown subagent (reads feature-breakdown/SKILL.md) → produce 04_FeatureBreakdown.md
   - Step 5: Spawn a Frontend Design subagent (reads frontend-design/SKILL.md) → produce 05_FrontendDesign.md
4. Pass all outputs downstream between steps (PRD feeds System Design, etc.)
5. Return ALL 5 complete document outputs so I can synthesise the Master Prompt.
```

**Wait for master-dev-skill to complete and return all 5 outputs.**

Announce to user:
- "🔄 Invoking master-dev-skill pipeline orchestrator..."
- "📋 Pipeline is running: PRD → System Design → UI/UX → Features → Frontend..."

On completion:
- "✅ Pipeline complete — all 5 documents generated. Now synthesising Master Prompt..."

---

## Step 2 — Synthesise the Master Prompt (This Skill's Core Job)

Now that master-dev-skill has returned all 5 outputs, **you** (master-prompt) synthesise everything into the final Master Prompt.

### Inputs Available

You now have all of these from the pipeline:
- `{shared_context_block}` — the original idea and constraints
- `{prd_output}` — full PRD from pipeline Step 1
- `{system_design_output}` — full System Design from pipeline Step 2
- `{uiux_output}` — full UI/UX Wireframes from pipeline Step 3
- `{feature_breakdown_output}` — full Feature Breakdown from pipeline Step 4
- `{frontend_design_output}` — full Frontend Design from pipeline Step 5

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
- **Resolve conflicts**: If outputs conflict, prefer System Design for technical decisions, PRD for scope decisions
- **No placeholders**: Replace all `REPLACE_ME` or template text with actual project-specific content

---

## Output & Delivery

After Step 2 (synthesis) is complete, present ALL 6 files to the user:

### Files Produced

| # | Document | Produced By | Status |
|---|----------|------------|--------|
| 1 | `01_PRD.md` | prd (via master-dev-skill) | ✅ Complete |
| 2 | `02_SystemDesign.md` | system-design (via master-dev-skill) | ✅ Complete |
| 3 | `03_UIUXWireframes.md` | uiux-wireframes (via master-dev-skill) | ✅ Complete |
| 4 | `04_FeatureBreakdown.md` | feature-breakdown (via master-dev-skill) | ✅ Complete |
| 5 | `05_FrontendDesign.md` | frontend-design (via master-dev-skill) | ✅ Complete |
| 6 | `06_MasterPrompt.md` | **master-prompt (this skill)** | ✅ Complete |

### How to Use (share with user)

> ### 🚀 How to Use Your Master Dev Package
>
> You now have 6 files produced by all 7 skills working together:
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
> 4. Use `05_FrontendDesign.md` as your living design system reference
>
> **Skill hierarchy that produced this:**
> ```
> master-prompt (entry point)
>   └── master-dev-skill (pipeline orchestrator)
>         ├── prd
>         ├── system-design
>         ├── uiux-wireframes
>         ├── feature-breakdown
>         └── frontend-design
> ```

---

## Fallback: Direct Pipeline Mode

If `master-dev-skill/SKILL.md` is NOT available (file not found), this skill falls back to invoking each sub-skill directly:

1. Invoke `prd/SKILL.md` as subagent → capture output
2. Invoke `system-design/SKILL.md` as subagent → capture output
3. Invoke `uiux-wireframes/SKILL.md` as subagent → capture output
4. Invoke `feature-breakdown/SKILL.md` as subagent → capture output
5. Invoke `frontend-design/SKILL.md` as subagent → capture output
6. Synthesise all outputs into `06_MasterPrompt.md`

This ensures the skill works even if master-dev-skill is missing — it just handles the orchestration itself.

---

## Fallback: Standalone Mode

If NONE of the sub-skill files are available:

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
- You are the **entry point**. The user talks to you.
- You invoke `master-dev-skill` as a subagent to run Steps 1–5.
- You do NOT produce Steps 1–5 yourself — that's master-dev-skill's job.
- You handle Step 0 (idea intake) and Step 6 (synthesis) yourself.

### Context Passing
- Pass the Shared Context Block to master-dev-skill.
- master-dev-skill passes it downstream to each sub-skill subagent.
- master-dev-skill returns all 5 outputs back to you.
- You use all 5 outputs to synthesise the Master Prompt.
- **Do NOT summarise or truncate** any outputs — full fidelity is required.

### Skill File Resolution
All skill files are in sibling folders relative to this skill:
- `master-dev-skill/SKILL.md` — Pipeline Orchestrator
- `prd/SKILL.md` — PRD Worker
- `system-design/SKILL.md` — System Design Worker
- `uiux-wireframes/SKILL.md` — UI/UX Wireframes Worker
- `feature-breakdown/SKILL.md` — Feature Breakdown Worker
- `frontend-design/SKILL.md` — Frontend Design Worker

### Progress Updates
Announce to the user at each stage:
- "🔄 Gathering your idea..."
- "🔄 Invoking master-dev-skill pipeline orchestrator..."
- "📋 Pipeline running: PRD → System Design → UI/UX → Features → Frontend..."
- "✅ Pipeline complete — all 5 documents generated."
- "🔄 Synthesising Master Prompt from all outputs..."
- "✅ Master Prompt complete! Here are your 6 documents."

### Error Handling
- If master-dev-skill fails, fall back to Direct Pipeline Mode (invoke each skill individually).
- If a sub-skill fails, retry once, then skip and note the gap.
- If the user wants to skip a step, skip it and note it.
- Do NOT ask the user for approval between steps — run end-to-end.
