# SquadBoard - Atomic Architecture Map

## Why This Doc Exists
Decomposes SquadBoard into independent reasoning units so we can:
- Navigate quickly ("where do I build X?")
- Change one area without breaking others
- Validate behavior with clear invariants

## Atom Index

| ID | Atom | Purpose | Key Functions |
|----|------|---------|---------------|
| A1 | Config | Supabase connection | `CONFIG` object |
| A2 | App State | Global state container | `app` object |
| A3 | Sidebar | Agent panel, activity feed, rules | `renderAgents()`, `renderActivityFeed()` |
| A4 | Board | Kanban columns with DnD | `render()`, `setupDropZones()` |
| A5 | Task Cards | Card display with priority/dates | In `render()` |
| A6 | Task Modal | Edit view with comments | `openTaskModal()`, `saveTaskFromModal()` |
| A7 | Create Modal | New task form | `openCreateModal()`, `createTask()` |
| A8 | Supabase API | All REST operations | `supabaseFetch()`, CRUD functions |
| A9 | Keyboard | Shortcuts & navigation | `setupKeyboardShortcuts()` |
| A10 | Search | Filter by title | `setupSearchInput()` |

---

## Atom Details

### A1: Config
```javascript
const CONFIG = {
    supabaseUrl: 'https://onyrtmjnktrqcwvmemlp.supabase.co',
    supabaseKey: '[service_role_key]',
    table: 'tasks'
}
```
**Invariants:**
- Key must have write access
- URL must be valid Supabase endpoint

---

### A2: App State
```javascript
app = {
    tasks: [],              // All tasks from Supabase
    agents: [],             // Hardcoded agent list
    activities: [],         // Activity log
    selectedAgent: null,    // Filter state
    selectedTaskId: null,   // Keyboard selection
    searchQuery: '',        // Search filter
    refreshInterval: null,  // Auto-refresh timer
    currentModal: null      // Active modal element
}
```
**Invariants:**
- `tasks` is always an array
- `selectedAgent` is null or agent name string

---

### A3: Sidebar
**Components:**
- Agent list with status indicators
- Global rules display
- Activity feed (last 20 actions)
- Collapse toggle

**Functions:**
- `renderAgents()` - Renders agent cards
- `renderActivityFeed()` - Renders activity list
- `toggleSidebar()` - Collapse/expand
- `logActivity(action, taskTitle, emoji)` - Add activity

**Invariants:**
- 5 agents always shown (Steve, Jarvis, Kodi, Scout, Teddy)
- Collapsed state persisted to localStorage

---

### A4: Board
**Columns:** To Do, In Progress, In Review, Done

**Functions:**
- `render()` - Renders all task cards
- `setupDropZones()` - Initializes drag-drop

**Status Mapping:**
| Supabase Status | UI Column |
|-----------------|-----------|
| `todo` | To Do |
| `in_progress` | In Progress |
| `in_progress` + `uiColumn='inreview'` | In Review |
| `archived` | Done |

**Invariants:**
- Every task appears in exactly one column
- Count badges match visible cards
- Empty columns show "No tasks"

---

### A5: Task Cards
**Display:**
- Agent emoji (top-left)
- Priority dot (top-right): 🔴 High, 🟡 Medium, 🟢 Low
- Title (truncated to 2 lines)
- Due date with relative time
- Comment count

**Interactions:**
- Click → Open task modal
- Drag → Move between columns
- Hover → Quick actions menu

---

### A6: Task Modal
**Editable Fields:**
- Title, Description, Assignee, Priority

**Read-only:**
- Created date, Due date, Current status

**Actions:**
- Save changes
- Move to status (4 buttons)
- Delete (with confirmation)
- Add comment

**Invariants:**
- ESC or overlay click closes modal
- Comments persist to `metadata.comments[]`

---

### A7: Create Modal
**Fields:**
- Title (required)
- Description (optional)
- Assignee dropdown
- Priority dropdown

**Auto-suggest Logic:**
| Keywords | Suggested Agent |
|----------|-----------------|
| research, find, look up | Scout 🔭 |
| build, fix, code, implement | Kodi 🛠️ |
| draft, email, document | Teddy 📋 |
| default | Steve 🦞 |

---

### A8: Supabase API
```javascript
supabaseFetch(path, options) → Promise<any>
loadTasks() → Promise<void>
createTask() → Promise<void>
saveTaskFromModal(id) → Promise<void>
moveTask(id, status, uiColumn?) → Promise<void>
deleteTask(id) → Promise<void>
addComment(id) → Promise<void>
```

**Headers:**
- `apikey` and `Authorization` with service key
- `Prefer: return=representation` on POST

**Invariants:**
- 204 responses handled (no JSON parse)
- Errors shown via alert()

---

### A9: Keyboard Shortcuts
| Key | Action | Context |
|-----|--------|---------|
| `n` | New task | Global |
| `/` | Focus search | Global |
| `?` | Show help | Global |
| `Escape` | Close modal | Modal open |
| `j` / `↓` | Next task | No modal |
| `k` / `↑` | Previous task | No modal |
| `Enter` | Open task | Task selected |
| `1-4` | Move to column | Task selected |

**Invariants:**
- Disabled when typing in input/textarea
- Selection visual: blue outline

---

### A10: Search
- Input in header
- Filters `app.tasks` by title match
- Case-insensitive
- Real-time (on keyup)

---

## Cross-Cutting Concerns

### Memory Management
- Event listeners cleaned up on modal close
- Drag-drop handlers attached once

### XSS Prevention
- All user input through `escapeHtml()`
- Never raw `innerHTML` with user data

### Mobile Responsive
- < 768px: Sidebar becomes horizontal bar
- Touch drag-drop supported

### localStorage
| Key | Data |
|-----|------|
| `squadboard-sidebar-collapsed` | boolean |
| `squadboard-activities` | Activity[] |

---

## Testing Checklist

When modifying ANY atom:
- [ ] Page loads without console errors
- [ ] Tasks load from Supabase
- [ ] Can create new task via modal
- [ ] Can edit task in modal
- [ ] Drag-drop moves tasks
- [ ] Keyboard shortcuts work
- [ ] Search filters correctly
- [ ] Comments save and display
- [ ] Activity feed updates
- [ ] Mobile layout works
- [ ] Sidebar collapse works
