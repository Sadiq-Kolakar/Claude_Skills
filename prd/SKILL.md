---
name: prd
description: >
  Generate a structured, professional Product Requirement Document (PRD) as a formatted .docx file.
  Use this skill whenever the user wants to write, create, draft, or produce a PRD, product spec,
  product requirement, feature spec, or project brief — even if they just say "help me plan my product"
  or "I want to document what I'm building." Trigger also for "write a spec for X", "create a product doc",
  "I need requirements for my app", or any request to formalize a product idea into a structured document.
  This skill conducts a short interview to gather context, then produces a polished PRD .docx output.
---

# PRD Skill

Generates a professional **Product Requirement Document (PRD)** as a formatted `.docx` file.

---

## When You're Triggered

The user wants to write a PRD or formalize a product idea. Your job is to:

1. **Interview** — Gather the 5 PRD sections through targeted questions
2. **Generate** — Produce a polished, well-formatted `.docx` PRD

---

## Step 1: Interview the User

Run a short, conversational interview. Do NOT ask all questions at once — group them naturally.

Ask about these 5 sections:

### 1. Problem Statement
- What problem does this product solve?
- Why does it need to exist now?
- What's painful about the current situation?

### 2. Target Users
- Who is this for? (role, age range, tech-savviness, context)
- Who is NOT the target user? (optional but useful)
- Any specific user personas in mind?

### 3. Feature List
- What are the must-have (basic/MVP) features?
- What are the nice-to-have (advanced) features?
- Any features that are explicitly out of scope?

### 4. User Flow
- Walk through the step-by-step journey from user arrives → goal achieved
- What does the user do first? Then what?
- Any branching paths (e.g., logged in vs guest)?

### 5. Tech Preference *(Optional)*
- Any preferred stack, framework, or platform?
- Any constraints (mobile-only, browser extension, API-first, etc.)?
- If the user has no preference, skip this section

**Interview Tips:**
- If the user already provided details in their initial message, extract them and only ask for missing info
- Keep questions conversational, not form-like
- Confirm your understanding before generating: *"Here's what I have — does this look right?"*

---

## Step 2: Generate the PRD

Once you have all the info, use the **docx skill** to produce the `.docx` file.

### Read the docx skill first:
```
/mnt/skills/public/docx/SKILL.md
```

### Document Structure

Produce the PRD with this exact structure:

```
PRD: [Product Name]
Version 1.0 | [Date]
─────────────────────────────────────────

1. PROBLEM STATEMENT
   - The Problem
   - Why Now
   - Current Pain Points

2. TARGET USERS
   - Primary Users (with persona description)
   - Secondary Users (if any)
   - Out of Scope Users (if specified)

3. FEATURE LIST
   3.1 Basic / MVP Features        ← numbered list
   3.2 Advanced / Future Features  ← numbered list
   3.3 Out of Scope                ← if applicable

4. USER FLOW
   Step 1 → Step 2 → Step 3 → ...
   (written as a numbered walkthrough with branching noted)

5. TECH PREFERENCES                ← omit entire section if user has none
   - Stack / Framework
   - Constraints / Platform
   - Integrations

─────────────────────────────────────────
NOTES / OPEN QUESTIONS               ← add any gaps or assumptions here
```

### Formatting Rules

- **Title**: Large, bold — `PRD: [Product Name]`
- **Section headers**: Bold, uppercase, with horizontal rule separator
- **Feature lists**: Numbered within each tier (Basic 1.1, 1.2... Advanced 2.1, 2.2...)
- **User flow**: Numbered steps, with branching shown as sub-steps (4a, 4b)
- **Tech section**: Bullet list, omit entirely if not provided
- **Font**: Use a clean readable font (Calibri or similar)
- **Page margins**: Standard (1 inch)
- **Date**: Auto-fill with today's date

---

## Step 3: Output

- Save the `.docx` to `/mnt/user-data/outputs/PRD_[ProductName].docx`
- Use `present_files` to share it with the user
- End with a short summary: *"Your PRD is ready. Here's what's covered: [brief 2-line recap]."*

---

## Edge Cases

| Situation | Handling |
|---|---|
| User gives full info upfront | Skip the interview, confirm understanding, generate immediately |
| User is vague about features | Suggest a basic MVP feature set based on the problem type |
| No tech preferences | Omit section 5 entirely from the document |
| User wants markdown instead of .docx | Generate a `.md` file with the same structure |
| User wants to revise after seeing output | Accept edits and regenerate the specific section |

---

## Quality Checklist (before outputting)

- [ ] Problem statement is specific, not generic
- [ ] Target users are concrete (not just "everyone")
- [ ] Features are split into Basic vs Advanced tiers clearly
- [ ] User flow reads as a real journey, not a feature list
- [ ] Tech section is present only if the user specified preferences
- [ ] No placeholder text left in the document
- [ ] File is saved and presented via `present_files`
