---
name: feature-breakdown
description: >
  Use this skill whenever the user wants to create a Feature Breakdown Document — a structured Word document (.docx) that breaks down product features into their technical sub-components. Trigger this skill when the user mentions: 'feature breakdown', 'feature document', 'feature spec', 'feature list', 'break down features', 'technical breakdown', 'feature components', 'sub-features', 'system breakdown', or when they describe a feature and want it documented with its parts (e.g., "Login System → Email Validation, Password Encryption, OTP"). Also trigger when the user wants to document how any app feature works internally, list engineering tasks per feature, or produce a handoff doc for developers. Always use this skill for any feature decomposition deliverable that should be saved as a .docx file — even if the user just says "make a breakdown doc" or "document our features".
---

# Feature Breakdown Skill

This skill produces a polished **Feature Breakdown Document** (.docx) that decomposes product features into their technical sub-components, implementation notes, and acceptance criteria.

**Inspired by the example pattern:**
```
Login System
  ├── 1. Email Validation
  ├── 2. Password Encryption
  └── 3. OTP / Auth Flow
```

---

## Workflow

### Step 1: Gather Context (if not already provided)

Collect the following before writing:

| Info | Example |
|------|---------|
| Product/App name | "ShopEasy", "TaskFlow", etc. |
| List of top-level features | Login, Dashboard, Payments, Notifications… |
| Sub-components per feature | User provides or Claude infers from best practices |
| Tech stack (optional) | React + Node, Django, Flutter, etc. |
| Audience for the doc | Developers, PMs, stakeholders |

If enough context is in the user's message, skip straight to Step 2.

---

### Step 2: Plan the Feature Tree

Before writing, mentally (or explicitly) construct the feature tree:

```
Feature 1
  ├── Sub-feature 1.1
  ├── Sub-feature 1.2
  └── Sub-feature 1.3

Feature 2
  ├── Sub-feature 2.1
  └── Sub-feature 2.2
...
```

For each sub-feature include:
- **Description** — what it does in 1–2 sentences
- **Implementation Notes** — key technical considerations
- **Acceptance Criteria** — how to verify it's working correctly

---

### Step 3: Create the .docx Document

**Read the DOCX skill first:**
```
/mnt/skills/public/docx/SKILL.md
```

Install docx if needed:
```bash
npm install -g docx
```

#### Document Structure

```
Title Page
  - "Feature Breakdown Document"
  - Product name, version, date

1. Introduction
   - Purpose of this document
   - How to read it (feature → sub-feature hierarchy)

2. Feature Overview Table
   - Quick-reference table: Feature | # Sub-features | Priority | Status

3. Feature Breakdown (one section per feature)
   3.1 [Feature Name] (e.g., Login System)
       - Feature summary
       - Sub-feature table:
         | # | Sub-Feature | Description | Notes |
         |---|------------|-------------|-------|
         | 1 | Email Validation | ... | ... |
         | 2 | Password Encryption | ... | ... |
         | 3 | OTP/Auth Flow | ... | ... |
       - Acceptance Criteria (bulleted list)
       - Dependencies (other features this relies on)

   3.2 [Next Feature] ...
   (repeat for all features)

4. Implementation Notes & Tech Stack
   - Global notes on stack, frameworks, patterns

5. Open Questions / To-Do
   - Unresolved items, decisions pending

6. Revision History
   - Table: Version | Date | Author | Changes
```

---

### Step 4: Code the Document (Node.js)

Key patterns from the DOCX skill to apply:

**Imports:**
```javascript
const { Document, Packer, Paragraph, TextRun, Table, TableRow, TableCell,
        HeadingLevel, AlignmentType, BorderStyle, WidthType, ShadingType,
        LevelFormat, PageNumber, PageBreak } = require('docx');
```

**Feature Overview Table** — 4 columns: Feature | Sub-features | Priority | Status
- Header row: shading `#1E3A8A` fill color, white bold text (`ShadingType.CLEAR`)
- Alternating row shading for readability
- `columnWidths` must sum to content width (9360 DXA for US Letter, 1" margins)

**Sub-feature tables** — per feature, 4 columns: # | Sub-Feature | Description | Notes
- Header row with accent color (e.g., `#2563EB`)
- Use `margins: { top: 80, bottom: 80, left: 120, right: 120 }` on all cells
- Always use `WidthType.DXA`, never `WidthType.PERCENTAGE`

**Acceptance Criteria** — use `LevelFormat.BULLET` numbering config (never unicode bullets)

**Headings** — use `HeadingLevel.HEADING_1` for feature names, `HEADING_2` for subsections, always with `outlineLevel`

**Page size** — US Letter: `width: 12240, height: 15840` (DXA), 1-inch margins

---

### Step 5: Validate & Save

```bash
python scripts/office/validate.py feature_breakdown.docx
cp feature_breakdown.docx /mnt/user-data/outputs/Feature_Breakdown_Document.docx
```

Use `present_files` to deliver the file to the user.

---

## Quality Checklist

Before delivering, verify:
- [ ] Every top-level feature has its own section (3.x)
- [ ] Each section has a sub-feature table with at least description + notes columns
- [ ] Feature Overview Table at the top lists all features
- [ ] Acceptance Criteria listed for each feature
- [ ] Dependencies noted where applicable
- [ ] Document validates without errors
- [ ] File saved to `/mnt/user-data/outputs/`

---

## Built-in Feature Examples (use if user provides no sub-components)

### Login System
| # | Sub-Feature | Description | Notes |
|---|------------|-------------|-------|
| 1 | Email Validation | Validate format and existence of email on input | Use regex + DNS MX check |
| 2 | Password Encryption | Hash passwords before storage | bcrypt with salt rounds ≥ 12 |
| 3 | OTP / Auth Flow | Time-based OTP for 2FA, sent via email or SMS | TOTP (RFC 6238), 30s expiry |
| 4 | Session Management | Issue JWT or session cookie on successful login | Refresh token rotation |
| 5 | Forgot Password | Email-based reset link with expiry | Token valid for 15 minutes |

### User Dashboard
| # | Sub-Feature | Description |
|---|------------|-------------|
| 1 | Stats Cards | Key metrics at the top (users, revenue, activity) |
| 2 | Activity Feed | Recent actions/notifications list |
| 3 | Charts | Line/bar charts for trend data |
| 4 | Quick Actions | Shortcuts to common tasks |

### Payments
| # | Sub-Feature | Description |
|---|------------|-------------|
| 1 | Payment Gateway Integration | Stripe/Razorpay SDK integration |
| 2 | Card Tokenization | Store card token, never raw card data |
| 3 | Invoice Generation | Auto-generate PDF invoice on payment |
| 4 | Refund Flow | Initiate and track refunds |
| 5 | Webhook Handling | Handle payment success/failure events |

---

## Notes

- If the user provides a **feature list without sub-components**, infer the sub-components from industry best practices and note they are suggestions.
- If the user wants **Jira/Linear-style tasks** instead of a prose document, add a "Task Breakdown" column with short imperative task names (e.g., "Implement email regex validator").
- If the user specifies a **tech stack**, add stack-specific implementation notes (e.g., "Use Passport.js for auth middleware" for Node.js apps).
- Keep all descriptions **developer-actionable** — someone should be able to read this doc and know exactly what to build.
