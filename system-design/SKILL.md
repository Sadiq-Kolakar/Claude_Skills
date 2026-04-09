---
name: system-design
description: >
  Use this skill whenever the user asks to create, generate, write, or document a system design.
  Triggers include: "system design document", "architecture document", "technical design doc",
  "design my system", "document the architecture", "create a design doc", "write up the architecture",
  "API design doc", "database schema doc", "auth flow doc", or any combination of these.
  Also triggers when the user shares a project and says "document it", "write the design",
  or "make a system design for [project name]".
  Produces a professional .docx System Design Document covering all four pillars:
  Frontend + Backend Architecture, API Structure and Flow, Database Schema, and
  Authentication & Security Flow. Always use this skill — even if only one section is
  mentioned — because the full doc is almost always what the user actually wants.
---

# System Design Document Skill

Generates a professional **System Design Document** as a `.docx` file. The doc covers four mandatory sections regardless of how much info the user provides upfront. Gather what you can, infer the rest from the tech stack, and produce a complete, well-structured document.

---

## Step 1 — Gather Context

Before writing, collect answers to these. If the user already supplied them in the conversation, extract rather than ask. Only ask for what's missing and only if it's truly needed to write the doc.

| Info | Why it matters |
|------|----------------|
| **Project name** | Document title |
| **What the app does** | Narrative context for every section |
| **Frontend stack** (React, Vue, Flutter, etc.) | Section 1 |
| **Backend stack** (Node/Express, Django, Spring, etc.) | Section 1 |
| **Database(s)** (MongoDB, PostgreSQL, Redis, etc.) | Section 3 |
| **Auth method** (JWT, OAuth2, Session, Firebase Auth, etc.) | Section 4 |
| **Key entities / data models** | Section 3 |
| **Main API endpoints or features** | Section 2 |
| **Deployment target** (AWS, GCP, Vercel, etc.) — optional | Architecture diagram |

If the user gives almost nothing, default to a generic MERN stack example and label it as a template they should customise.

---

## Step 2 — Plan the Document

Before writing any code, mentally outline each of the four sections based on the gathered info. Ensure:

- Section 1 has a clear layered architecture diagram (ASCII or described in a styled block)  
- Section 2 has a table of key endpoints with method, path, request body, response  
- Section 3 has schema tables for each major entity  
- Section 4 has a numbered auth flow and a security checklist table  

---

## Step 3 — Generate the .docx

Use `docx` (npm) to produce the file. Always install first:

```bash
npm install -g docx
```

### Document Structure

```
Cover Page
  └── Project Name (H1)
  └── "System Design Document" subtitle
  └── Date

Table of Contents (auto)

1. Frontend + Backend Architecture
   1.1 Overview
   1.2 Architecture Diagram (ASCII art in a styled code block / monospace table)
   1.3 Frontend Details
   1.4 Backend Details
   1.5 Infrastructure & Deployment

2. API Structure and Flow
   2.1 Overview
   2.2 Base URL & Versioning
   2.3 Authentication Header Pattern
   2.4 Endpoints Table (Method | Path | Auth? | Description | Request Body | Response)
   2.5 Request / Response Flow Diagram

3. Database Schema
   3.1 Overview & Database Choice Rationale
   3.2 Entity Relationship Summary
   3.3 Schema Tables (one per major entity: Field | Type | Constraints | Description)
   3.4 Indexing Strategy

4. Authentication and Security Flow
   4.1 Auth Strategy Overview
   4.2 Registration Flow (numbered steps)
   4.3 Login Flow (numbered steps)
   4.4 Token Lifecycle (issue → verify → refresh → revoke)
   4.5 Security Checklist Table (Control | Status | Notes)
```

---

## Step 4 — Code Template

Use this as your base Node.js script. Fill in the actual content for the project.

```javascript
const {
  Document, Packer, Paragraph, TextRun, Table, TableRow, TableCell,
  HeadingLevel, AlignmentType, LevelFormat, BorderStyle, WidthType,
  ShadingType, PageNumber, TableOfContents, PageBreak
} = require('docx');
const fs = require('fs');

// ── Helpers ──────────────────────────────────────────────────────────────────

const BORDER = { style: BorderStyle.SINGLE, size: 1, color: "CCCCCC" };
const BORDERS = { top: BORDER, bottom: BORDER, left: BORDER, right: BORDER };
const HEADER_SHADE = { fill: "1F3864", type: ShadingType.CLEAR };
const ALT_SHADE    = { fill: "EDF2F8", type: ShadingType.CLEAR };

function h1(text) {
  return new Paragraph({ heading: HeadingLevel.HEADING_1, children: [new TextRun({ text, bold: true })] });
}
function h2(text) {
  return new Paragraph({ heading: HeadingLevel.HEADING_2, children: [new TextRun({ text })] });
}
function h3(text) {
  return new Paragraph({ heading: HeadingLevel.HEADING_3, children: [new TextRun({ text })] });
}
function body(text) {
  return new Paragraph({ children: [new TextRun({ text, size: 22 })] });
}
function spacer() {
  return new Paragraph({ children: [new TextRun("")] });
}
function bullet(text, ref = "bullets") {
  return new Paragraph({ numbering: { reference: ref, level: 0 }, children: [new TextRun({ text, size: 22 })] });
}
function numbered(text) {
  return bullet(text, "numbers");
}
function mono(text) {
  return new Paragraph({ children: [new TextRun({ text, font: "Courier New", size: 20 })] });
}
function pageBreak() {
  return new Paragraph({ children: [new PageBreak()] });
}

// Header cell (dark bg, white bold text)
function th(text, width) {
  return new TableCell({
    borders: BORDERS,
    shading: HEADER_SHADE,
    width: { size: width, type: WidthType.DXA },
    margins: { top: 80, bottom: 80, left: 120, right: 120 },
    children: [new Paragraph({ children: [new TextRun({ text, bold: true, color: "FFFFFF", size: 20 })] })]
  });
}

// Data cell (alternating shade optional)
function td(text, width, shade = false) {
  return new TableCell({
    borders: BORDERS,
    shading: shade ? ALT_SHADE : undefined,
    width: { size: width, type: WidthType.DXA },
    margins: { top: 80, bottom: 80, left: 120, right: 120 },
    children: [new Paragraph({ children: [new TextRun({ text, size: 20 })] })]
  });
}

function makeTable(headers, rows, colWidths) {
  const totalWidth = colWidths.reduce((a, b) => a + b, 0);
  return new Table({
    width: { size: totalWidth, type: WidthType.DXA },
    columnWidths: colWidths,
    rows: [
      new TableRow({ children: headers.map((h, i) => th(h, colWidths[i])), tableHeader: true }),
      ...rows.map((row, ri) =>
        new TableRow({ children: row.map((cell, ci) => td(cell, colWidths[ci], ri % 2 === 1)) })
      )
    ]
  });
}

// ── Document ──────────────────────────────────────────────────────────────────

const PROJECT_NAME = "YOUR_PROJECT_NAME"; // ← replace
const DATE = new Date().toLocaleDateString("en-US", { year: "numeric", month: "long", day: "numeric" });

const doc = new Document({
  styles: {
    default: { document: { run: { font: "Arial", size: 22 } } },
    paragraphStyles: [
      { id: "Heading1", name: "Heading 1", basedOn: "Normal", next: "Normal", quickFormat: true,
        run: { size: 36, bold: true, color: "1F3864", font: "Arial" },
        paragraph: { spacing: { before: 360, after: 120 }, outlineLevel: 0 } },
      { id: "Heading2", name: "Heading 2", basedOn: "Normal", next: "Normal", quickFormat: true,
        run: { size: 28, bold: true, color: "2E5FA3", font: "Arial" },
        paragraph: { spacing: { before: 240, after: 80 }, outlineLevel: 1 } },
      { id: "Heading3", name: "Heading 3", basedOn: "Normal", next: "Normal", quickFormat: true,
        run: { size: 24, bold: true, color: "444444", font: "Arial" },
        paragraph: { spacing: { before: 160, after: 60 }, outlineLevel: 2 } },
    ]
  },

  numbering: {
    config: [
      { reference: "bullets",
        levels: [{ level: 0, format: LevelFormat.BULLET, text: "•", alignment: AlignmentType.LEFT,
          style: { paragraph: { indent: { left: 720, hanging: 360 } } } }] },
      { reference: "numbers",
        levels: [{ level: 0, format: LevelFormat.DECIMAL, text: "%1.", alignment: AlignmentType.LEFT,
          style: { paragraph: { indent: { left: 720, hanging: 360 } } } }] },
    ]
  },

  sections: [{
    properties: {
      page: {
        size: { width: 12240, height: 15840 },
        margin: { top: 1440, right: 1440, bottom: 1440, left: 1440 }
      }
    },
    children: [

      // ── Cover ────────────────────────────────────────────────────────────
      spacer(), spacer(), spacer(),
      new Paragraph({ alignment: AlignmentType.CENTER, children: [new TextRun({ text: PROJECT_NAME, bold: true, size: 56, color: "1F3864" })] }),
      spacer(),
      new Paragraph({ alignment: AlignmentType.CENTER, children: [new TextRun({ text: "System Design Document", size: 36, color: "2E5FA3" })] }),
      spacer(),
      new Paragraph({ alignment: AlignmentType.CENTER, children: [new TextRun({ text: DATE, size: 22, color: "888888" })] }),
      pageBreak(),

      // ── Table of Contents ────────────────────────────────────────────────
      h1("Table of Contents"),
      new TableOfContents("Table of Contents", { hyperlink: true, headingStyleRange: "1-3" }),
      pageBreak(),

      // ════════════════════════════════════════════════════════════════════
      // SECTION 1 — Frontend + Backend Architecture
      // ════════════════════════════════════════════════════════════════════
      h1("1. Frontend + Backend Architecture"),

      h2("1.1 Overview"),
      body("Describe the high-level architecture of the system — what the main layers are and how they interact."),
      spacer(),

      h2("1.2 Architecture Diagram"),
      // Replace below with actual ASCII for the project
      mono("┌─────────────────────────────────────────┐"),
      mono("│              CLIENT LAYER                │"),
      mono("│   React SPA / Mobile App (Flutter)       │"),
      mono("└──────────────────┬──────────────────────┘"),
      mono("                   │ HTTPS / REST / WS"),
      mono("┌──────────────────▼──────────────────────┐"),
      mono("│              API GATEWAY                 │"),
      mono("│   Rate Limiting · Auth Middleware · CORS │"),
      mono("└──────┬──────────────────┬───────────────┘"),
      mono("       │                  │"),
      mono("┌──────▼──────┐   ┌───────▼───────┐"),
      mono("│  Auth Svc   │   │  Core API Svc  │"),
      mono("│  (JWT/OAuth)│   │  (Business Lg) │"),
      mono("└──────┬──────┘   └───────┬────────┘"),
      mono("       │                  │"),
      mono("┌──────▼──────────────────▼──────────────┐"),
      mono("│              DATABASE LAYER              │"),
      mono("│  Primary DB (Postgres/Mongo) · Cache     │"),
      mono("└─────────────────────────────────────────┘"),
      spacer(),

      h2("1.3 Frontend Details"),
      bullet("Framework: REPLACE_ME (e.g. React 19 + TypeScript + Vite)"),
      bullet("State Management: REPLACE_ME (e.g. Zustand / Redux Toolkit)"),
      bullet("Routing: REPLACE_ME (e.g. React Router v6)"),
      bullet("UI Library: REPLACE_ME (e.g. shadcn/ui + Tailwind CSS)"),
      bullet("Build / Deploy: REPLACE_ME (e.g. Vercel / GitHub Actions)"),
      spacer(),

      h2("1.4 Backend Details"),
      bullet("Runtime: REPLACE_ME (e.g. Node.js 20 + Express)"),
      bullet("Architecture Pattern: REPLACE_ME (e.g. MVC / Layered / Microservices)"),
      bullet("Key Middleware: REPLACE_ME (e.g. cors, helmet, morgan, express-validator)"),
      bullet("Background Jobs: REPLACE_ME (e.g. BullMQ / cron)"),
      bullet("File Storage: REPLACE_ME (e.g. AWS S3 / Cloudinary)"),
      spacer(),

      h2("1.5 Infrastructure & Deployment"),
      bullet("Hosting: REPLACE_ME"),
      bullet("CI/CD: REPLACE_ME"),
      bullet("Environment Config: .env + dotenv / secrets manager"),
      bullet("Monitoring: REPLACE_ME (e.g. Sentry / Datadog)"),
      spacer(),
      pageBreak(),

      // ════════════════════════════════════════════════════════════════════
      // SECTION 2 — API Structure and Flow
      // ════════════════════════════════════════════════════════════════════
      h1("2. API Structure and Flow"),

      h2("2.1 Overview"),
      body("All API communication follows REST conventions over HTTPS. Responses are JSON. Errors use standard HTTP status codes."),
      spacer(),

      h2("2.2 Base URL & Versioning"),
      mono("Production : https://api.yourproject.com/v1"),
      mono("Staging    : https://api-staging.yourproject.com/v1"),
      spacer(),

      h2("2.3 Authentication Header"),
      mono("Authorization: Bearer <access_token>"),
      spacer(),

      h2("2.4 Endpoints"),
      // Endpoint table — replace rows with actual endpoints
      makeTable(
        ["Method", "Path", "Auth", "Description"],
        [
          ["POST", "/auth/register", "No",  "Register a new user"],
          ["POST", "/auth/login",    "No",  "Login, receive JWT"],
          ["POST", "/auth/refresh",  "No",  "Refresh access token"],
          ["GET",  "/users/me",      "Yes", "Get current user profile"],
          ["PUT",  "/users/me",      "Yes", "Update profile"],
          ["GET",  "/resource",      "Yes", "List resources (paginated)"],
          ["POST", "/resource",      "Yes", "Create a resource"],
          ["GET",  "/resource/:id",  "Yes", "Get single resource"],
          ["PUT",  "/resource/:id",  "Yes", "Update resource"],
          ["DELETE","/resource/:id", "Yes", "Delete resource"],
        ],
        [1200, 2500, 800, 4860]
      ),
      spacer(),

      h2("2.5 Request / Response Flow"),
      numbered("Client builds request with headers and JSON body"),
      numbered("API Gateway validates rate limit and CORS"),
      numbered("Auth middleware verifies JWT signature and expiry"),
      numbered("Route handler validates request body (schema validation)"),
      numbered("Controller delegates to service layer (business logic)"),
      numbered("Service interacts with repository / database"),
      numbered("Response serialized and returned with appropriate HTTP status"),
      spacer(),

      h3("Standard Success Response"),
      mono('{ "success": true, "data": { ... }, "meta": { "page": 1, "total": 42 } }'),
      spacer(),
      h3("Standard Error Response"),
      mono('{ "success": false, "error": { "code": "VALIDATION_ERROR", "message": "..." } }'),
      spacer(),
      pageBreak(),

      // ════════════════════════════════════════════════════════════════════
      // SECTION 3 — Database Schema
      // ════════════════════════════════════════════════════════════════════
      h1("3. Database Schema"),

      h2("3.1 Overview"),
      body("Describe the choice of database and rationale (e.g. relational vs document vs hybrid)."),
      spacer(),

      h2("3.2 Entity Relationship Summary"),
      bullet("User — owns — many Resources"),
      bullet("Resource — has — many Tags"),
      bullet("Session — belongs to — User"),
      body("(Replace with your actual ER relationships.)"),
      spacer(),

      // ── Users Table ──
      h2("3.3 Schema: Users"),
      makeTable(
        ["Field", "Type", "Constraints", "Description"],
        [
          ["_id / id",     "ObjectId / UUID",   "PK, auto",           "Unique user identifier"],
          ["name",         "String",             "required, max:100",  "Full display name"],
          ["email",        "String",             "required, unique",   "Login email address"],
          ["passwordHash", "String",             "required",           "bcrypt hash (never plain text)"],
          ["role",         "Enum",               "user | admin",       "Access role"],
          ["isVerified",   "Boolean",            "default: false",     "Email verified flag"],
          ["createdAt",    "DateTime",           "auto",               "Account creation timestamp"],
          ["updatedAt",    "DateTime",           "auto",               "Last update timestamp"],
        ],
        [1600, 1600, 2000, 4160]
      ),
      spacer(),

      // ── Resource Table (template — clone per entity) ──
      h2("3.4 Schema: Resource (Template)"),
      makeTable(
        ["Field", "Type", "Constraints", "Description"],
        [
          ["_id / id",  "ObjectId / UUID", "PK, auto",          "Unique resource identifier"],
          ["userId",    "ObjectId / UUID", "FK → Users, index", "Owner reference"],
          ["title",     "String",          "required, max:200", "Resource title"],
          ["status",    "Enum",            "active | archived", "Lifecycle status"],
          ["metadata",  "JSON / Object",   "optional",          "Flexible extra fields"],
          ["createdAt", "DateTime",        "auto",              "Creation timestamp"],
        ],
        [1600, 1600, 2000, 4160]
      ),
      spacer(),

      h2("3.5 Indexing Strategy"),
      makeTable(
        ["Collection", "Index Field(s)", "Type", "Reason"],
        [
          ["Users",     "email",              "Unique",   "Fast login lookup"],
          ["Users",     "createdAt",          "Ascending","Time-series queries"],
          ["Resources", "userId",             "Ascending","Owner filter"],
          ["Resources", "userId + createdAt", "Compound", "Paginated owner list"],
        ],
        [2000, 2500, 1500, 3360]
      ),
      spacer(),
      pageBreak(),

      // ════════════════════════════════════════════════════════════════════
      // SECTION 4 — Authentication and Security Flow
      // ════════════════════════════════════════════════════════════════════
      h1("4. Authentication and Security Flow"),

      h2("4.1 Auth Strategy"),
      body("Strategy: JWT (Access + Refresh token pair). Access tokens are short-lived (15 min). Refresh tokens are long-lived (7 days) and stored server-side for revocation support."),
      spacer(),

      h2("4.2 Registration Flow"),
      numbered("User submits name, email, and password"),
      numbered("Server validates input (format, uniqueness of email)"),
      numbered("Password hashed with bcrypt (cost factor 12)"),
      numbered("User record created with isVerified: false"),
      numbered("Verification email sent with signed link (expires 24h)"),
      numbered("User clicks link → email verified → account active"),
      spacer(),

      h2("4.3 Login Flow"),
      numbered("User submits email + password"),
      numbered("Server looks up user by email"),
      numbered("bcrypt.compare() validates password against stored hash"),
      numbered("On success: generate Access Token (JWT, 15 min) + Refresh Token (opaque, 7 days)"),
      numbered("Refresh Token saved to DB (hashed) and sent as HttpOnly Secure cookie"),
      numbered("Access Token returned in response body — client stores in memory only"),
      spacer(),

      h2("4.4 Token Lifecycle"),
      makeTable(
        ["Event", "Action", "Token Affected"],
        [
          ["Login",           "Issue Access + Refresh tokens",              "Both"],
          ["API Request",     "Validate Access Token signature & expiry",   "Access"],
          ["Access Expired",  "POST /auth/refresh with cookie",             "Access (new)"],
          ["Logout",          "Delete Refresh Token from DB, clear cookie", "Both"],
          ["Password Change", "Revoke all Refresh Tokens for user",         "Refresh (all)"],
        ],
        [2500, 4300, 2560]
      ),
      spacer(),

      h2("4.5 Security Checklist"),
      makeTable(
        ["Control", "Status", "Notes"],
        [
          ["HTTPS enforced (TLS 1.2+)",          "✅ Implemented", "Force redirect at gateway"],
          ["Passwords hashed (bcrypt, cost 12)",  "✅ Implemented", "Never store plain text"],
          ["JWT signed (RS256 or HS256)",         "✅ Implemented", "Rotate signing secret regularly"],
          ["HttpOnly Secure cookie for RT",       "✅ Implemented", "Prevents JS access"],
          ["CORS restricted to allowed origins",  "✅ Implemented", "allowedOrigins whitelist"],
          ["Rate limiting on auth endpoints",     "✅ Implemented", "5 req/min per IP on /login"],
          ["Input validation & sanitization",     "✅ Implemented", "express-validator / zod"],
          ["SQL/NoSQL injection prevention",      "✅ Implemented", "Parameterised queries / ODM"],
          ["Security headers (helmet.js)",        "✅ Implemented", "CSP, X-Frame, HSTS etc."],
          ["Secrets in env vars (not in code)",   "✅ Implemented", ".env + secret manager"],
          ["Dependency vulnerability scan",       "⚠ In Progress", "npm audit in CI pipeline"],
          ["Penetration testing",                 "⏳ Planned",     "Scheduled pre-launch"],
        ],
        [3500, 1800, 4060]
      ),
      spacer(),

    ]
  }]
});

// Write output
Packer.toBuffer(doc).then(buffer => {
  fs.writeFileSync("System_Design_Document.docx", buffer);
  console.log("✅  System_Design_Document.docx created");
});
```

---

## Step 5 — Output

```bash
node generate.js
```

Then move to outputs:

```bash
cp System_Design_Document.docx /mnt/user-data/outputs/System_Design_Document.docx
```

Present the file to the user with `present_files`.

---

## Customisation Guide

When filling in the template for a real project, replace every `REPLACE_ME` with the actual values and:

- **Section 1**: Update the ASCII diagram to match the real topology (monorepo? microservices? edge CDN?)
- **Section 2**: Replace the endpoint table rows with the actual API surface. Add sub-sections for WebSocket events if needed.
- **Section 3**: Add one `h2` + `makeTable` block per major database entity. Delete the template entity.
- **Section 4**: Update the auth strategy blurb if using OAuth2, Firebase Auth, Supabase, Clerk, etc. Adjust the token lifecycle table accordingly.

---

## Quality Bar

Before presenting the file, verify:

- [ ] Cover page shows correct project name and today's date
- [ ] All four sections are present and populated (no `REPLACE_ME` left)
- [ ] Tables render without black backgrounds (`ShadingType.CLEAR` used)
- [ ] Endpoint table covers all major routes
- [ ] Schema tables cover all major entities
- [ ] Security checklist reflects the actual tech choices
- [ ] File opens in Word / LibreOffice without errors
