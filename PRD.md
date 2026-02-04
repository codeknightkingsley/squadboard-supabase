# SquadBoard v2.0 - Product Requirements Document

**Version:** 2.0  
**Date:** 2026-02-04  
**Owner:** Steve Kingsley  
**Status:** Draft

---

## Executive Summary

SquadBoard is a real-time collaborative Kanban board designed for AI agent task management. This PRD outlines improvements to transform the current MVP into a polished, production-ready interface inspired by Linear, ClickUp, and Plane.

---

## Current State Analysis

### What Works ✅
- Clean dark theme (GitHub-style)
- Supabase backend with real persistence
- Agent filtering system
- Task modal with full editing
- Auto-refresh with visibility API
- Mobile responsive layout

### What Needs Improvement 🔧
| Issue | Impact | Priority |
|-------|--------|----------|
| `prompt()` for task creation | Poor UX, jarring | 🔴 Critical |
| No drag-and-drop | Missing core Kanban feature | 🔴 Critical |
| No priority/due date on cards | Information hidden | 🟡 High |
| Service role key exposed | Security vulnerability | 🔴 Critical |
| Agent panel horizontal scroll | Awkward on desktop | 🟡 High |
| No keyboard shortcuts | Power user friction | 🟢 Medium |

---

## Goals

1. **Modern UI** — Match quality of Linear/ClickUp
2. **Intuitive UX** — Zero learning curve for Kanban users
3. **Agent-First** — Optimize for AI agent task management
4. **Secure** — No exposed secrets, proper RLS
5. **Fast** — Optimistic updates, minimal re-renders

---

## Feature Specifications

### Phase 1: Core UI Fixes (Priority: Critical)

#### 1.1 Task Creation Modal
**Replace browser `prompt()` with proper modal form**

```
┌─────────────────────────────────────┐
│ ✨ New Task                      ✕  │
├─────────────────────────────────────┤
│ Title                               │
│ ┌─────────────────────────────────┐ │
│ │ Enter task title...             │ │
│ └─────────────────────────────────┘ │
│                                     │
│ Description (optional)              │
│ ┌─────────────────────────────────┐ │
│ │                                 │ │
│ └─────────────────────────────────┘ │
│                                     │
│ ┌──────────┐  ┌──────────┐         │
│ │ Assignee │  │ Priority │         │
│ │ 🦞 Steve▼│  │ Medium ▼ │         │
│ └──────────┘  └──────────┘         │
│                                     │
│              [Cancel] [Create Task] │
└─────────────────────────────────────┘
```

**Acceptance Criteria:**
- [ ] Modal opens on "+ New Task" click
- [ ] Auto-suggest agent based on keywords (e.g., "research" → Scout)
- [ ] Keyboard: Enter to submit, Escape to cancel
- [ ] Focus trap within modal
- [ ] Create button disabled until title entered

#### 1.2 Drag-and-Drop
**Native HTML5 drag-and-drop for task cards**

**Acceptance Criteria:**
- [ ] Drag cards between columns
- [ ] Visual drop indicator on hover
- [ ] Card shows "grabbing" cursor
- [ ] Optimistic update (move immediately, sync after)
- [ ] Revert on API failure with toast notification
- [ ] Touch support for mobile

#### 1.3 Enhanced Task Cards
**Show more info at-a-glance**

```
┌─────────────────────────┐
│ 🦞                  🔴  │  ← Agent emoji + Priority dot
│ Fix authentication bug  │
│ 📅 Feb 6 · 💬 3         │  ← Due date + comment count
└─────────────────────────┘
```

**Acceptance Criteria:**
- [ ] Priority indicator (dot or badge): 🔴 High, 🟡 Medium, 🟢 Low
- [ ] Due date with relative time ("Today", "Tomorrow", "Feb 6")
- [ ] Overdue dates in red
- [ ] Comment count if > 0
- [ ] Truncate long titles with ellipsis

---

### Phase 2: Agent Panel Redesign (Priority: High)

#### 2.1 Vertical Sidebar Layout
**Move agents to left sidebar instead of horizontal bar**

```
┌──────────────────────────────────────────────────────────┐
│ 🦞 SquadBoard              ● Connected    [+ New Task]   │
├─────────────┬────────────────────────────────────────────┤
│ AGENTS      │  📋 To Do    ⚡ Progress   👀 Review   ✅   │
│             │  ┌───────┐   ┌───────┐    ┌───────┐       │
│ 🦞 Steve    │  │ Task  │   │ Task  │    │ Task  │       │
│   Router  ● │  │       │   │       │    │       │       │
│             │  └───────┘   └───────┘    └───────┘       │
│ 🤖 Jarvis   │                                           │
│   Orch.   ○ │                                           │
│             │                                           │
│ 🛠️ Kodi     │                                           │
│   Builder ○ │                                           │
│             │                                           │
│ 🔭 Scout    │                                           │
│   Research○ │                                           │
│             │                                           │
│ 📋 Teddy    │                                           │
│   Executor○ │                                           │
│─────────────│                                           │
│ GLOBAL RULES│                                           │
│ ⚡ Write only│                                           │
│ ⚡ No extern │                                           │
└─────────────┴────────────────────────────────────────────┘
```

**Acceptance Criteria:**
- [ ] Sidebar width: 200px, collapsible to 60px (icons only)
- [ ] Agent cards show: emoji, name, role, status indicator
- [ ] Click agent to filter tasks
- [ ] Active filter highlighted with accent color
- [ ] "All" option to clear filter
- [ ] Collapse button for more board space

#### 2.2 Agent Status Modes
**Visual mode indicators**

| Mode | Color | Icon |
|------|-------|------|
| Active | Green | ● |
| Focus | Orange | ⭘ |
| Standby | Gray | ○ |

---

### Phase 3: Keyboard & Power User Features (Priority: Medium)

#### 3.1 Keyboard Shortcuts
| Key | Action |
|-----|--------|
| `N` | New task |
| `Escape` | Close modal |
| `↑/↓` | Navigate tasks |
| `Enter` | Open selected task |
| `1-4` | Move to column (1=To Do, 2=Progress, etc.) |
| `/` | Search/filter |
| `?` | Show shortcuts help |

#### 3.2 Quick Actions
**Hover menu on task cards**
- Move to → (submenu with columns)
- Edit
- Delete

---

### Phase 4: Real-time & Collaboration (Priority: Low - Future)

#### 4.1 Supabase Realtime
- Subscribe to `tasks` table changes
- Remove polling, get instant updates
- Show "Task updated by X" toast

#### 4.2 Activity Feed
- Sidebar section showing recent activity
- Format: "🛠️ Kodi moved 'Fix bug' to Done"
- Last 10 activities, scrollable

#### 4.3 Comments System
- Comment thread in task modal
- Markdown support
- @mentions (for agents)

---

## Security Fixes (Critical)

### Move to Anonymous Key + RLS
1. Replace `service_role` key with `anon` key
2. Add RLS policies to `tasks` table:
   - `SELECT`: authenticated or anon
   - `INSERT`: authenticated or anon  
   - `UPDATE`: authenticated or anon
   - `DELETE`: authenticated or anon

*For MVP, open policies are acceptable. Add auth later.*

---

## Technical Implementation

### File Structure (Keep Single File for MVP)
```
squadboard-supabase/
├── index.html          # Main app (HTML + CSS + JS)
├── AGENTS.md           # Agent instructions
├── ATOMIC_ARCHITECTURE.md  # Architecture docs
├── PRD.md              # This document
└── README.md           # Setup instructions
```

### Dependencies
- None (vanilla JS, no build step)
- Supabase REST API only

### Browser Support
- Chrome 90+
- Firefox 88+
- Safari 14+
- Edge 90+

---

## Success Metrics

| Metric | Target |
|--------|--------|
| Task creation time | < 5 seconds |
| Drag-drop success rate | 100% |
| Mobile usability | Touch DnD works |
| Page load time | < 2 seconds |
| Zero console errors | ✓ |

---

## Implementation Phases

| Phase | Scope | Estimate |
|-------|-------|----------|
| **Phase 1** | Task modal, DnD, card enhancements | 2-3 hours |
| **Phase 2** | Sidebar layout, agent redesign | 1-2 hours |
| **Phase 3** | Keyboard shortcuts, quick actions | 1 hour |
| **Phase 4** | Realtime, activity, comments | 3-4 hours |

---

## Appendix: Research Sources

### Open Source Inspiration
- [Plane](https://github.com/makeplane/plane) - 45k stars, modern UI
- [Focalboard](https://github.com/mattermost-community/focalboard) - 26k stars, Notion-like
- [Planka](https://github.com/plankanban/planka) - Trello alternative
- [WeKan](https://github.com/wekan/wekan) - Feature-rich

### Design Inspiration
- Linear - Keyboard-first, minimal
- ClickUp - Feature-rich, customizable
- GitHub Projects - Clean, integrated
