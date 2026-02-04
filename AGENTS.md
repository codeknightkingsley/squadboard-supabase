# SquadBoard - AGENTS.md

> **📚 Documentation Structure:**
> - **ATOMIC_ARCHITECTURE.md** (this folder) - High-level architecture, reasoning units, component breakdown
> - **AGENTS.md** (this file) - Gotchas, implementation details, common mistakes

## App Purpose
Real-time collaborative Kanban board connected to LYFE Supabase database. Joe and Steve can create, edit, move, and manage tasks together from any device.

## Architecture Overview
```
squadboard-supabase/
├── index.html               # Single-file app: HTML + CSS + JS
├── ATOMIC_ARCHITECTURE.md   # Architecture map (reasoning units)
└── AGENTS.md                # This file (gotchas & details)
```

**How to use these docs:**
1. Start with ATOMIC_ARCHITECTURE.md to understand the system structure
2. Use this AGENTS.md file for specific implementation gotchas
3. Both docs live in the same folder for easy reference

## Component Breakdown

### 1. Header (lines ~25-35)
- App title with 🦞 emoji
- Connection status indicator (green/red dot)
- "+ New Task" button
- **Key**: Connection status shows ● Connected or ● Offline

### 2. Agent Panel (NEEDS ADDITION - lines ~50-80)
- Shows 5 controller agents: Steve 🦞, Jarvis 🤖, Kodi 🛠️, Scout 🔭, Teddy 📋
- Each agent has: name, model, status (Active/Focus/Standby), current task
- Click agent to filter tasks by assignee
- **Key**: Agents stored in `controller_agents` table in Supabase

### 3. Board Grid (lines ~90-120)
- 4 columns: To Do, In Progress, In Review, Done
- Each column: header with count, task container
- **Status Mapping**: 
  - LYFE `todo` → Column "To Do"
  - LYFE `in_progress` → Column "In Progress" 
  - LYFE `in_progress` + metadata.uiColumn='inreview' → Column "In Review"
  - LYFE `archived` → Column "Done"

### 4. Task Cards (lines ~200-250)
- Shows: emoji, title, assignee
- Click → Opens Task Modal (NOT edit prompt)
- Right-click → Context menu (move/delete)
- **Key**: Cards get `.processing` class during operations

### 5. Task Modal (NEEDS ADDITION - lines ~300-400)
- Popup overlay showing full task details
- Fields: title (editable), description (editable), status, priority, assigned_to (dropdown), due_date, created_at, comments
- Actions: Save, Move Status, Delete
- **Key**: Modal should feel like ClickUp detail view

### 6. Context Menu (lines ~420-480)
- Right-click on task card
- Options: Move to To Do, In Progress, In Review, Done, Delete
- **Key**: Properly removes event listeners on close (memory leak prevention)

### 7. Supabase Integration (lines ~500-600)
- `supabaseFetch()` - wrapper for REST API calls
- `loadTasks()` - fetches from `tasks` table
- `createTask()`, `editTask()`, `moveTask()`, `deleteTask()` - CRUD operations
- **Key**: Uses `service_role` key (bypasses RLS)

## Data Flow
1. Page loads → `init()` → `loadTasks()` → `render()`
2. User creates task → `createTask()` → POST to Supabase → `loadTasks()` → `render()`
3. Auto-refresh every 10 seconds when tab visible
4. Real-time sync: Both users see changes instantly

## Critical Gotchas
- DO NOT use `innerHTML` with unescaped text (XSS risk)
- ALWAYS clean up event listeners (memory leaks)
- LYFE `assigned_to` is UUID field, not text - store name in metadata
- `inreview` column uses metadata.uiColumn to persist state
- DELETE returns 204 (no JSON body) - handle specially
- Auto-refresh pauses when tab hidden (saves API calls)

## LYFE Database Schema
**Table**: `tasks`
- `id`: uuid (PK)
- `title`: text
- `status`: enum ('todo', 'in_progress', 'archived')
- `priority`: enum ('low', 'medium', 'high')
- `assigned_to`: uuid (references controller_agents)
- `description`: text
- `due_date`: timestamp
- `metadata`: jsonb (stores emoji, uiColumn, comments)
- `created_at`: timestamp
- `updated_at`: timestamp

**Table**: `controller_agents`
- `id`: uuid (PK)
- `name`: text (Steve, Jarvis, Kodi, Scout, Teddy)
- `model`: text (claude-opus-4-5, gpt-5.2, etc.)
- `system_prompt`: text
- `status`: text (Active, Focus, Standby)

## Recent Changes
- 2026-02-04: Added service_role key, fixed memory leaks, added context menu
- 2026-02-04: Need to add Agent Panel and Task Modal

## Agent Instructions
If modifying:
1. Check this map to find the component
2. Read AGENTS.md in this folder for gotchas
3. Test all CRUD operations after changes
4. Verify mobile responsive layout
5. Check browser console for errors
