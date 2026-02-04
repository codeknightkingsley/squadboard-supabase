# SquadBoard - AGENTS.md

> **📚 Documentation Structure:**
> - **ATOMIC_ARCHITECTURE.md** - High-level architecture, reasoning units
> - **AGENTS.md** (this file) - Gotchas, implementation details, commands
> - **PRD.md** - Product requirements and feature specs

## Quick Start

```bash
# Start local server
cd /Users/stevekingsley/Desktop/repo
python3 -m http.server 3000

# Access app
open http://localhost:3000
```

## App Purpose
Real-time collaborative Kanban board connected to LYFE Supabase database. Designed for AI agent task management with Steve 🦞, Jarvis 🤖, Kodi 🛠️, Scout 🔭, and Teddy 📋.

## File Structure
```
repo/
├── index.html               # Single-file app (HTML + CSS + JS, ~1900 lines)
├── AGENTS.md                # This file
├── ATOMIC_ARCHITECTURE.md   # Architecture docs
└── PRD.md                   # Product requirements
```

## Current Features (v2.0)

### ✅ Implemented
| Feature | Description |
|---------|-------------|
| **Sidebar Layout** | Vertical agent panel (220px), collapsible to 60px |
| **Agent Filtering** | Click agent to filter tasks |
| **Task Creation Modal** | Full form with auto-suggest agent |
| **Drag-and-Drop** | HTML5 DnD between columns |
| **Task Cards** | Priority dots, due dates, comment counts |
| **Task Modal** | Edit all fields, move status, delete |
| **Comments** | Add/view comments on tasks |
| **Activity Feed** | Recent actions in sidebar |
| **Keyboard Shortcuts** | `n`, `/`, `?`, `j/k`, `1-4`, etc. |
| **Search** | Real-time filter by title |
| **Quick Actions** | Hover menu on cards |

### Keyboard Shortcuts
| Key | Action |
|-----|--------|
| `n` | New task |
| `/` | Focus search |
| `?` | Show help |
| `Escape` | Close modal |
| `j` / `↓` | Next task |
| `k` / `↑` | Previous task |
| `Enter` | Open selected task |
| `1-4` | Move to column |

## Component Map

### 1. Header
- App title, connection status, search input, "+ New Task" button
- Search filters tasks in real-time

### 2. Sidebar (left)
- **Agent List**: 5 agents with status indicators (●/⭘/○)
- **Global Rules**: Write only, no overwrite, no external sends, time/budget caps
- **Activity Feed**: Recent task actions with timestamps
- **Toggle**: Collapse to icons-only (60px)

### 3. Board Grid
- 4 columns: To Do, In Progress, In Review, Done
- **Status Mapping**:
  - `todo` → To Do
  - `in_progress` → In Progress
  - `in_progress` + `metadata.uiColumn='inreview'` → In Review
  - `archived` → Done

### 4. Task Cards
- Draggable between columns
- Shows: agent emoji, priority dot, title, due date, comment count
- Click → Task Modal
- Hover → Quick actions menu (⋮)

### 5. Task Modal
- Editable: title, description, assignee, priority
- Meta info: created date, due date, status
- Status buttons: move to any column
- Comments section with add/view
- Delete with confirmation

### 6. Create Task Modal
- Form: title (required), description, assignee dropdown, priority
- Auto-suggests agent based on keywords:
  - "research", "find" → Scout
  - "build", "fix", "code" → Kodi
  - "draft", "email" → Teddy
  - default → Steve

## Supabase Integration

**Project**: `onyrtmjnktrqcwvmemlp.supabase.co`
**Table**: `tasks`

### API Functions
| Function | Method | Endpoint |
|----------|--------|----------|
| `loadTasks()` | GET | `/rest/v1/tasks?order=created_at.desc` |
| `createTask()` | POST | `/rest/v1/tasks` |
| `saveTaskFromModal()` | PATCH | `/rest/v1/tasks?id=eq.{id}` |
| `moveTask()` | PATCH | `/rest/v1/tasks?id=eq.{id}` |
| `deleteTask()` | DELETE | `/rest/v1/tasks?id=eq.{id}` |

### Database Schema
**Table: `tasks`**
| Column | Type | Notes |
|--------|------|-------|
| id | uuid | PK |
| title | text | Required |
| status | enum | 'todo', 'in_progress', 'archived' |
| priority | enum | 'low', 'medium', 'high' |
| description | text | Optional |
| due_date | timestamp | Optional |
| metadata | jsonb | `{emoji, uiColumn, assignee_name, comments[]}` |
| created_at | timestamp | Auto |

## Critical Gotchas

### Security
- ⚠️ Service role key is in frontend (acceptable for private use, not production)
- All user input escaped with `escapeHtml()` function

### State Management
- `app.tasks` - Array of all tasks from Supabase
- `app.activities` - Activity log (persisted to localStorage)
- `app.selectedAgent` - Current filter (null = all)
- `app.selectedTaskId` - Keyboard navigation selection
- `app.searchQuery` - Current search filter

### Event Handling
- Keyboard shortcuts disabled when typing in input/textarea
- Modal closes on Escape or overlay click
- Drag-drop uses optimistic updates (move card, then sync)

### localStorage Keys
| Key | Purpose |
|-----|---------|
| `squadboard-sidebar-collapsed` | Sidebar state |
| `squadboard-activities` | Activity feed history |

## Development

### Making Changes
1. Edit `index.html` directly
2. Refresh browser to see changes
3. Test: create, edit, move, delete tasks
4. Check console for errors
5. Commit and push

### Testing Checklist
- [ ] Page loads without errors
- [ ] Tasks load from Supabase
- [ ] Create task works
- [ ] Drag-drop works
- [ ] Keyboard shortcuts work
- [ ] Search filters correctly
- [ ] Comments save
- [ ] Activity feed updates
- [ ] Mobile layout works

## GitHub
**Repo**: https://github.com/codeknightkingsley/squadboard-supabase

```bash
git add . && git commit -m "message" && git push
```

## Recent Changes
- 2026-02-04: Initial Supabase connection
- 2026-02-04: Phase 1 - Task modal, drag-drop, enhanced cards
- 2026-02-04: Phase 2 - Sidebar layout, collapsible
- 2026-02-04: Phase 3 - Keyboard shortcuts, search, quick actions
- 2026-02-04: Phase 4 - Activity feed, comments system
