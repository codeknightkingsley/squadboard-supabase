# SquadBoard - Atomic Architecture Map (Reasoning Units)

## Why this doc exists
This document decomposes SquadBoard into independent atomic reasoning units ("atoms") so we can:
- Navigate the system quickly ("where do I build X?")
- Change one area without accidentally breaking others
- Validate behavior with clear invariants

## How to use this doc (practical workflow)
- **Step 1** — Identify the atom(s) you're touching using the "Atom Index"
- **Step 2** — Confirm dependencies (Supabase, agents, other atoms) before coding
- **Step 3** — Preserve invariants listed under that atom (these are the "must not break" rules)
- **Step 4** — Verify with evidence: run the app, check console, test the feature

---

## Atom Index

| ID | Atom Name | Purpose | Location (lines) |
|---|---|---|---|
| A1 | Config | Supabase connection settings | 25-32 |
| A2 | App State | Global state management (tasks, agents, UI) | 34-42 |
| A3 | Agent Panel | Display and filter by agents | 44-110, 295-330 |
| A4 | Task Board | Kanban columns with cards | 112-200, 380-450 |
| A5 | Task Modal | Detail view for editing tasks | 550-750 |
| A6 | Supabase API | All database operations | 480-530 |
| A7 | Auto-refresh | Background sync when tab visible | 76-95, 410-430 |

---

## Atom Details

### A1: Config
**Purpose:** Central configuration for Supabase connection

**Interface:**
```javascript
const CONFIG = {
    supabaseUrl: string,
    supabaseKey: string,
    table: 'tasks'
}
```

**Dependencies:**
- Supabase project `onyrtmjnktrqcwvmemlp`
- `tasks` table with proper RLS or service_role key

**Invariants (MUST NOT BREAK):**
- `supabaseKey` must have write access (service_role or RLS policy)
- `table` must exist in Supabase
- URL must be valid Supabase REST endpoint

**Evidence of working:**
- Page loads with "● Connected" status
- Console shows "SquadBoard connected to Supabase"

---

### A2: App State
**Purpose:** Central state container for the entire app

**Interface:**
```javascript
app = {
    tasks: Array<Task>,
    agents: Array<Agent>,
    selectedAgent: string|null,
    currentModal: HTMLElement|null,
    refreshInterval: number|null
}
```

**Dependencies:**
- A1 (Config) for Supabase connection
- A6 (Supabase API) to populate state

**Invariants:**
- `tasks` is always an array (never null)
- `agents` is hardcoded, never fetched
- `selectedAgent` filters render but doesn't affect data fetch

**Evidence of working:**
- Tasks display in correct columns
- Agent panel shows all 5 agents
- Selecting agent filters task list

---

### A3: Agent Panel
**Purpose:** Display squad agents, show status, enable filtering

**Interface:**
```javascript
// Render
renderAgents() → void

// Interaction
selectAgent(name: string) → void
```

**Dependencies:**
- A2 (App State) for `app.agents` and `app.selectedAgent`

**Invariants:**
- Always shows exactly 5 agents: Steve, Jarvis, Kodi, Scout, Teddy
- Clicking agent toggles filter (click again to clear)
- Visual highlight shows selected agent
- Status colors: Active (green), Focus (orange), Standby (gray)

**Evidence of working:**
- All 5 agents visible with emoji, name, model, status
- Click Steve → only Steve's tasks show
- Click Steve again → all tasks show

---

### A4: Task Board
**Purpose:** Render Kanban columns with draggable task cards

**Interface:**
```javascript
// Render
render() → void  // Uses app.tasks, app.selectedAgent

// Status Mapping (LYFE → SquadBoard)
todo → 'backlog' (To Do column)
in_progress → 'inprogress' (In Progress column) 
in_progress + metadata.uiColumn='inreview' → 'inreview' (In Review column)
archived → 'done' (Done column)
```

**Dependencies:**
- A2 (App State) for tasks and selected agent filter
- A3 (Agent Panel) for filter state

**Invariants:**
- Every task appears in exactly one column
- Task count badge matches visible cards
- Empty columns show "No tasks" message
- Clicking card opens A5 (Task Modal)

**Evidence of working:**
- 4 columns display with correct headers
- Tasks sorted by created_at desc
- Count badges accurate
- Click task → modal opens

---

### A5: Task Modal
**Purpose:** Full-featured task editor (ClickUp-style detail view)

**Interface:**
```javascript
// Open
openTaskModal(task: Task) → void

// Actions
saveTaskFromModal(taskId: string) → void
moveTaskFromModal(taskId: string, status: string, uiColumn?: string) → void
deleteTaskFromModal(taskId: string) → void
closeModal() → void
```

**Dependencies:**
- A2 (App State) to read task data
- A6 (Supabase API) to persist changes
- A4 (Task Board) to refresh after changes

**Invariants:**
- Modal shows all task fields: title, description, assignee, priority, dates, status
- Save updates task without closing modal (user can continue editing)
- Move status buttons immediately apply and close modal
- Delete requires confirmation
- ESC key or click overlay closes modal

**Evidence of working:**
- Click task card → modal opens with correct data
- Edit title → Save → task updates in background
- Click "In Progress" → task moves, modal closes, board refreshes
- Click Delete → confirm → task removed

---

### A6: Supabase API
**Purpose:** All database operations via REST API

**Interface:**
```javascript
// Generic fetch
supabaseFetch(path: string, options?: RequestInit) → Promise<any>

// CRUD operations
loadTasks() → Promise<void>
createTask() → Promise<void>
editTask(task: Task) → Promise<void>
moveTask(id: string, status: string) → Promise<void>
deleteTask(id: string) → Promise<void>
```

**Dependencies:**
- A1 (Config) for URL and API key
- Supabase REST API availability

**Invariants:**
- All requests include `apikey` and `Authorization` headers
- POST includes `Prefer: return=representation` header
- 204 responses (DELETE) handled gracefully (no JSON parse)
- Errors thrown with descriptive messages

**Evidence of working:**
- Network tab shows 200 responses
- Console shows "Task created" logs
- Failed requests show alert with error message

---

### A7: Auto-refresh
**Purpose:** Background sync when tab is visible

**Interface:**
```javascript
startRefreshInterval() → void
stopRefreshInterval() → void
```

**Dependencies:**
- A2 (App State) for interval management
- A6 (Supabase API) to fetch updates
- Browser `visibilitychange` API

**Invariants:**
- Only runs when tab is visible (saves API calls)
- Interval is 10 seconds (not too aggressive)
- Previous interval cleared before starting new one
- Cleanup on page unload

**Evidence of working:**
- Switch to tab → tasks refresh
- Switch away → no network requests in background
- Console shows regular "SquadBoard connected" messages

---

## Cross-Cutting Concerns

### Memory Management
- **Rule:** Every `addEventListener` must have matching `removeEventListener`
- **Location:** A5 (Task Modal), Context Menu (if re-added)
- **Evidence:** No detached DOM nodes in Chrome DevTools Memory tab

### XSS Prevention
- **Rule:** Never use `innerHTML` with user input
- **Location:** A4 (Task Board), A5 (Task Modal)
- **Method:** `escapeHtml()` function creates text node

### Mobile Responsive
- **Rule:** Must work on phone screens
- **Location:** CSS media queries at lines ~130-140
- **Test:** Resize browser to <768px, verify single column layout

---

## Testing Checklist

When modifying ANY atom:
- [ ] Page loads without console errors
- [ ] Can create new task
- [ ] Can edit task title/description
- [ ] Can move task between columns
- [ ] Can delete task
- [ ] Agent filter works
- [ ] Modal opens and closes properly
- [ ] Changes persist after refresh
- [ ] Mobile layout works

---

## Recent Changes Log

| Date | Atom | Change |
|---|---|---|
| 2026-02-04 | A1 | Added service_role key |
| 2026-02-04 | A3 | Added Agent Panel |
| 2026-02-04 | A5 | Added Task Modal |
| 2026-02-04 | A6 | Fixed memory leaks in event listeners |

---

## AGENTS.md Cross-Reference

For detailed implementation notes, see `AGENTS.md` in this folder:
- Agent Panel implementation details
- Task Modal form handling
- Supabase schema documentation
- Gotchas and common mistakes
