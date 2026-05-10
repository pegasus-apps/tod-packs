# Bracr — Master Build Plan
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
Bracr is a schema-first JSON content editor. Users define the structure of their data once, then fill it in with a clean form UI — no code required. Think of it as a lightweight CMS you own entirely, with no backend, no database, and no subscription.

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
- No user accounts or cloud database
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
│  Bracr                        [+ New Project]│
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
│  [Sign in with Google — enable Drive sync]      │
│  Continue without account →                     │
└─────────────────────────────────────────────────┘
```

**Rules:**
- Google sign-in is optional and equal in weight to guest mode — no guilt-tripping
- Sync status shown on every card: `● Synced`, `↑ Syncing`, `⚠ Conflict`, `○ Local only`, `✕ Sync error`
- Guest users see subtle persistent reminder: *"Export your JSON regularly to avoid data loss"*
- Clicking a project card navigates into it immediately
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
│  │ pack_id     [Short Text ▾]  [Required ✓] │   │
│  │ name        [Short Text ▾]  [Required ✓] │   │
│  │ description [Long Text  ▾]  [Required  ] │   │
│  │ ads_required[Number     ▾]  [Required  ] │   │
│  │                              [× delete]  │   │
│  └──────────────────────────────────────────┘   │
│  [+ Add Root Field]                             │
│                                                 │
│  LIST FIELDS  (arrays you'll add items to)      │
│  ┌──────────────────────────────────────────┐   │
│  │ cards  →  [Define card shape ▾]          │   │
│  │                                          │   │
│  │   id           [Short Text  ▾] [Req ✓]  │   │
│  │   type         [Dropdown    ▾] [Req ✓]  │   │
│  │               Options: truth, dare       │   │
│  │               [+ add option]             │   │
│  │   content      [Long Text   ▾] [Req ✓]  │   │
│  │   level        [Number      ▾] [Req ✓]  │   │
│  │   topic_tags   [Tags        ▾]           │   │
│  │   repeatable   [Toggle      ▾]           │   │
│  │   gender       [Dropdown    ▾]           │   │
│  │               Options: any, male, female │   │
│  │   timer_seconds[Number Null ▾]           │   │
│  │   follow_up    [Object Opt. ▾]           │   │
│  │     └── id          [Short Text ▾]      │   │
│  │     └── content     [Long Text  ▾]      │   │
│  │     └── probability [Number     ▾]      │   │
│  │     └── timer_text  [Short Text ▾]      │   │
│  └──────────────────────────────────────────┘   │
│  [+ Add List Field]                             │
│                                                 │
│                            [Save Schema →]      │
└─────────────────────────────────────────────────┘
```

**Path B — Import from example JSON (collapsed by default):**
```
  [+ Import from example JSON]
  ▼ expanded:
  ┌──────────────────────────────────┐
  │ Paste one example entry here... │
  │                                  │
  │ { "type": "truth", "tier":...   │
  └──────────────────────────────────┘
  [Detect Fields →]
```
After detection, fields appear pre-filled in the schema editor. User confirms type for each, adds enum options to dropdowns, marks required fields. The example JSON is a **hint only** — user has full control.

**Field detection logic (simple, reliable):**
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
- Dropdown fields show inline option editor — type and hit Enter to add options
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
│                                                 │
│  id            [nau_t_033                    ]  │
│  type          [truth              ▾]           │
│  content       [                             ]  │
│                [                             ]  │
│                [                             ]  │
│  level         [2    ]                          │
│  topic_tags    [attraction ×] [flirty ×] [+]   │
│  repeatable    [✓]                              │
│  gender        [any                ▾]           │
│  timer_seconds [null ]                          │
│  follow_up     [ ] Add follow-up               │
│                ▼ expanded when toggled:         │
│                content  [                    ]  │
│                prob.    [0.7 ]                  │
│                                                 │
│  [Save Entry]  ← clears form, stays on screen  │
│                                                 │
│  ════ 32 ENTRIES ════════════ [↓ Export JSON]  │
│  Search...                   [Filter: All ▾]   │
│                                                 │
│  nau_t_001  truth  lv1  What's the most...  ⋮  │
│  nau_t_002  truth  lv2  Have you ever had.. ⋮  │
│  nau_t_003  truth  lv2  What's the most fl. ⋮  │
│                                                 │
│  [See all entries →]    [Sync with Drive ↑]    │
└─────────────────────────────────────────────────┘
```

**Critical UX rules:**
- Form is **always visible at the top** — never hidden behind a sheet or modal
- [Save Entry] clears the form instantly and stays on screen — conveyor belt pattern
- The last saved entry **briefly highlights** at the top of the list (300ms flash) as confirmation
- Clicking an entry in the list opens it **inline for editing** — replaces the add form temporarily, with a [Cancel] to return to add mode
- Duplicate entry from the ⋮ menu — prefills the form with that entry's data
- [See all entries] expands the list to full screen for bulk review
- Context dropdown at top switches between **Pack metadata** (simple field editor) and each **list field** (the add form)
- Export JSON always available — exports the complete reconstructed JSON

**Importing existing JSON:**
- Option on content editor: [Import existing JSON]
- Tool parses file → separates metadata from array fields
- Shows preview: "Found pack metadata (5 fields). Found 1 list: cards (20 items)"
- User confirms → existing entries load into the list
- User continues adding from entry 21 onwards
- Export concatenates everything correctly

---

### Sync & Version Model

**project.meta.json (stored in Drive folder):**
```json
{
  "name": "Couple Game",
  "created": "2026-05-10",
  "lastModified": "2026-05-10T14:32:00Z",
  "lastModifiedBy": "device-web-chrome",
  "version": 4,
  "schemas": ["truth-cards", "dare-cards"]
}
```

**Conflict resolution (never auto-merge):**
```
⚠️  Sync conflict on "Couple Game"

  This device:    Version 4  (edited 2 mins ago)
  Google Drive:   Version 6  (edited on another device)

  [Keep mine]   [Use Drive version]   [Compare side by side]
```

**Drive folder structure:**
```
Bracr/
├── Couple Game/
│   ├── project.meta.json
│   ├── schemas/
│   │   ├── truth-cards.schema.json
│   │   └── dare-cards.schema.json
│   └── content/
│       ├── truth-cards.json
│       └── dare-cards.json
└── Chat Tool/
    ├── project.meta.json
    └── ...
```

**Sync status indicators (always honest):**
- `● Synced` — "Synced 2 mins ago"
- `↑ Syncing` — "Uploading..."
- `⚠ Conflict` — "Edited on 2 devices — tap to resolve"
- `○ Local only` — "Not connected to Drive"
- `✕ Error` — "Sync failed — tap to retry"

---

### Mobile Adaptations
- Bottom tab bar: Projects | Recents | Settings
- Content editor: form on top, list scrolls below — same layout, larger tap targets
- Schema editor: one field at a time, accordion style
- Drive sync status in top bar always visible
- Add to home screen as PWA (no App Store needed for v1)

---

## 3. Tech Stack

| Layer | Choice | Reason |
|---|---|---|
| Framework | React 18 + Vite | Fast, great ecosystem, easy to wrap |
| Styling | Tailwind CSS | Utility-first, consistent, fast to build |
| State | Zustand | Lightweight, no boilerplate |
| Local storage | localStorage + IndexedDB | localStorage for settings, IndexedDB for content |
| Desktop | Tauri v2 | Same React code, 5MB app vs 150MB Electron |
| Drive API | Google Drive API v3 + Picker API | Free, well documented |
| Auth | Google OAuth 2.0 | Only for Drive — guest mode needs nothing |
| Deploy (web) | Vercel | Free forever at this scale |
| Package manager | pnpm | Faster than npm |
| Language | TypeScript | Catch schema shape errors early |

---

## 4. Repo & Project Setup Instructions

### Step 1 — Create GitHub Repository

1. Go to github.com → New Repository
2. Name it: `bracr`
3. Description: `Schema-first JSON content editor for indie developers and content creators`
4. Set to **Public**
5. Add README: **Yes**
6. Add .gitignore: **Node**
7. License: **MIT**
8. Click Create Repository

### Step 2 — Clone Locally

```bash
git clone https://github.com/YOUR_USERNAME/bracr.git
cd bracr
```

### Step 3 — Create Folder Structure

```bash
mkdir -p apps/web apps/desktop packages/core docs
```

Your repo structure will be:
```
bracr/
├── apps/
│   ├── web/          ← React + Vite web app (also PWA)
│   └── desktop/      ← Tauri wrapper
├── packages/
│   └── core/         ← Shared logic (schema parser, JSON builder, Drive API)
├── docs/             ← Documentation
├── .github/
│   └── workflows/    ← CI/CD
├── PACKFORGE_MASTER_PLAN.md
└── README.md
```

### Step 4 — Initialize the Web App

```bash
cd apps/web
pnpm create vite . --template react-ts
pnpm install
pnpm add -D tailwindcss postcss autoprefixer
pnpm add zustand
npx tailwindcss init -p
```

### Step 5 — Set Up Google Cloud Project (for Drive)

1. Go to console.cloud.google.com
2. New Project → Name: `Bracr`
3. Enable APIs:
   - Google Drive API
   - Google Picker API
4. Create OAuth 2.0 credentials:
   - Application type: Web application
   - Authorized origins: `http://localhost:5173`, `https://bracr.vercel.app`
5. Save your `CLIENT_ID` — you'll need it later
6. Create `.env` file in `apps/web/`:
```
VITE_GOOGLE_CLIENT_ID=your_client_id_here
```

### Step 6 — Initial Commit

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
> - Instructions marked `[YOU]` are things you do manually
> - Prompts marked `🤖 CLAUDE — CRITICAL` go to Claude (hard logic, architecture, core systems)
> - Prompts marked `🤖 GEMINI — BUILD` go to Gemini (building UI around established patterns)
> - After each prompt: test it, then `git commit` before moving to the next
> - Claude prompts include a note to be concise to save tokens

---

### PHASE 1 — Foundation & Design System

---

**[YOU]** Make sure you are inside `apps/web/` before starting Phase 1.

---

#### Prompt 1.1 🤖 CLAUDE — CRITICAL
*Tag: Architecture + Design System*

```
You are building Bracr — a schema-first JSON content editor built with React 18, TypeScript, Vite, Tailwind CSS, and Zustand.

Be concise. No lengthy explanations. Output code only with brief inline comments where needed.

Set up the following:

1. Tailwind config with this design system:
   - Colors: slate scale for neutrals, indigo-500 as accent
   - Dark mode: class-based
   - Custom font: Inter (UI), JetBrains Mono (code/JSON)
   - Custom border radius: sm=4px, md=8px, lg=12px

2. Global CSS with CSS variables for both light and dark theme:
   --bg-primary, --bg-secondary, --bg-tertiary
   --text-primary, --text-secondary, --text-muted
   --border, --border-subtle
   --accent, --accent-hover
   --success, --warning, --error

3. Base component files (unstyled logic only, styled with Tailwind):
   - components/ui/Button.tsx (variants: primary, secondary, ghost, danger)
   - components/ui/Input.tsx (short text input)
   - components/ui/Textarea.tsx
   - components/ui/Toggle.tsx (boolean switch)
   - components/ui/Dropdown.tsx (select with custom options)
   - components/ui/TagInput.tsx (add/remove string tags)
   - components/ui/Badge.tsx (status indicators)
   - components/ui/Card.tsx (container with hover state)

4. ThemeProvider.tsx that reads system preference and allows manual toggle. Persists to localStorage.

5. App.tsx with a basic router (react-router-dom v6):
   - / → ProjectDashboard (placeholder)
   - /project/:id → ProjectView (placeholder)
   - /project/:id/schema/:schemaId → SchemaEditor (placeholder)
   - /project/:id/content/:schemaId → ContentEditor (placeholder)

6. Zustand store: store/appStore.ts
   - currentTheme: 'light' | 'dark'
   - isGoogleAuthed: boolean
   - googleUser: { name, email, avatar } | null

Install react-router-dom if not present.
```

---

**[YOU]** After Claude responds:
- Copy code into your project
- Run `pnpm dev` — confirm app loads with no errors
- Confirm dark/light toggle works
- `git commit -m "feat: design system and base components"`

---

#### Prompt 1.2 🤖 CLAUDE — CRITICAL
*Tag: Core TypeScript Types*

```
Be concise. Output types only.

Create packages/core/src/types.ts with all TypeScript types for Bracr.

Types needed:

FieldType = 'short-text' | 'long-text' | 'number' | 'number-nullable' | 'toggle' | 'dropdown' | 'tags' | 'object-optional' | 'list'

SchemaField {
  id: string
  name: string        // the JSON key name
  label: string       // display label
  type: FieldType
  required: boolean
  options?: string[]  // for dropdown type
  subFields?: SchemaField[]  // for object-optional and list types
}

Schema {
  id: string
  name: string
  rootFields: SchemaField[]   // metadata fields (set once)
  listFields: SchemaField[]   // array fields (add items to)
}

Entry = Record<string, any>   // a single content item

ContentList {
  schemaId: string
  listFieldName: string   // e.g. "cards"
  entries: Entry[]
}

Project {
  id: string
  name: string
  created: string       // ISO date
  lastModified: string  // ISO datetime
  lastModifiedBy: string
  version: number
  schemas: Schema[]
  contentLists: ContentList[]
  syncStatus: 'synced' | 'syncing' | 'conflict' | 'local' | 'error'
  driveFolder?: string
}

AppState {
  projects: Project[]
  activeProjectId: string | null
  activeSchemaId: string | null
}

Also create:
- packages/core/src/schemaInference.ts
  Function: inferSchemaFromJSON(json: Record<string, any>, sampleEntries?: Record<string, any>[]): Schema
  Logic:
  - Top-level non-array fields → rootFields
  - Top-level array-of-objects fields → listFields (with sub-fields inferred)
  - string with ≤5 unique values in samples → dropdown suggestion
  - string avg length >60 → long-text
  - null or object → object-optional
  - array of strings → tags
  - boolean → toggle
  - number → number (check if any sample has null → number-nullable)

- packages/core/src/jsonBuilder.ts
  Function: buildJSON(project: Project, schemaId: string): Record<string, any>
  Reconstructs the exact original JSON shape from project data.
  Root metadata fields go at top level.
  List fields become arrays of entries.
```

---

**[YOU]**
- Copy into `packages/core/src/`
- `git commit -m "feat: core types and schema inference logic"`

---

### PHASE 2 — Project Dashboard

---

#### Prompt 2.1 🤖 GEMINI — BUILD
*Tag: Project Dashboard UI*

```
Build the Bracr Project Dashboard page for a React + TypeScript + Tailwind app.

File: apps/web/src/pages/ProjectDashboard.tsx

Design: Clean, card-based. Neutral slate background. Indigo accent. Supports dark mode via Tailwind dark: classes.

Use these existing UI components: Card, Button, Badge from components/ui/

Use the Project type from packages/core/src/types.ts

Layout:
- Top bar: "Bracr" logo left, "[+ New Project]" button right
- Project grid: responsive 1-col mobile, 2-col tablet, 3-col desktop
- Each project card shows: name, schema count, total entry count, sync status badge, last modified time
- Sync status badge colors: green=synced, blue=syncing, yellow=conflict, gray=local, red=error
- Bottom of page: Google sign-in strip (if not authed) OR user avatar + "Connected to Drive" (if authed)
- Empty state: centered illustration placeholder + "Create your first project" button

Interactions:
- Click card → navigate to /project/:id
- Right-click card → context menu: Rename, Duplicate, Delete, Export
- [+ New Project] → opens inline modal with just a name input and [Create] button

Use mock data for 2-3 projects. No real store needed yet — use useState with mock data.

Keep it clean. No lorem ipsum. Use realistic project names like "Couple Card Game" and "Quiz App".
```

---

**[YOU]**
- Copy into project
- `pnpm dev` and review in browser
- Check mobile view (browser devtools)
- `git commit -m "feat: project dashboard UI"`

---

#### Prompt 2.2 🤖 CLAUDE — CRITICAL
*Tag: Zustand Store + LocalStorage Persistence*

```
Be concise. Code only.

Create the main Zustand store for Bracr.

File: apps/web/src/store/projectStore.ts

Use zustand with persist middleware (localStorage).

State:
- projects: Project[]
- activeProjectId: string | null

Actions:
- createProject(name: string): Project
- updateProject(id: string, updates: Partial<Project>): void
- deleteProject(id: string): void
- duplicateProject(id: string): Project
- addSchema(projectId: string, schema: Schema): void
- updateSchema(projectId: string, schemaId: string, updates: Partial<Schema>): void
- deleteSchema(projectId: string, schemaId: string): void
- copySchema(fromProjectId: string, fromSchemaId: string, toProjectId: string, newName: string): Schema
- addEntry(projectId: string, schemaId: string, listFieldName: string, entry: Entry): void
- updateEntry(projectId: string, schemaId: string, listFieldName: string, entryIndex: number, entry: Entry): void
- deleteEntry(projectId: string, schemaId: string, listFieldName: string, entryIndex: number): void
- duplicateEntry(projectId: string, schemaId: string, listFieldName: string, entryIndex: number): void
- setActiveProject(id: string | null): void
- bumpVersion(projectId: string): void  // increment version + update lastModified

All IDs use crypto.randomUUID().
createProject sets version=1, created=now, lastModified=now, syncStatus='local'.
bumpVersion is called automatically inside any action that mutates content.

Wire up ProjectDashboard.tsx to use this store instead of mock data.
```

---

**[YOU]**
- Apply changes
- Test: create project, see it persist after page refresh
- `git commit -m "feat: zustand store with localStorage persistence"`

---

### PHASE 3 — Schema Editor

---

#### Prompt 3.1 🤖 CLAUDE — CRITICAL
*Tag: Schema Editor Core Logic*

```
Be concise. Code only.

Build the Schema Editor page for Bracr.

File: apps/web/src/pages/SchemaEditor.tsx

This is the most critical screen. It defines the shape of all data.

The editor has two sections:
1. ROOT FIELDS — metadata fields (SchemaField[] mapped from schema.rootFields)
2. LIST FIELDS — array fields (SchemaField[] mapped from schema.listFields), each with expandable sub-fields

UI for each field row:
- Drag handle (left) — use @dnd-kit/core for drag to reorder
- Field name input (the JSON key)
- Field type dropdown (all FieldTypes from types.ts)
- Required toggle
- Delete button
- If type === 'dropdown': show inline chip editor for options (type + Enter to add, click × to remove)
- If type === 'object-optional' or 'list': show expandable sub-fields section (same field row UI, indented)

Import from example JSON (collapsed by default):
- Collapsible section at top with a textarea
- [Detect Fields] button calls inferSchemaFromJSON from packages/core
- Detected fields populate the editor — user then adjusts types and options
- Show a notice: "Review detected fields below. Change any types or add dropdown options."

Top of page:
- Back button (← Project name)
- Schema name input
- [Save Schema] button (disabled until name + at least 1 field)

On save: call updateSchema from projectStore, navigate back to ProjectView.

Install @dnd-kit/core and @dnd-kit/sortable.
```

---

**[YOU]**
- Apply and test
- Try: detect fields from pasting the naughty pack JSON sample
- Try: manually add fields
- Try: reorder via drag
- `git commit -m "feat: schema editor with inference and drag reorder"`

---

#### Prompt 3.2 🤖 GEMINI — BUILD
*Tag: Schema Editor Polish*

```
Polish the SchemaEditor.tsx in the Bracr React app.

Current state: functional but plain. Make it feel professional.

Improvements:
1. Each field row should be a well-defined card with subtle border and hover highlight
2. Field type selector should show an icon next to each type name:
   short-text: Aa, long-text: ¶, number: #, toggle: ◐, dropdown: ▾, tags: ⊞, object-optional: {}, list: []
3. Dropdown option chips: indigo background, white text, × to remove, clean pill shape
4. Sub-fields section for object-optional/list: indented with a left border line in accent color
5. The "Import from example JSON" section: collapsible with smooth animation, has a code-style textarea (monospace font, dark background)
6. Empty state for ROOT FIELDS and LIST FIELDS sections: dashed border box with [+ Add Field] centered inside
7. [Save Schema] button: fixed to bottom right of screen, always visible

Keep all existing logic intact. Only change styling and add small UX touches.
Dark mode must work throughout.
```

---

**[YOU]**
- Review and adjust if needed
- `git commit -m "feat: schema editor polish"`

---

### PHASE 4 — Content Editor

---

#### Prompt 4.1 🤖 CLAUDE — CRITICAL
*Tag: Content Editor Core — Conveyor Belt Pattern*

```
Be concise. Code only.

Build the Content Editor page for Bracr.

File: apps/web/src/pages/ContentEditor.tsx

This is the daily driver screen. The core UX principle: form on top always visible, list below. Save clears form instantly. No sheets or modals.

TOP SECTION — Context switcher:
- Dropdown: "Currently editing: [cards ▾]"
- Options: "Pack metadata" + each list field name from schema.listFields
- Switching to "Pack metadata" shows a simple field editor for rootFields values (one input per field)
- Switching to a list field shows the ADD ENTRY form

ADD ENTRY FORM (when a list field is selected):
- Render one input per sub-field of the selected list field
- Input type matches field type:
  - short-text → Input component
  - long-text → Textarea component
  - number / number-nullable → number input (nullable shows "null" placeholder, accepts empty as null)
  - toggle → Toggle component
  - dropdown → Dropdown component with the schema's defined options
  - tags → TagInput component
  - object-optional → Toggle to enable, expands sub-fields when enabled
- [Save Entry] button below form
- On save: call addEntry from store, clear all form fields, focus first field, flash confirmation

ENTRY LIST (below a divider):
- Search input (filters by any string field value)
- Filter dropdown (filters by any dropdown field value)
- Compact list rows: show first 3 fields as preview columns, ⋮ menu on right
- ⋮ menu: Edit (loads entry into form, replaces add mode), Duplicate, Delete
- When in edit mode: form title changes to "Edit Entry", shows [Save Changes] and [Cancel] buttons
- After saving edit: return to add mode, clear form
- "Last added" entry briefly highlights (300ms, indigo background flash) at top of list after save
- [Export JSON] button above list: calls buildJSON from packages/core, triggers file download
- Entry count shown: "32 entries"

Import existing JSON:
- [Import JSON] button near the top
- User pastes or uploads JSON
- Tool calls inferSchemaFromJSON, compares to current schema, shows diff
- If compatible: loads entries into the list (appends, does not overwrite)
- If incompatible: shows warning with field mismatch details

Wire up to projectStore for all data operations.
```

---

**[YOU]**
- Apply and test thoroughly
- Test: add 5 entries fast — confirm form clears and is ready immediately each time
- Test: edit an entry from the list
- Test: export JSON — open file and confirm structure matches original
- `git commit -m "feat: content editor with conveyor belt UX"`

---

#### Prompt 4.2 🤖 GEMINI — BUILD
*Tag: Content Editor Polish + Mobile*

```
Polish ContentEditor.tsx in the Bracr React app.

Improvements:
1. The ADD ENTRY section: give it a subtle top border in accent color, section label "ADD ENTRY" in small caps muted text
2. Save button: large, full-width on mobile. Fixed to bottom on mobile screens.
3. Entry list rows: alternating subtle background, smooth hover highlight, first field bold, others muted
4. The "last added" flash animation: use a CSS keyframe @keyframes flash { 0%,100%{background:transparent} 50%{background: indigo-100 dark:indigo-900} }
5. ⋮ context menu: clean floating dropdown, not a browser default
6. Search and filter bar: sticky below the divider so it stays visible when scrolling the list
7. Empty list state: "No entries yet. Add your first one above." centered, muted
8. Mobile layout: form stacks vertically with larger inputs, list rows show only 2 columns, export button moves to a floating bottom bar

Dark mode throughout.
Preserve all existing logic.
```

---

**[YOU]**
- Review on mobile viewport
- `git commit -m "feat: content editor mobile polish"`

---

### PHASE 5 — Project View

---

#### Prompt 5.1 🤖 GEMINI — BUILD
*Tag: Project View Page*

```
Build the Project View page for Bracr.

File: apps/web/src/pages/ProjectView.tsx

Layout:
- Top bar: back arrow + "Bracr" | Project name (editable on click) | version badge "v4" | sync status badge
- Section: "SCHEMAS" header
- Schema cards grid (2 col desktop, 1 col mobile):
  Each card: schema name, entry count, [Open] button → /project/:id/content/:schemaId, [Edit Schema] button → /project/:id/schema/:schemaId
- Below grid: [+ New Schema] button and [Copy from ▾] dropdown
- Copy from dropdown: grouped list of schemas from this project and other projects. Clicking one opens a small inline dialog: "New schema name: [input] [Create Copy]"
- Bottom bar: "Export entire project" label + [↓ Download JSON] button

[+ New Schema] flow:
- Opens inline (not a modal): a name input appears in the grid as a new card outline
- User types name, hits Enter or [Create] → navigates to SchemaEditor for that new schema

[↓ Download JSON]:
- Calls buildJSON for every schema in the project
- Combines into one object: { "schemas": { "truth-cards": {...}, "dare-cards": {...} } }
- Or alternatively respects each schema's own root shape
- Downloads as projectname.json

Wire to projectStore.
Use the Project and Schema types.
Dark mode. Clean, professional.
```

---

**[YOU]**
- Test full navigation flow: Dashboard → Project → Schema Editor → back → Content Editor → back
- `git commit -m "feat: project view page"`

---

### PHASE 6 — Google Drive Sync

---

#### Prompt 6.1 🤖 CLAUDE — CRITICAL
*Tag: Google OAuth + Drive API Integration*

```
Be concise. Code only.

Implement Google OAuth and Drive API integration for Bracr.

Files to create:
- packages/core/src/driveService.ts
- apps/web/src/hooks/useGoogleAuth.ts
- apps/web/src/hooks/useDriveSync.ts

driveService.ts:
- Use Google Drive API v3 (REST, no SDK — use fetch)
- Functions:
  initGoogleAuth(clientId: string): Promise<void>
  signIn(): Promise<GoogleUser>
  signOut(): void
  getAccessToken(): string | null

  createFolder(name: string, parentId?: string): Promise<string>  // returns folderId
  findFolder(name: string, parentId?: string): Promise<string | null>
  ensureFolder(name: string, parentId?: string): Promise<string>  // find or create

  uploadFile(name: string, content: string, folderId: string, existingFileId?: string): Promise<string>
  downloadFile(fileId: string): Promise<string>
  findFile(name: string, folderId: string): Promise<string | null>
  listFiles(folderId: string): Promise<{id: string, name: string, modifiedTime: string}[]>

useDriveSync.ts:
- syncProject(project: Project): Promise<void>
  1. Ensure Bracr/ root folder exists in Drive
  2. Ensure Bracr/ProjectName/ folder exists
  3. Upload project.meta.json
  4. Ensure schemas/ and content/ subfolders exist
  5. Upload each schema as schemas/schemaname.schema.json
  6. Upload each content list as content/schemaname.json
  7. Update project syncStatus to 'synced'

- loadProjectFromDrive(projectFolderId: string): Promise<Project>
  Download and reconstruct a project from Drive files

- checkConflict(project: Project): Promise<boolean>
  Compare local version with Drive meta version
  Return true if Drive version > local version

- resolveConflict(project: Project, useLocal: boolean): Promise<void>

useGoogleAuth.ts:
- Wraps driveService auth
- Returns: { isAuthed, user, signIn, signOut }
- Persists auth state

Load VITE_GOOGLE_CLIENT_ID from env.
Handle token expiry and refresh.
All Drive operations should update project syncStatus in the store before and after.
```

---

**[YOU]**
- This is complex — test carefully
- Sign in with Google, check Drive for created folders
- Upload a project, check files appear in Drive
- `git commit -m "feat: google drive sync integration"`

---

#### Prompt 6.2 🤖 GEMINI — BUILD
*Tag: Sync UI Components*

```
Build sync-related UI components for Bracr.

Files:
- apps/web/src/components/SyncStatusBadge.tsx
- apps/web/src/components/ConflictModal.tsx
- apps/web/src/components/DriveAuthStrip.tsx

SyncStatusBadge:
Props: status ('synced'|'syncing'|'conflict'|'local'|'error'), lastSynced?: string
- synced: green dot + "Synced X mins ago"
- syncing: animated blue spinner + "Uploading..."
- conflict: yellow warning icon + "Conflict — tap to resolve"
- local: gray circle + "Local only"
- error: red icon + "Sync failed — tap to retry"
Clicking conflict or error badges triggers a callback prop.

ConflictModal:
Props: localVersion, driveVersion, localDate, driveDate, onKeepLocal, onUseDrive, onCompare
- Not a full modal — a prominent banner/card that appears inline
- Shows both versions with dates
- Three buttons: [Keep mine] [Use Drive version] [Compare side by side]
- Compare opens a split view showing both JSONs side by side (use a simple two-column diff display)

DriveAuthStrip:
Props: isAuthed, user, onSignIn, onSignOut
- Not authed: subtle strip at bottom of Dashboard — "Sign in with Google to sync across devices" + [Connect Google Drive] button
- Authed: small avatar + email + "Drive connected" + [Disconnect] link
- Never intrusive — equal visual weight to the guest mode option

Integrate SyncStatusBadge into ProjectDashboard cards and ProjectView top bar.
Integrate ConflictModal into ProjectView (shown when conflict detected on load).
Integrate DriveAuthStrip into ProjectDashboard.
```

---

**[YOU]**
- Test conflict flow by manually editing version number in Drive meta file
- `git commit -m "feat: sync UI components"`

---

### PHASE 7 — PWA + Responsive Polish

---

#### Prompt 7.1 🤖 GEMINI — BUILD
*Tag: PWA Setup + Mobile Navigation*

```
Configure Bracr as a Progressive Web App and add mobile navigation.

1. PWA setup (Vite PWA plugin):
   Install: pnpm add -D vite-plugin-pwa
   Configure in vite.config.ts:
   - name: "Bracr"
   - short_name: "Bracr"
   - theme_color: "#6366f1" (indigo-500)
   - icons: generate placeholder icons at 192x192 and 512x512 (simple "PF" text on indigo background as SVG)
   - Cache strategy: StaleWhileRevalidate for assets

2. Mobile bottom navigation bar:
   File: apps/web/src/components/MobileNav.tsx
   Show only on screens < 768px (md breakpoint)
   Three tabs: 
   - Projects (home icon) → /
   - Recents (clock icon) → shows last 3 accessed projects
   - Settings (gear icon) → /settings
   Fixed to bottom, safe area inset aware (env(safe-area-inset-bottom))
   Active tab uses accent color

3. Settings page:
   File: apps/web/src/pages/Settings.tsx
   Sections:
   - Appearance: Light / Dark / System toggle
   - Account: Google Drive connection status + connect/disconnect
   - Data: Export all projects as ZIP, Clear all local data (with confirmation)
   - About: Version number, GitHub link, Buy Me a Coffee link (https://buymeacoffee.com — placeholder)
   - Credits: "Built by [your name]"

4. Add "Add to Home Screen" prompt handling:
   Listen for beforeinstallprompt event
   Show a subtle banner after user has used the app for 2+ sessions:
   "Install Bracr for offline access" [Install] [Not now]
```

---

**[YOU]**
- Test on mobile browser — confirm bottom nav works
- Test PWA install prompt
- `git commit -m "feat: PWA and mobile navigation"`

---

#### Prompt 7.2 🤖 CLAUDE — CRITICAL
*Tag: Dark Mode + Theme Consistency Audit*

```
Be concise.

Audit the entire Bracr app for dark mode consistency and theme issues.

Go through every component and page file. For each:
1. Ensure all background colors use CSS variables or Tailwind dark: variants — no hardcoded colors
2. Ensure text contrast passes WCAG AA in both modes
3. Ensure borders, shadows, and dividers adapt correctly
4. Ensure the JSON preview textarea always uses dark background + light text regardless of theme (it's a code editor)
5. Ensure sync status badge colors work in both modes (don't rely only on color — add icons too)

Also:
- Ensure all focus states are visible (ring style) in both modes
- Ensure hover states are subtle but visible in dark mode
- Check the ConflictModal, SyncStatusBadge, DriveAuthStrip components

Output a list of files changed and what was fixed. Then output the corrected code for each file.
```

---

**[YOU]**
- Toggle theme and review every screen
- `git commit -m "fix: dark mode consistency audit"`

---

### PHASE 8 — Desktop App (Tauri)

---

**[YOU]** Make sure Rust is installed: `curl --proto '=https' --tlsv1.2 -sSf https://sh.rustup.rs | sh`

---

#### Prompt 8.1 🤖 CLAUDE — CRITICAL
*Tag: Tauri Setup + File System Integration*

```
Be concise. Code only.

Set up Tauri v2 for the Bracr desktop app.

Working directory: apps/desktop/

1. Initialize Tauri in apps/desktop/ pointing to apps/web/ as the frontend:
   pnpm create tauri-app . --template vanilla
   Then configure src-tauri/tauri.conf.json to use the built web app.

2. Configure tauri.conf.json:
   - productName: "Bracr"
   - version: "0.1.0"
   - identifier: "com.bracr.app"
   - windows: title "Bracr", width 1200, height 800, minWidth 800, minHeight 600
   - allowlist: fs (all), dialog (all), shell (open)

3. Add Tauri file system commands in src-tauri/src/main.rs:
   - save_file(path: String, content: String) → saves JSON to local file system
   - open_file(path: String) → reads and returns file content
   - pick_save_path(default_name: String) → opens save dialog, returns chosen path
   - pick_open_path() → opens file picker, returns chosen path

4. Create apps/web/src/services/fileService.ts:
   - Detects if running in Tauri (window.__TAURI__ exists) or browser
   - exportJSON(filename: string, content: string):
     - Tauri: use pick_save_path + save_file commands
     - Browser: create blob URL + click download link
   - importJSON():
     - Tauri: use pick_open_path + open_file commands
     - Browser: create hidden file input + read FileReader

5. Replace all direct download/upload logic in ContentEditor and ProjectView with fileService calls.

The web app should work identically in browser and desktop — fileService abstracts the difference.
```

---

**[YOU]**
- `cd apps/desktop && pnpm tauri dev`
- Test export JSON from desktop app — confirm save dialog appears
- `git commit -m "feat: tauri desktop wrapper with file system integration"`

---

#### Prompt 8.2 🤖 GEMINI — BUILD
*Tag: Desktop App Window Polish*

```
Add desktop-specific polish to the Bracr Tauri app.

1. Custom titlebar (apps/web/src/components/TitleBar.tsx):
   Show only when running in Tauri (check window.__TAURI__)
   - Draggable area with app name "Bracr" centered
   - Window controls (minimize, maximize, close) on right — call Tauri window API
   - Left side: current project name if inside a project
   - Height: 40px, background: --bg-secondary
   Remove default Tauri decorations in tauri.conf.json (decorations: false)

2. Keyboard shortcuts (apps/web/src/hooks/useKeyboardShortcuts.ts):
   - Cmd/Ctrl + S → save current content / schema
   - Cmd/Ctrl + N → new entry (focus first field in add form)
   - Cmd/Ctrl + E → export JSON
   - Cmd/Ctrl + , → open settings
   - Escape → cancel edit mode in content editor
   Show shortcut hints in tooltips on relevant buttons.

3. App menu (src-tauri/src/main.rs):
   File menu: New Project, Open Project, Export JSON, separator, Quit
   Edit menu: standard (handled by OS)
   View menu: Toggle Dark Mode, Toggle Sidebar
   Help menu: About Bracr, GitHub, Buy Me a Coffee
```

---

**[YOU]**
- Test all keyboard shortcuts
- `git commit -m "feat: desktop titlebar and keyboard shortcuts"`

---

### PHASE 9 — Final Polish & Release Prep

---

#### Prompt 9.1 🤖 CLAUDE — CRITICAL
*Tag: JSON Export Accuracy + Edge Cases*

```
Be concise.

Audit and harden the buildJSON function in packages/core/src/jsonBuilder.ts.

Requirements:
1. Root fields must appear in the exact order defined in schema.rootFields
2. List fields must appear as arrays in the exact order defined in schema.listFields
3. null values must export as JSON null (not undefined, not empty string)
4. number-nullable fields: empty input exports as null
5. toggle fields: must export as boolean true/false (not "true"/"false" strings)
6. tags fields: must export as array of strings (empty = [])
7. object-optional fields: if disabled/null, export as null. If enabled, export as nested object with all sub-fields
8. Maintain the exact key names from schema field definitions (the JSON key, not the display label)

Write comprehensive unit tests for buildJSON covering all field types and edge cases.
Use vitest.

Also audit inferSchemaFromJSON — ensure it correctly handles the naughty pack JSON structure provided (deeply nested follow_up object, nullable timer_seconds, mixed-value arrays in topic_tags).

Output fixed functions and test file.
```

---

**[YOU]**
- `pnpm test` — all tests must pass
- Manually export the naughty pack JSON and compare with original
- `git commit -m "fix: json export accuracy and unit tests"`

---

#### Prompt 9.2 🤖 GEMINI — BUILD
*Tag: Empty States, Loading States, Error States*

```
Add proper empty, loading, and error states throughout the Bracr app.

1. Empty states (use a consistent EmptyState component):
   File: apps/web/src/components/EmptyState.tsx
   Props: icon (emoji or lucide icon), title, description, action?: {label, onClick}

   Apply to:
   - ProjectDashboard: no projects yet → "Create your first project"
   - ProjectView: no schemas yet → "Define your first schema"
   - ContentEditor: no entries yet → "Add your first entry above"
   - ContentEditor search: no results → "No entries match your search"

2. Loading states:
   - Skeleton loaders for project cards (3 placeholder cards while loading from storage)
   - Spinner overlay for Drive sync operations
   - Inline "Saving..." text that replaces [Save Entry] button during save

3. Error states:
   - Drive sync error: inline error banner with retry button (not a modal)
   - JSON import error: inline error below the import textarea with specific message
   - Schema save error: inline below save button

4. Toast notifications (lightweight, no library — build it):
   File: apps/web/src/components/Toast.tsx
   Position: bottom-right desktop, bottom center mobile
   Types: success (green), error (red), info (blue)
   Auto-dismiss after 3 seconds
   Used for: entry saved, schema saved, export complete, sync complete

Use lucide-react for icons (pnpm add lucide-react).
```

---

**[YOU]**
- Review all empty states
- Test Drive error by revoking token mid-sync
- `git commit -m "feat: empty states, loading states, toasts"`

---

#### Prompt 9.3 🤖 GEMINI — BUILD
*Tag: Buy Me a Coffee + About + Branding*

```
Add final branding and attribution to Bracr.

1. Update Settings page About section:
   - App version pulled from package.json (import { version } from '../../../package.json')
   - GitHub repo link: opens in browser (use Tauri shell.open on desktop, window.open on web)
   - Buy Me a Coffee button: styled in yellow (#FFDD00), coffee cup emoji, text "Buy me a coffee"
     Link: https://buymeacoffee.com (user will update with their own link)
   - Tagline: "Bracr is free and open source. If it saves you time, a coffee keeps it going. ☕"
   - Built with: React, Tauri, TypeScript

2. App logo:
   Create a simple SVG logo for Bracr:
   - "PF" monogram or a simple box/pack icon
   - Indigo (#6366f1) primary color
   - Works on light and dark backgrounds
   - Save as apps/web/public/logo.svg
   - Use in top bar of dashboard and desktop titlebar

3. Footer on web app only (not desktop):
   Minimal: "Bracr · Open Source · Buy me a coffee ☕ · GitHub"
   Muted text, centered, very small

4. Update page <title> tags:
   - Dashboard: "Bracr"
   - Project: "ProjectName — Bracr"
   - Schema Editor: "Edit Schema — Bracr"
   - Content Editor: "SchemaName — Bracr"
```

---

**[YOU]**
- Update Buy Me a Coffee link with your actual URL
- `git commit -m "feat: branding, buy me a coffee, about page"`

---

### PHASE 10 — Deployment & Release

---

#### Prompt 10.1 🤖 CLAUDE — CRITICAL
*Tag: CI/CD Pipeline*

```
Be concise.

Set up GitHub Actions CI/CD for Bracr.

File: .github/workflows/deploy.yml

Jobs:

1. test:
   - Runs on every push and PR
   - pnpm install
   - pnpm test (vitest)
   - pnpm build (web app)
   - Fail PR if tests fail

2. deploy-web:
   - Runs on push to main only
   - Builds apps/web/
   - Deploys to Vercel using VERCEL_TOKEN, VERCEL_ORG_ID, VERCEL_PROJECT_ID secrets
   - Posts deployment URL as PR comment

3. build-desktop:
   - Runs on push to main and on version tags (v*)
   - Matrix: ubuntu-latest, macos-latest, windows-latest
   - Builds Tauri app for each platform
   - On version tags: creates GitHub Release and uploads installers as assets

Also create .github/workflows/pr-check.yml:
- Runs on all PRs
- Checks TypeScript (tsc --noEmit)
- Checks formatting (prettier --check)

Add scripts to root package.json:
  "build": "pnpm --filter ./apps/web build"
  "test": "pnpm --filter ./packages/core test"
  "typecheck": "tsc --noEmit"
```

---

**[YOU]**
- Add Vercel secrets to GitHub repo settings
- Push to main — confirm web deploy works
- `git commit -m "ci: github actions deploy pipeline"`

---

**[YOU — Manual Steps for Desktop Release]**

To release a new desktop version:
```bash
git tag v0.1.0
git push origin v0.1.0
```
GitHub Actions will build installers for Mac, Windows, Linux and attach them to a GitHub Release automatically.

---

## 6. Release Checklist

### Before First Public Release

- [ ] Web app deployed to Vercel and accessible
- [ ] Custom domain set up (optional: bracr.app or bracr.vercel.app)
- [ ] Desktop builds tested on Mac and Windows
- [ ] Google OAuth consent screen verified (add your production domain)
- [ ] All unit tests passing
- [ ] Dark mode tested on every screen
- [ ] Mobile layout tested on real device (iPhone and Android)
- [ ] JSON export tested with real data — compare byte-for-byte with original
- [ ] Drive sync tested with two different browsers simultaneously
- [ ] Conflict resolution tested manually
- [ ] Buy Me a Coffee link updated with real URL
- [ ] GitHub repo README is complete
- [ ] Open source license confirmed (MIT)

### Launch Channels

- [ ] Post on Reddit: r/SideProject, r/webdev, r/indiegaming, r/gamedev
- [ ] Post on Product Hunt
- [ ] Post on Hacker News (Show HN)
- [ ] Share on X/Twitter with a short demo GIF (record with Loom or ScreenToGif)
- [ ] Add to awesome-lists on GitHub related to game dev tools and JSON tools

---

## 7. README Templates

### Root README.md

```markdown
# Bracr

> Schema-first JSON content editor for indie developers and content creators.

Define your data structure once. Fill it in with a clean form. Export perfect JSON.

**No backend. No subscription. No lock-in.**

[Try it free →](https://bracr.vercel.app) · [Download desktop app](https://github.com/YOUR_USERNAME/bracr/releases) · [Buy me a coffee ☕](https://buymeacoffee.com/YOUR_LINK)

---

## What is Bracr?

If your app or game runs on JSON data packs — card games, quiz apps, flashcard sets, dialogue trees, item databases — Bracr gives you and your team a clean UI to create and manage that content without touching code.

## Features

- **Schema-first** — define your JSON structure visually or paste an example and let Bracr detect it
- **Fast content entry** — conveyor belt form: fill, save, instantly ready for the next entry
- **Google Drive sync** — optional. Save your projects to Drive and access them from any device
- **Works everywhere** — web app, desktop app (Mac/Windows/Linux), mobile browser PWA
- **No account required** — guest mode with local storage, just works
- **Open source** — MIT license, host it yourself if you want

## Getting Started

### Web App
Visit [bracr.vercel.app](https://bracr.vercel.app) — no install needed.

### Desktop App
Download the latest release for your platform from [Releases](https://github.com/YOUR_USERNAME/bracr/releases).

### Self-host
```bash
git clone https://github.com/YOUR_USERNAME/bracr.git
cd bracr/apps/web
pnpm install && pnpm dev
```

## Development

See [PACKFORGE_MASTER_PLAN.md](./PACKFORGE_MASTER_PLAN.md) for the full build plan.

```bash
pnpm install
pnpm dev          # web app on localhost:5173
pnpm test         # run unit tests
pnpm build        # production build
```

## License

MIT © [Your Name]
```

---

### apps/web/README.md

```markdown
# Bracr — Web App

React 18 + TypeScript + Vite + Tailwind CSS + Zustand

## Dev

```bash
pnpm install
pnpm dev
```

## Env

```
VITE_GOOGLE_CLIENT_ID=your_google_oauth_client_id
```

## Build

```bash
pnpm build
# Output in dist/
```
```

---

### apps/desktop/README.md

```markdown
# Bracr — Desktop App

Tauri v2 wrapper around the web app.

## Requirements

- Rust (install via rustup)
- Node.js 18+
- pnpm

## Dev

```bash
# Build web app first
cd ../web && pnpm build

# Then run Tauri dev
cd ../desktop
pnpm tauri dev
```

## Build Installers

```bash
pnpm tauri build
# Output in src-tauri/target/release/bundle/
```
```

---

*Bracr Master Plan — v1.0*
*Keep it simple. Keep it fast. Ship it.*
