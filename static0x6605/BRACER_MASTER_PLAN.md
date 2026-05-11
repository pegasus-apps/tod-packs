# Bracer — Master Build Plan
> Define your structure. Fill your content. Export your JSON.

A free, open-source cross-platform tool for creating and managing structured JSON content packs. Built for indie app developers, game makers, and content creators who need a clean UI to manage JSON-driven content without touching code.

---

## Table of Contents

1. [Product Vision](#1-product-vision)
2. [Final UX Specification](#2-final-ux-specification)
3. [Tech Stack](#3-tech-stack)
4. [Repo & Project Setup Instructions](#4-repo--project-setup-instructions)
5. [Build Phases & Agent Prompts](#5-build-phases--agent-prompts)
6. [Release Checklist](#6-release-checklist)
7. [README Templates](#7-readme-templates)

---

## 1. Product Vision

### What It Is
Bracer is a schema-first JSON content editor. Users define the structure of their data once, then fill it in with a clean form UI — no code required. Think of it as a lightweight CMS you own entirely, with no backend, no database, and no subscription.

### Who It's For
- Mobile/indie app developers who store content in JSON packs (card games, quiz apps, flashcard apps)
- Game developers managing items, dialogue, quests, or level data in JSON
- Non-technical content creators who collaborate with developers on JSON-driven apps

### Core Principles
- **Fast** — open a project, add an entry, done in under 10 seconds
- **Obvious** — zero learning curve, no docs needed
- **Trustworthy** — user always knows where their data is and what version they're on
- **No lock-in** — everything is a plain JSON file the user owns

### What It Does NOT Do
- No user accounts or cloud database (sync is through the user's own Drive or GitHub)
- No real-time collaboration
- No AI generation (planned for later)
- No backend of any kind

---

## 2. Final UX Specification

### Design Language
- Feels like **Notion meets Linear** — clean, spacious, confident
- Light and dark mode
- One accent color used sparingly (indigo)
- No icons without labels
- Mobile uses bottom navigation bar
- Desktop uses left sidebar navigation
- Typography: `Inter` for UI, `JetBrains Mono` for JSON preview

---

### Schema Data Model

Every project has two layers:

**Layer 1 — Root Fields (Metadata)**
Fields that exist once at the top level of the JSON. Set once, rarely changed.
Example: `pack_id`, `name`, `description`, `asset_path`, `ads_required`

**Layer 2 — List Fields (Arrays)**
Arrays of structured objects the user adds to repeatedly.
Example: `cards: [ {}, {}, {} ]`

Each list field has its own sub-schema (the shape of each item in the array).

**Supported Field Types:**

| Type | Description | Example |
|---|---|---|
| `short-text` | Single line string | `pack_id`, `name` |
| `long-text` | Multi-line textarea | `description`, `content` |
| `number` | Integer or float | `level`, `ads_required` |
| `number-nullable` | Number or null | `timer_seconds` |
| `toggle` | Boolean true/false | `repeatable`, `auto_start_timer` |
| `dropdown` | Enum with user-defined options | `type: truth/dare`, `gender: any/male/female` |
| `tags` | Array of strings | `topic_tags` |
| `object-optional` | Nullable nested object | `follow_up` (null or `{id, content, ...}`) |
| `list` | Array of sub-items | `cards` |

---

### Screen 1 — Home (Project Dashboard)

**Purpose:** Entry point. See all projects, create new ones, understand sync status at a glance.

**Layout:**
```
┌─────────────────────────────────────────────────┐
│  Bracer                           [+ New Project]│
│                                                 │
│  ┌───────────────┐   ┌───────────────┐          │
│  │ 🃏 Couple Game │   │ 💬 Chat Tool  │          │
│  │               │   │               │          │
│  │ 2 schemas     │   │ 1 schema      │          │
│  │ 48 entries    │   │ 12 entries    │          │
│  │               │   │               │          │
│  │ ● Synced      │   │ ○ Local only  │          │
│  │ 2 mins ago    │   │               │          │
│  └───────────────┘   └───────────────┘          │
│                                                 │
│  ──────────────────────────────────────────     │
│  [Connect Google Drive]  [Connect GitHub]       │
│  or  Continue without account →                 │
└─────────────────────────────────────────────────┘
```

**Rules:**
- Both sync options are optional and equal in weight — no guilt-tripping
- User can connect Drive, GitHub, both, or neither
- Sync status shown on every card: `● Synced`, `↑ Syncing`, `⚠ Conflict`, `○ Local only`, `✕ Sync error`
- Each project card shows which sync backend it uses (Drive icon or GitHub icon)
- Guest users see subtle persistent reminder: *"Export your JSON regularly to avoid data loss"*
- Right-click / long-press on project card: Rename, Duplicate, Delete, Export all

---

### Screen 2 — Project View

**Purpose:** See all schemas inside a project. Navigate into a schema to edit content or edit the schema definition.

**Layout:**
```
┌─────────────────────────────────────────────────┐
│  ← Home   Couple Game              v4  ● Synced │
│                                                 │
│  SCHEMAS                                        │
│  ┌──────────────────┐   ┌──────────────────┐    │
│  │ 🃏 Truth Cards    │   │ 🔥 Dare Cards     │    │
│  │ 32 entries       │   │ 16 entries       │    │
│  │                  │   │                  │    │
│  │ [Open] [Edit]    │   │ [Open] [Edit]    │    │
│  └──────────────────┘   └──────────────────┘    │
│                                                 │
│  [+ New Schema]   [Copy from ▾]                 │
│                    ├── This project             │
│                    │    ├── Truth Cards         │
│                    │    └── Dare Cards          │
│                    └── Other projects           │
│                         └── Chat Tool → Msgs   │
│                                                 │
│  ──────────────────────────────────────────     │
│  Export entire project   [↓ Download JSON]      │
└─────────────────────────────────────────────────┘
```

**Rules:**
- Version number always visible top-right
- Two CTAs per schema: **Open** (go to content editor) and **Edit** (go to schema definition)
- Copy schema dialog: pick source → enter new name → creates a copy with no entries
- Copying across projects is supported
- Export entire project = one JSON file containing all schemas and their entries

---

### Screen 3 — Schema Editor

**Purpose:** Define the shape of data. This screen is the foundation — get it right and the content editor works perfectly.

**Two entry paths:**

**Path A — Start blank (default):**
```
┌─────────────────────────────────────────────────┐
│  ← Couple Game    Truth Cards — Schema          │
│                                                 │
│  Schema name:  [Truth Cards                  ]  │
│                                                 │
│  [+ Import from example JSON]  ← collapsed      │
│                                                 │
│  ROOT FIELDS  (metadata, set once)              │
│  ┌──────────────────────────────────────────┐   │
│  │ ⠿ pack_id     [Short Text ▾]  [Req ✓] × │   │
│  │ ⠿ name        [Short Text ▾]  [Req ✓] × │   │
│  │ ⠿ description [Long Text  ▾]  [Req  ] × │   │
│  │ ⠿ ads_required[Number     ▾]  [Req  ] × │   │
│  └──────────────────────────────────────────┘   │
│  [+ Add Root Field]                             │
│                                                 │
│  LIST FIELDS  (arrays you'll add items to)      │
│  ┌──────────────────────────────────────────┐   │
│  │ cards  →  [Define card shape]            │   │
│  │   ⠿ id           [Short Text  ▾] [Req ✓]│   │
│  │   ⠿ type         [Dropdown    ▾] [Req ✓]│   │
│  │              Options: truth, dare        │   │
│  │              [+ add option]              │   │
│  │   ⠿ content      [Long Text   ▾] [Req ✓]│   │
│  │   ⠿ level        [Number      ▾] [Req ✓]│   │
│  │   ⠿ topic_tags   [Tags        ▾]         │   │
│  │   ⠿ repeatable   [Toggle      ▾]         │   │
│  │   ⠿ timer_seconds[Number Null ▾]         │   │
│  │   ⠿ follow_up    [Object Opt. ▾]         │   │
│  │     └── content     [Long Text  ▾]      │   │
│  │     └── probability [Number     ▾]      │   │
│  └──────────────────────────────────────────┘   │
│  [+ Add List Field]                             │
│                                                 │
│                            [Save Schema →]      │
└─────────────────────────────────────────────────┘
```

**Path B — Import from example JSON (collapsed by default):**
After detection, fields appear pre-filled. User confirms type for each, adds enum options to dropdowns, marks required fields. The example JSON is a **hint only** — user has full control.

**Field detection logic:**
- `string` with ≤ 5 unique values across sample → suggest Dropdown
- `string` with avg length > 60 chars → suggest Long Text
- `string` short → Short Text
- `number` → Number
- `boolean` → Toggle
- `array of strings` → Tags
- `null or object` → Object Optional
- `array of objects` → List

**Rules:**
- Fields are drag-to-reorder
- Dropdown fields show inline chip editor — type and hit Enter to add options
- Object Optional fields expand to show sub-fields inline
- Schema changes warn user if existing entries will be affected
- Save is disabled until schema has a name and at least one field

---

### Screen 4 — Content Editor

**Purpose:** The daily driver. Add entries fast. The form is the hero, the list is secondary.

**Layout:**
```
┌─────────────────────────────────────────────────┐
│  ← Couple Game    Truth Cards      32 entries   │
│                                                 │
│  Currently editing: [cards              ▾]      │
│                      Pack metadata              │
│                    ▸ cards                      │
│                                                 │
│  ════ ADD ENTRY ══════════════════════════════  │
│  id            [nau_t_033                    ]  │
│  type          [truth              ▾]           │
│  content       [                             ]  │
│  level         [2    ]                          │
│  topic_tags    [attraction ×] [flirty ×] [+]   │
│  repeatable    [✓]                              │
│  timer_seconds [null ]                          │
│  follow_up     [ ] Add follow-up               │
│                                                 │
│  [Save Entry]  ← clears form, stays on screen  │
│                                                 │
│  ════ 32 ENTRIES ════════════ [↓ Export JSON]  │
│  Search...                   [Filter: All ▾]   │
│                                                 │
│  nau_t_001  truth  lv1  What's the most...  ⋮  │
│  nau_t_002  truth  lv2  Have you ever had.. ⋮  │
│                                                 │
│  [See all entries →]    [Sync ↑]               │
└─────────────────────────────────────────────────┘
```

**Critical UX rules:**
- Form always visible at the top — never hidden behind a sheet or modal
- [Save Entry] clears the form instantly — conveyor belt pattern
- Last saved entry briefly highlights at top of list (300ms flash) as confirmation
- Clicking an entry in the list opens it inline for editing
- Context dropdown switches between Pack metadata and each list field
- Import existing JSON → parses and appends entries to the list

---

### Sync Model — Two Backends

Users choose their sync backend per project (or none). The sync interface is identical regardless of backend.

---

#### Google Drive Sync

**Best for:** Non-technical users, cross-device access, content teams

**How it works:**
- Sign in with Google OAuth
- Bracer creates a `Bracer/` root folder in their Drive
- Each project = a subfolder with `project.meta.json`, `schemas/`, and `content/`
- Every save uploads the changed files silently

**Drive folder structure:**
```
Bracer/
├── Couple Game/
│   ├── project.meta.json
│   ├── schemas/
│   │   ├── truth-cards.schema.json
│   │   └── dare-cards.schema.json
│   └── content/
│       ├── truth-cards.json
│       └── dare-cards.json
└── Chat Tool/
    └── ...
```

---

#### GitHub Sync

**Best for:** Developers, version history, projects where JSON lives in a code repo

**How it works:**
- Sign in with GitHub OAuth
- User picks an existing repo (or Bracer creates a new one)
- User picks a folder path inside that repo (e.g. `content/packs/`)
- Every save = a commit to that repo with a descriptive message
- Full history on GitHub — revert to any point

**Commit message format:**
```
bracer: add entry to truth-cards (nau_t_033)
bracer: update schema truth-cards
bracer: update pack metadata
```

**GitHub repo structure:**
```
your-game-repo/
└── content/packs/          ← user-chosen folder
    ├── .bracer/
    │   └── project.meta.json
    ├── schemas/
    │   ├── truth-cards.schema.json
    │   └── dare-cards.schema.json
    └── truth-cards.json
```

**Why this is powerful for developers:**
- JSON content lives in the same repo as the game/app code
- Every content change is tracked in Git history alongside code changes
- Team members can see exactly what content changed and when
- No separate tool needed — Bracer commits directly into the workflow they already use

**GitHub API used:** GitHub Contents API (REST) — no Git installation required on the user's machine

---

#### Conflict Resolution (same for both backends)

Never auto-merge. Always show the user:

```
⚠️  Sync conflict on "Couple Game"

  This device:    Version 4  (edited 2 mins ago)
  Remote:         Version 6  (edited on another device)

  [Keep mine]   [Use remote version]   [Compare side by side]
```

---

#### Sync Status Indicators

| Status | Display |
|---|---|
| `● Synced` | "Synced 2 mins ago" |
| `↑ Syncing` | "Uploading..." |
| `⚠ Conflict` | "Edited on 2 devices — tap to resolve" |
| `○ Local only` | "Not connected to sync" |
| `✕ Error` | "Sync failed — tap to retry" |

---

### project.meta.json

```json
{
  "name": "Couple Game",
  "created": "2026-05-10",
  "lastModified": "2026-05-10T14:32:00Z",
  "lastModifiedBy": "device-web-chrome",
  "version": 4,
  "syncBackend": "drive",
  "schemas": ["truth-cards", "dare-cards"]
}
```

`syncBackend` is `"drive"`, `"github"`, or `"local"`.

---

### Mobile Adaptations
- Bottom tab bar: Projects | Recents | Settings
- Content editor: form on top, list scrolls below — same layout, larger tap targets
- Schema editor: one field at a time, accordion style
- Sync status in top bar always visible
- Add to home screen as PWA (no App Store needed for v1)

---

## 3. Tech Stack

| Layer | Choice | Reason |
|---|---|---|
| Framework | React 18 + Vite | Fast, great ecosystem |
| Styling | Tailwind CSS | Utility-first, consistent |
| State | Zustand | Lightweight, no boilerplate |
| Local storage | localStorage + IndexedDB | localStorage for settings, IndexedDB for content |
| Desktop | **Tauri v2** | Same React code, ~5MB app vs ~150MB Electron, fast, no Chromium bloat |
| Drive sync | Google Drive API v3 + Picker API | Free, well documented |
| GitHub sync | GitHub Contents API (REST) | No Git required, free, full history |
| Auth | Google OAuth 2.0 + GitHub OAuth | Separate flows, separate scopes |
| Deploy (web) | Vercel | Free forever at this scale |
| Package manager | pnpm | Faster than npm |
| Language | TypeScript | Catch schema shape errors early |

### Why Tauri over Electron

- **App size:** ~5MB (Tauri) vs ~150MB (Electron)
- **Memory:** Tauri uses the OS webview — no bundled Chromium
- **Speed:** Opens faster, uses less RAM
- **Cost:** You don't need to write Rust — Tauri handles everything, you write React
- **Only downside:** Rust must be installed on your dev machine, and compile times are slower
- **Verdict:** Worth it. A 5MB download signals quality. Users notice.

---

## 4. Repo & Project Setup Instructions

### Step 1 — Create GitHub Repository

1. Go to github.com → New Repository
2. Name it: `bracer`
3. Description: `Schema-first JSON content editor for indie developers and content creators`
4. Set to **Public**
5. Add README: **Yes**
6. Add .gitignore: **Node**
7. License: **MIT**
8. Click Create Repository

### Step 2 — Clone Locally

```bash
git clone https://github.com/YOUR_USERNAME/bracer.git
cd bracer
```

### Step 3 — Create Folder Structure

```bash
mkdir -p apps/web apps/desktop packages/core docs
```

Your repo structure:
```
bracer/
├── apps/
│   ├── web/          ← React + Vite web app (also PWA)
│   └── desktop/      ← Tauri wrapper
├── packages/
│   └── core/         ← Shared logic (schema parser, JSON builder, sync services)
├── docs/
├── .github/
│   └── workflows/
├── BRACER_MASTER_PLAN.md
└── README.md
```

### Step 4 — Initialize the Web App

```bash
cd apps/web
pnpm create vite . --template react-ts
pnpm install
pnpm add -D tailwindcss postcss autoprefixer
pnpm add zustand react-router-dom
npx tailwindcss init -p
```

### Step 5 — Set Up Google Cloud Project (for Drive)

1. Go to console.cloud.google.com → New Project → Name: `Bracer`
2. Enable: Google Drive API, Google Picker API
3. Create OAuth 2.0 credentials → Web application
4. Authorized origins: `http://localhost:5173`, `https://bracer.vercel.app`
5. Save your `CLIENT_ID`

### Step 6 — Set Up GitHub OAuth App (for GitHub sync)

1. Go to github.com → Settings → Developer settings → OAuth Apps → New OAuth App
2. Name: `Bracer`
3. Homepage URL: `https://bracer.vercel.app`
4. Authorization callback URL: `https://bracer.vercel.app/auth/github/callback`
5. Save your `Client ID` and `Client Secret`
6. Note: GitHub OAuth requires a backend for the token exchange step (client secret cannot be in frontend). Use a Vercel serverless function for this — one endpoint only.

### Step 7 — Environment Variables

Create `apps/web/.env`:
```
VITE_GOOGLE_CLIENT_ID=your_google_oauth_client_id
VITE_GITHUB_CLIENT_ID=your_github_oauth_client_id
VITE_GITHUB_REDIRECT_URI=http://localhost:5173/auth/github/callback
```

Create `apps/web/.env.production`:
```
VITE_GOOGLE_CLIENT_ID=your_google_oauth_client_id
VITE_GITHUB_CLIENT_ID=your_github_oauth_client_id
VITE_GITHUB_REDIRECT_URI=https://bracer.vercel.app/auth/github/callback
```

### Step 8 — Initial Commit

```bash
cd ../../
git add .
git commit -m "chore: initial project structure"
git push origin main
```

---

## 5. Build Phases & Agent Prompts

---

> **How to use these prompts:**
> - `[YOU]` — manual steps you do yourself
> - `🤖 CLAUDE — CRITICAL` — hard logic, architecture, core systems. **Start prompt with "Be concise. No lengthy explanations. Output code only."** to save tokens
> - `🤖 GEMINI — BUILD` — UI construction built around Claude's foundations
> - After each prompt: test, then `git commit` before moving on
> - ✅ = completed

---

### PHASE 1 — Foundation & Design System ✅

#### Prompt 1.1 🤖 CLAUDE — CRITICAL ✅
Design system, Tailwind config, base UI components, ThemeProvider, App router, base Zustand store.

#### Prompt 1.2 🤖 CLAUDE — CRITICAL ✅
Core TypeScript types, schemaInference.ts, jsonBuilder.ts in packages/core.

---

### PHASE 2 — Project Dashboard ✅

#### Prompt 2.1 🤖 GEMINI — BUILD ✅
Project dashboard UI with mock data, project cards, sync status badges, Google auth strip.

#### Prompt 2.2 🤖 CLAUDE — CRITICAL ✅
Zustand store with localStorage persistence, all project/schema/entry CRUD actions.

---

### PHASE 3 — Schema Editor

#### Prompt 3.1 🤖 CLAUDE — CRITICAL ✅
Schema editor core: drag-to-reorder fields, all field types, dropdown chip editor, object-optional sub-fields, JSON inference integration.

---

#### ✅ YOU ARE HERE → Prompt 3.2 🤖 GEMINI — BUILD ✅
Schema editor polish: field row cards, type icons, dropdown chips, sub-field indentation, collapsible JSON import, empty states, sticky save button.

---

### PHASE 4 — Content Editor

---

#### Prompt 4.1 🤖 CLAUDE — CRITICAL
*Tag: Content Editor Core — Conveyor Belt Pattern*

```
Be concise. No lengthy explanations. Output code only.

Build the Content Editor page for Bracer.

File: apps/web/src/pages/ContentEditor.tsx

Core UX: form on top always visible, list below. Save clears form instantly. No sheets or modals for adding — conveyor belt pattern.

TOP SECTION — Context switcher:
- Dropdown: "Currently editing: [cards ▾]"
- Options: "Pack metadata" + each list field name from schema.listFields
- Pack metadata mode: simple field editor for rootFields values (one input per field, [Save Changes] button)
- List field mode: shows ADD ENTRY form

ADD ENTRY FORM:
- One input per sub-field of the selected list field
- Input type matches FieldType:
  - short-text → Input
  - long-text → Textarea
  - number → number input
  - number-nullable → number input (empty = null, show "null" placeholder)
  - toggle → Toggle
  - dropdown → Dropdown with schema's defined options
  - tags → TagInput
  - object-optional → toggle to enable, expands sub-fields inline when enabled
- [Save Entry] below form
- On save: call addEntry from projectStore, clear all fields, focus first field, trigger list flash

ENTRY LIST:
- Search input (filters by any string field value, case-insensitive)
- Filter dropdown (filters by first dropdown field in schema)
- Compact rows: first 3 fields as preview columns, ⋮ menu right
- ⋮ menu: Edit, Duplicate, Delete
- Edit mode: loads entry into the form area (replaces add form), title changes to "Edit Entry", shows [Save Changes] and [Cancel]
- After save edit: return to add mode, clear form
- Last added entry: briefly highlight top of list with indigo flash (300ms CSS keyframe animation)
- Entry count: "32 entries" shown above list
- [↓ Export JSON] button: calls buildJSON from packages/core, triggers file download named after schema

IMPORT EXISTING JSON:
- [↑ Import JSON] button near top
- User uploads or pastes JSON
- Calls inferSchemaFromJSON, compares to current schema
- If compatible: appends entries to existing list (does not overwrite)
- If incompatible: shows inline warning with field mismatch details

Wire all data operations to projectStore.
Use the Project, Schema, Entry types from packages/core/src/types.ts.
```

---

**[YOU]** After Prompt 4.1:
- Add 10 entries fast — confirm form clears instantly each time
- Edit an entry from the list
- Export JSON — open file, confirm structure matches your original naughty pack format
- `git commit -m "feat: content editor core conveyor belt"`

---

#### Prompt 4.2 🤖 GEMINI — BUILD
*Tag: Content Editor Polish + Mobile*

```
Polish ContentEditor.tsx in the Bracer React app.

Improvements:
1. ADD ENTRY section: subtle top accent border (indigo), section label "ADD ENTRY" in small-caps muted text above
2. Save button: large full-width on mobile, fixed to bottom on mobile screens
3. Entry list rows: alternating subtle background, smooth hover, first field bold, others muted text
4. Flash animation for last saved entry:
   @keyframes flashEntry { 0%,100%{background:transparent} 50%{background: indigo-100 / dark:indigo-900} }
   Apply for 300ms then remove class
5. ⋮ context menu: custom floating dropdown (not browser default), closes on outside click
6. Search + filter bar: sticky below divider, always visible when scrolling list
7. Empty list state: "No entries yet. Add your first one above." centered, muted
8. Import JSON section: collapsible, code-style textarea (monospace, dark bg), inline success/error feedback
9. Mobile: form stacks full width with large inputs, list shows 2 columns only, export button in floating bottom bar

Dark mode throughout. Preserve all logic.
```

---

**[YOU]** After Prompt 4.2:
- Test on mobile viewport in browser devtools
- `git commit -m "feat: content editor polish and mobile"`

---

### PHASE 5 — Project View

---

#### Prompt 5.1 🤖 GEMINI — BUILD
*Tag: Project View Page*

```
Build the Project View page for Bracer.

File: apps/web/src/pages/ProjectView.tsx

Layout:
- Top bar: back arrow + "Bracer" | Project name (click to rename inline) | version badge "v4" | sync status badge
- Section label: "SCHEMAS"
- Schema cards grid: 2 col desktop, 1 col mobile
  Each card: schema name, entry count, [Open] → /project/:id/content/:schemaId, [Edit Schema] → /project/:id/schema/:schemaId
- Below grid: [+ New Schema] and [Copy from ▾] dropdown
  Copy from dropdown: grouped — "This project" schemas + "Other projects" schemas
  On select: inline name input "New schema name: [input] [Create Copy]"
- Bottom bar: "Export entire project" + [↓ Download JSON]

[+ New Schema]:
- Inline in grid: new card outline with name input + [Create] button
- On create: navigate to SchemaEditor for the new schema

[↓ Download JSON]:
- Calls buildJSON for every schema
- Merges into: { "project": "name", "schemas": { "truth-cards": {...}, "dare-cards": {...} } }
- Downloads as projectname.json

Wire to projectStore. Use Project and Schema types. Dark mode. Clean layout.
```

---

**[YOU]** After Prompt 5.1:
- Test full navigation: Dashboard → Project → Schema Editor → back → Content Editor → back
- Test copy schema within project and across projects
- `git commit -m "feat: project view page"`

---

### PHASE 6 — Google Drive Sync

---

#### Prompt 6.1 🤖 CLAUDE — CRITICAL
*Tag: Google Drive Service*

```
Be concise. No lengthy explanations. Output code only.

Implement Google Drive sync for Bracer.

Files:
- packages/core/src/sync/driveService.ts
- apps/web/src/hooks/useGoogleAuth.ts
- apps/web/src/hooks/useDriveSync.ts

driveService.ts:
Use Google Drive API v3 REST (fetch only, no SDK).

Functions:
  initGoogleAuth(clientId: string): Promise<void>
  signIn(): Promise<{ name: string, email: string, avatar: string }>
  signOut(): void
  getAccessToken(): string | null

  ensureFolder(name: string, parentId?: string): Promise<string>
  uploadFile(name: string, content: string, folderId: string, existingFileId?: string): Promise<string>
  downloadFile(fileId: string): Promise<string>
  findFile(name: string, folderId: string): Promise<string | null>
  listFiles(folderId: string): Promise<{id: string, name: string, modifiedTime: string}[]>
  deleteFile(fileId: string): Promise<void>

useDriveSync.ts:
syncProject(project: Project): Promise<void>
  1. Ensure Bracer/ root folder in Drive
  2. Ensure Bracer/ProjectName/ folder
  3. Upload project.meta.json
  4. Ensure schemas/ and content/ subfolders
  5. Upload each schema as schemas/schemaname.schema.json
  6. Upload each content list as content/schemaname.json
  7. Update syncStatus to 'synced' in store

loadProjectFromDrive(projectFolderId: string): Promise<Project>
checkConflict(project: Project): Promise<boolean>
  Compare local version with Drive meta version number
resolveConflict(project: Project, useLocal: boolean): Promise<void>

useGoogleAuth.ts:
Wraps driveService auth. Returns { isAuthed, user, signIn, signOut }.
Persists token to localStorage. Handles token expiry.

Update project.meta.json to include syncBackend: 'drive'.
Update projectStore to call bumpVersion before every sync.
Load VITE_GOOGLE_CLIENT_ID from env.
```

---

**[YOU]** After Prompt 6.1:
- Sign in, check Drive for Bracer/ folder
- Save a project, confirm files appear in Drive
- `git commit -m "feat: google drive sync"`

---

### PHASE 7 — GitHub Sync

---

#### Prompt 7.1 🤖 CLAUDE — CRITICAL
*Tag: GitHub OAuth Serverless Function*

```
Be concise. Output code only.

Bracer needs GitHub OAuth. The client secret cannot be in the frontend.
Create a Vercel serverless function to handle the token exchange.

File: apps/web/api/github-oauth.ts (Vercel serverless function)

This function:
- Accepts GET request with ?code=xxx from GitHub OAuth callback
- Exchanges code for access token using:
  POST https://github.com/login/oauth/access_token
  with client_id, client_secret, code
- Returns { access_token } as JSON
- Uses env vars: GITHUB_CLIENT_ID, GITHUB_CLIENT_SECRET

Add to vercel.json (create if needed):
{
  "functions": {
    "api/**": { "runtime": "@vercel/node" }
  }
}

Also create apps/web/src/pages/GitHubCallback.tsx:
- Route: /auth/github/callback
- On mount: reads ?code from URL, calls /api/github-oauth, stores access_token in localStorage
- Shows "Connecting to GitHub..." then redirects to /
- On error: shows error message with retry

Add route in App.tsx: /auth/github/callback → GitHubCallback
```

---

**[YOU]** After Prompt 7.1:
- Add `GITHUB_CLIENT_ID` and `GITHUB_CLIENT_SECRET` to Vercel environment variables
- Deploy to Vercel, test GitHub OAuth flow end to end
- `git commit -m "feat: github oauth serverless function"`

---

#### Prompt 7.2 🤖 CLAUDE — CRITICAL
*Tag: GitHub Contents API Sync Service*

```
Be concise. Output code only.

Implement GitHub sync for Bracer using the GitHub Contents API (REST).

Files:
- packages/core/src/sync/githubService.ts
- apps/web/src/hooks/useGitHubAuth.ts
- apps/web/src/hooks/useGitHubSync.ts

githubService.ts:
All requests use Authorization: Bearer {token} header.
Base URL: https://api.github.com

Functions:
  getUser(token: string): Promise<{ login: string, name: string, avatar_url: string }>

  listRepos(token: string): Promise<{ name: string, full_name: string, default_branch: string }[]>

  getFile(token: string, repo: string, path: string): Promise<{ content: string, sha: string } | null>
  // content is base64 encoded — decode it

  createOrUpdateFile(
    token: string,
    repo: string,
    path: string,
    content: string,      // plain string, function encodes to base64
    message: string,      // commit message
    sha?: string          // required for updates, omit for creates
  ): Promise<{ sha: string }>

  deleteFile(token: string, repo: string, path: string, sha: string, message: string): Promise<void>

  listFolder(token: string, repo: string, path: string): Promise<{ name: string, path: string, sha: string, type: 'file'|'dir' }[]>

useGitHubSync.ts:
State per project: { repo: string, basePath: string, token: string }
Store this in project settings (add githubConfig to Project type).

syncProject(project: Project): Promise<void>
  File paths follow this structure inside the user's chosen repo + basePath:
    {basePath}/.bracer/project.meta.json
    {basePath}/schemas/{schemaname}.schema.json
    {basePath}/{schemaname}.json   ← the actual content, clean export
  
  For each file:
    1. Try getFile to get existing sha
    2. Call createOrUpdateFile with sha if exists, without if new
    3. Commit message format: "bracer: sync {project.name} v{version}"

  Update project syncStatus in store.

loadProjectFromGitHub(repo: string, basePath: string, token: string): Promise<Project>
  Read .bracer/project.meta.json
  Read all schemas and content files
  Reconstruct Project object

checkConflict(project: Project): Promise<boolean>
  Read remote project.meta.json version field
  Compare to local project.version

resolveConflict(project: Project, useLocal: boolean): Promise<void>

useGitHubAuth.ts:
Returns { isAuthed, user, token, signIn, signOut }
signIn: redirects to GitHub OAuth URL with scopes: repo (to read/write repos)
Stores token from localStorage (set by GitHubCallback page)

Add githubConfig?: { repo: string, basePath: string } to Project type in types.ts.
```

---

**[YOU]** After Prompt 7.2:
- Connect GitHub, pick a repo
- Save a project — check the repo on GitHub for committed files
- Verify the exported JSON in GitHub matches your original naughty pack format exactly
- `git commit -m "feat: github sync service"`

---

#### Prompt 7.3 🤖 GEMINI — BUILD
*Tag: Sync Settings UI + GitHub Repo Picker*

```
Build sync settings UI for Bracer.

Files:
- apps/web/src/components/SyncStatusBadge.tsx
- apps/web/src/components/ConflictBanner.tsx
- apps/web/src/components/DriveAuthStrip.tsx
- apps/web/src/components/GitHubSyncSetup.tsx

SyncStatusBadge:
Props: status, lastSynced?, backend: 'drive'|'github'|'local'
Shows Drive icon or GitHub icon next to status text.
Colors: green=synced, blue=syncing, yellow=conflict, gray=local, red=error.
Always includes icon not just color (accessibility).

ConflictBanner:
Inline banner (not modal). Shows when conflict detected on project load.
Props: localVersion, remoteVersion, localDate, remoteDate, backend, onKeepLocal, onUseRemote, onCompare
[Keep mine] [Use remote] [Compare side by side]
Compare: side-by-side JSON diff view in a panel below the banner.

DriveAuthStrip:
Compact strip in Settings and Dashboard.
Not authed: [Connect Google Drive] button.
Authed: avatar + email + "Drive connected" + [Disconnect].

GitHubSyncSetup:
Shown when user connects GitHub to a project for the first time.
Step 1: "Choose a repository" — searchable dropdown list of their repos
Step 2: "Choose a folder path" — text input (default: bracer/) with explanation:
  "Bracer will save your project files here. Use an existing content folder in your repo if you have one."
Step 3: [Connect] — runs first sync

Also: in ProjectView top bar, show sync backend icon (Drive or GitHub) next to sync status badge.
In project card on Dashboard, show small Drive/GitHub icon bottom right.

Integrate ConflictBanner into ProjectView (detect conflict on page load, show banner if true).
```

---

**[YOU]** After Prompt 7.3:
- Test GitHub repo picker with a real repo
- Test conflict by manually editing version in GitHub
- `git commit -m "feat: sync UI components drive and github"`

---

### PHASE 8 — PWA + Responsive Polish

---

#### Prompt 8.1 🤖 GEMINI — BUILD
*Tag: PWA + Mobile Navigation + Settings*

```
Configure Bracer as a PWA and add mobile navigation and settings page.

1. PWA (vite-plugin-pwa):
   Install: pnpm add -D vite-plugin-pwa
   Configure vite.config.ts:
   name: "Bracer", short_name: "Bracer"
   theme_color: "#6366f1"
   Icons: simple SVG "B" monogram on indigo background, 192x192 and 512x512
   Cache: StaleWhileRevalidate for assets

2. MobileNav (apps/web/src/components/MobileNav.tsx):
   Show only on screens < 768px
   Three tabs: Projects (home icon) / Recents (clock) / Settings (gear)
   Fixed bottom, safe-area-inset-bottom aware
   Active tab: accent color

3. Settings page (apps/web/src/pages/Settings.tsx):
   Sections:
   - Appearance: Light / Dark / System
   - Sync: Drive status + connect/disconnect, GitHub status + connect/disconnect
   - Data: Export all projects as ZIP, Clear all local data (confirm dialog)
   - About: version from package.json, GitHub link, Buy Me a Coffee button (yellow, ☕)
     Text: "Bracer is free and open source. If it saves you time, a coffee keeps it going. ☕"
   - Credits: "Built by [your name]"

4. PWA install prompt:
   Listen for beforeinstallprompt
   After 2 sessions: subtle bottom banner "Install Bracer for offline access" [Install] [Not now]
   Dismiss permanently on "Not now"
```

---

**[YOU]** After Prompt 8.1:
- Test on real mobile device
- `git commit -m "feat: PWA mobile nav and settings"`

---

#### Prompt 8.2 🤖 CLAUDE — CRITICAL
*Tag: Dark Mode Audit*

```
Be concise. Output only changed files.

Audit all Bracer components and pages for dark mode consistency.

Check every file. Fix:
1. All backgrounds use CSS variables or Tailwind dark: — no hardcoded colors
2. Text contrast passes WCAG AA in both modes
3. Borders, shadows, dividers adapt correctly
4. JSON/code textareas always dark bg + light text regardless of theme
5. Sync status badges use icons not just color
6. All focus rings visible in both modes
7. Hover states visible in dark mode (not just color change — use opacity or border)

Output: list of files changed + what was fixed, then corrected code.
```

---

**[YOU]**
- Toggle theme, review every screen
- `git commit -m "fix: dark mode audit"`

---

### PHASE 9 — Desktop App (Tauri)

---

**[YOU]** Install Rust before starting this phase:
```bash
curl --proto '=https' --tlsv1.2 -sSf https://sh.rustup.rs | sh
source ~/.cargo/env
```

---

#### Prompt 9.1 🤖 CLAUDE — CRITICAL
*Tag: Tauri Setup + File System*

```
Be concise. Output code only.

Set up Tauri v2 for Bracer desktop app.

Working directory: apps/desktop/

1. Initialize Tauri v2 pointing to apps/web/dist as frontend:
   pnpm add -D @tauri-apps/cli
   Configure src-tauri/tauri.conf.json:
   - productName: "Bracer"
   - version: "0.1.0"
   - identifier: "com.bracer.app"
   - window: title "Bracer", width 1200, height 800, minWidth 800, minHeight 600
   - build: frontendDist pointing to ../../web/dist

2. Tauri commands in src-tauri/src/main.rs:
   save_file(path: String, content: String) → Result<(), String>
   read_file(path: String) → Result<String, String>
   pick_save_path(default_name: String) → Result<String, String>  // save dialog
   pick_open_path() → Result<String, String>  // open file dialog

3. apps/web/src/services/fileService.ts:
   Detects Tauri: typeof window.__TAURI__ !== 'undefined'

   exportJSON(filename: string, content: string): Promise<void>
     Tauri: pick_save_path(filename) → save_file(path, content)
     Browser: blob URL download

   importJSON(): Promise<string | null>
     Tauri: pick_open_path() → read_file(path)
     Browser: hidden file input + FileReader

4. Replace all download/upload file logic in ContentEditor and ProjectView with fileService calls.

Add to apps/desktop/package.json build scripts:
  "tauri:dev": "tauri dev"
  "tauri:build": "tauri build"
```

---

**[YOU]** After Prompt 9.1:
```bash
cd apps/web && pnpm build
cd ../desktop && pnpm tauri dev
```
- Test export JSON from desktop — confirm save dialog appears
- `git commit -m "feat: tauri desktop app"`

---

#### Prompt 9.2 🤖 GEMINI — BUILD
*Tag: Desktop Polish*

```
Add desktop polish to Bracer Tauri app.

1. Custom titlebar (apps/web/src/components/TitleBar.tsx):
   Only render when window.__TAURI__ exists.
   - Draggable region: "Bracer" centered, current project name left
   - Window controls right: minimize, maximize, close (call @tauri-apps/api/window)
   - Height 40px, bg: --bg-secondary
   Set decorations: false in tauri.conf.json.

2. Keyboard shortcuts (apps/web/src/hooks/useKeyboardShortcuts.ts):
   Cmd/Ctrl+S → save current form/schema
   Cmd/Ctrl+N → new entry (focus first field)
   Cmd/Ctrl+E → export JSON
   Cmd/Ctrl+, → open settings
   Escape → cancel edit mode
   Show hints in button tooltips.

3. Tauri app menu (src-tauri/src/main.rs):
   File: New Project, Open Project, Export JSON, separator, Quit
   View: Toggle Dark Mode
   Help: About Bracer, GitHub, Buy Me a Coffee
```

---

**[YOU]**
- Test all keyboard shortcuts
- `git commit -m "feat: desktop titlebar and keyboard shortcuts"`

---

### PHASE 10 — Final Polish

---

#### Prompt 10.1 🤖 CLAUDE — CRITICAL
*Tag: JSON Export Accuracy + Unit Tests*

```
Be concise. Output fixed code and tests only.

Audit and harden buildJSON in packages/core/src/jsonBuilder.ts.

Requirements:
1. Root fields output in exact order defined in schema.rootFields
2. List fields output as arrays in exact order of schema.listFields
3. null values export as JSON null (not undefined, not empty string)
4. number-nullable: empty input → null
5. toggle: exports as boolean (not string)
6. tags: exports as string[] (empty → [])
7. object-optional: disabled → null, enabled → nested object with all sub-fields
8. Key names come from schema field name property (the JSON key), not label

Write vitest unit tests covering all field types and edge cases.
Include a test using the exact naughty pack JSON structure (pack_id, name, description, ads_required, cards array with follow_up object-optional).

Also harden inferSchemaFromJSON:
- Correctly handles nullable fields (timer_seconds: null)
- Correctly identifies follow_up as object-optional
- topic_tags correctly identified as tags type
```

---

**[YOU]**
- `pnpm test` — all pass
- Export naughty pack JSON, diff with original
- `git commit -m "fix: json export accuracy and tests"`

---

#### Prompt 10.2 🤖 GEMINI — BUILD
*Tag: Empty States + Toasts + Branding*

```
Final polish for Bracer.

1. EmptyState component (apps/web/src/components/EmptyState.tsx):
   Props: icon, title, description, action?: {label, onClick}
   Apply to:
   - Dashboard: no projects → "Create your first project"
   - ProjectView: no schemas → "Define your first schema"
   - ContentEditor: no entries → "Add your first entry above"
   - ContentEditor search: no results → "No entries match your search"

2. Toast system (apps/web/src/components/Toast.tsx):
   No library. Build lightweight.
   Position: bottom-right desktop, bottom-center mobile
   Types: success (green), error (red), info (blue)
   Auto-dismiss 3 seconds, manual close
   Use for: entry saved, schema saved, export complete, sync complete, sync error

3. Loading states:
   - Skeleton loaders for project cards (3 placeholder cards)
   - Spinner overlay during Drive/GitHub sync
   - "Saving..." text replaces [Save Entry] during save

4. Branding:
   SVG logo: simple "B" mark or bracket icon, indigo, works light+dark
   Save as apps/web/public/logo.svg
   Use in dashboard top bar and desktop titlebar

   Page titles:
   Dashboard: "Bracer"
   Project: "ProjectName — Bracer"
   Schema: "Edit Schema — Bracer"
   Content: "SchemaName — Bracer"

   Web footer only:
   "Bracer · Open Source · ☕ Buy me a coffee · GitHub"
   Muted, small, centered

5. Settings About section:
   Version from package.json
   Buy Me a Coffee: yellow button, link to https://buymeacoffee.com/YOUR_LINK
   Text: "Bracer is free and open source. If it saves you time, a coffee keeps it going. ☕"

Install lucide-react for icons if not already installed.
```

---

**[YOU]**
- Update Buy Me a Coffee link with your real URL
- `git commit -m "feat: empty states, toasts, branding"`

---

### PHASE 11 — CI/CD & Release

---

#### Prompt 11.1 🤖 CLAUDE — CRITICAL
*Tag: GitHub Actions Pipeline*

```
Be concise. Output workflow files only.

Set up GitHub Actions CI/CD for Bracer.

.github/workflows/ci.yml:
Triggers: push and PR to main
Jobs:
  typecheck: tsc --noEmit
  test: pnpm test (vitest in packages/core)
  build: pnpm build (apps/web)
Fail PR if any job fails.

.github/workflows/deploy.yml:
Triggers: push to main only
Jobs:
  deploy-web:
    Build apps/web
    Deploy to Vercel using secrets: VERCEL_TOKEN, VERCEL_ORG_ID, VERCEL_PROJECT_ID

.github/workflows/release.yml:
Triggers: push of version tags (v*)
Jobs:
  build-desktop:
    Matrix: ubuntu-latest, macos-latest, windows-latest
    Each: checkout, install Rust, pnpm install, pnpm build (web), pnpm tauri build
    Upload build artifacts
  create-release:
    Needs: build-desktop
    Creates GitHub Release with tag name
    Attaches all platform installers as release assets

Root package.json scripts:
  "build": "pnpm --filter ./apps/web build"
  "test": "pnpm --filter ./packages/core test"
  "typecheck": "tsc -p apps/web/tsconfig.json --noEmit"
```

---

**[YOU]** After Prompt 11.1:
- Add secrets to GitHub repo: `VERCEL_TOKEN`, `VERCEL_ORG_ID`, `VERCEL_PROJECT_ID`, `GITHUB_CLIENT_SECRET`
- Push to main — confirm web deploys to Vercel
- `git commit -m "ci: github actions pipeline"`

**[YOU] — To release desktop app:**
```bash
git tag v0.1.0
git push origin v0.1.0
```
GitHub Actions builds Mac, Windows, Linux installers and attaches to a GitHub Release automatically.

---

## 6. Release Checklist

### Pre-Release

- [ ] Web app live on Vercel
- [ ] Custom domain configured (optional)
- [ ] Desktop builds tested on Mac and Windows
- [ ] Google OAuth consent screen approved for production domain
- [ ] GitHub OAuth app updated with production callback URL
- [ ] Vercel env vars set: `GITHUB_CLIENT_ID`, `GITHUB_CLIENT_SECRET`, `VITE_GOOGLE_CLIENT_ID`, `VITE_GITHUB_CLIENT_ID`
- [ ] All unit tests passing
- [ ] Dark mode tested on every screen
- [ ] Mobile tested on real device
- [ ] JSON export verified byte-for-byte against original
- [ ] Drive sync tested across two browsers
- [ ] GitHub sync tested — commits appear in target repo
- [ ] Conflict resolution tested for both Drive and GitHub
- [ ] Buy Me a Coffee link updated
- [ ] README complete
- [ ] MIT license in repo

### Launch

- [ ] Reddit: r/SideProject, r/webdev, r/indiegaming, r/gamedev
- [ ] Product Hunt
- [ ] Hacker News (Show HN)
- [ ] X/Twitter with demo GIF
- [ ] Add to relevant awesome-lists on GitHub

---

## 7. README Templates

### Root README.md

```markdown
# Bracer

> Schema-first JSON content editor for indie developers and content creators.

Define your data structure once. Fill it in with a clean form. Export perfect JSON.

**No backend. No subscription. No lock-in.**

[Try it free →](https://bracer.vercel.app) · [Download desktop app](https://github.com/YOUR_USERNAME/bracer/releases) · [Buy me a coffee ☕](https://buymeacoffee.com/YOUR_LINK)

---

## What is Bracer?

If your app or game runs on JSON data packs — card games, quiz apps, flashcard sets, dialogue trees, item databases — Bracer gives you a clean UI to create and manage that content without touching code.

## Features

- **Schema-first** — paste an example JSON or define fields manually
- **Fast content entry** — conveyor belt form: save, instantly ready for next entry
- **Google Drive sync** — for non-technical users and cross-device access
- **GitHub sync** — for developers: commits your JSON directly into your repo with full version history
- **Works everywhere** — web app, desktop (Mac/Windows/Linux), mobile PWA
- **No account required** — local mode works offline, no sign-in needed
- **Open source** — MIT, host it yourself

## Getting Started

### Web App
[bracer.vercel.app](https://bracer.vercel.app) — no install.

### Desktop
[Download latest release](https://github.com/YOUR_USERNAME/bracer/releases)

### Self-host
```bash
git clone https://github.com/YOUR_USERNAME/bracer.git
cd bracer/apps/web
pnpm install && pnpm dev
```

## Development

See [BRACER_MASTER_PLAN.md](./BRACER_MASTER_PLAN.md) for full build plan.

```bash
pnpm install
pnpm dev
pnpm test
pnpm build
```

## License
MIT © [Your Name]
```

---

### apps/web/README.md

```markdown
# Bracer — Web App

React 18 + TypeScript + Vite + Tailwind CSS + Zustand

## Dev
```bash
pnpm install && pnpm dev
```

## Env
```
VITE_GOOGLE_CLIENT_ID=
VITE_GITHUB_CLIENT_ID=
VITE_GITHUB_REDIRECT_URI=http://localhost:5173/auth/github/callback
```

## Build
```bash
pnpm build   # output: dist/
```
```

---

### apps/desktop/README.md

```markdown
# Bracer — Desktop (Tauri v2)

## Requirements
- Rust (via rustup)
- Node.js 18+ and pnpm

## Dev
```bash
cd ../web && pnpm build
cd ../desktop && pnpm tauri dev
```

## Build
```bash
pnpm tauri build
# Output: src-tauri/target/release/bundle/
```
```

---

*Bracer Master Plan — v2.0*
*Two sync backends. One clean tool. Ship it.*
