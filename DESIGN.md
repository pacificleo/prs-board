# Nullboard (prs-board) - Design Document

## Project Overview

Nullboard is a minimalist, offline-first kanban board / task list manager delivered as a **single HTML file** (~5,185 lines). It requires no backend, no build process, and no internet connection after initial load. All data is persisted in the browser's `localStorage`.

**Live preview:** https://nullboard.io/preview

### Key Characteristics

- Single-file application (`nullboard.html`) with embedded CSS and JavaScript
- Completely offline capable (works as `file://` or served via HTTP)
- Multi-board support with instant switching
- 50-revision undo/redo history per board
- Drag-and-drop reordering of notes and lists
- Export/import via `.nbx` JSON files
- Optional backup to external HTTP agents
- Modern Notion/Trello-inspired UI with light and dark themes

---

## Tech Stack

| Layer       | Technology                         |
|-------------|------------------------------------|
| Language    | HTML5, CSS3, JavaScript (ES6+)     |
| Libraries   | None (zero dependencies)           |
| Storage     | Browser localStorage               |
| Data format | JSON                               |
| Fonts       | Barlow, IBM Plex Sans, Open Sans, Maven Pro, Segoe UI (WOFF) |
| Build tools | None (no build step required)      |
| License     | 2-clause BSD + Commons Clause      |

---

## Project Structure

```
prs-board/
├── nullboard.html              # Complete application (single file)
├── README.md                   # Project documentation
├── LICENSE                     # License
├── extras/                     # Static assets
│   ├── *.woff                  # Font files (Barlow, IBM Plex, Open Sans, Maven Pro)
│   └── favicon-16.png          # Favicon
└── images/                     # Screenshots and demo GIFs
```

There are no configuration files, no `package.json`, no bundler config. The entire application lives in `nullboard.html`.

---

## Architecture

### Code Organization (within nullboard.html)

The single HTML file is organized into these sections:

1. **`<style>`** - All CSS including theming via CSS custom properties
2. **`<tt>` templates** - HTML templates for boards, lists, and notes (cloned at runtime)
3. **`<script>` blocks** - Multiple script blocks containing:
   - Data model classes (Board, List, Note, BoardMeta, AppConfig)
   - Storage classes (Storage, Storage_Local)
   - Backup classes (BackupStorage, SimpleBackup)
   - UI classes (Drag2, VarAdjust)
   - Core application logic and event handlers

### Class Hierarchy

```
Storage (abstract)
└── Storage_Local          # localStorage-backed persistence

BackupStorage (abstract)
├── SimpleBackup           # HTTP REST backup to external agents
└── GistBackup             # GitHub Gist backup (stores all boards in a single gist)

Drag2                      # Drag-and-drop handler for notes and lists
VarAdjust                  # Mouse-drag UI preference adjustments

Board                      # Board data container
├── List                   # Column within a board
│   └── Note               # Individual task/item
├── BoardMeta              # Metadata (revision history, UI state)
└── AppConfig              # User preferences and app config
```

---

## Data Model

### Board
```
{
  id:       <timestamp-based unique ID>,
  title:    <string>,
  revision: <integer>,
  format:   <blob version>,
  lists:    [ List, ... ]
}
```

### List
```
{
  title:        <string>,
  notes:        [ Note, ... ],
  color:        <string (palette key, e.g. "blue", "red", "purple")>,
  completedMin: <boolean (collapsed state of completed section)>
}
```

### Note
```
{
  text: <string (HTML content)>,
  raw:  <boolean (raw text vs formatted)>,
  min:  <boolean (collapsed state)>,
  done: <boolean (completed state)>
}
```

### AppConfig
Stored in `localStorage` as `nullboard.config`:
- UI preferences: `fontName`, `fontSize`, `lineHeight`, `listWidth`, `theme`
- State: `activeBoard`, `verLast`, `verSeen`
- Backup agent configurations (array of `{ base, auth }` objects)
- `maxUndo`: revision history depth (default: 50)

---

## Data Flow

```
User Action (click, drag, edit)
         |
    DOM Event Handler (vanilla JS)
         |
    Model Update (Board/List/Note objects)
         |
    saveBoard()
         |
    Storage_Local.saveBoard(board)
         |
    +-- Assign new revision number
    +-- Trim old revisions (keep maxUndo)
    +-- Store board data:  localStorage['nullboard.board.{id}.{revision}']
    +-- Store metadata:    localStorage['nullboard.board.{id}.meta']
    +-- Trigger backup (if configured)
         |
    SimpleBackup agents (async, queued via fetch API)
         |
    HTTP PUT to backup agent endpoints
```

### localStorage Schema

| Key | Content |
|-----|---------|
| `nullboard.config` | App configuration (JSON) |
| `nullboard.board.{id}.meta` | Board metadata: title, current revision, history array, backup status |
| `nullboard.board.{id}.{rev}` | Board data snapshot at revision `rev` (JSON) |

### Revision History

Each board maintains a history array of up to 50 revision IDs. When a board is saved:
1. A new revision number is assigned
2. The board data is written under that revision key
3. The history array is appended
4. Revisions beyond `maxUndo` are deleted from localStorage
5. On undo, bypassed (future) revisions are trimmed

---

## Key Features - Implementation Details

### In-Place Editing
- Click any text (board title, list title, note) to edit inline
- Auto-saves on blur
- Escape cancels editing

### Drag and Drop (`Drag2` class)
- 500ms priming delay before drag activates (prevents accidental drags)
- Hold Alt for instant drag
- Notes draggable within and between lists
- Lists draggable to reorder
- Swap detection with CSS transition animations

### Link Recognition
- Detects `http://` and `https://` URLs in note text
- Supports markdown-style titled links: `[title](url)` renders as a clickable title
- All links open in new tabs (`target=_blank`)
- Links are always visible and left-clickable

### Completed Notes
- Notes can be marked complete via the `✓` button in the note popup
- Completed notes display with strikethrough text and reduced opacity
- Completed notes are moved below an auto-generated "Completed (N)" divider
- The divider shows the count of completed notes and can be collapsed to hide them
- Completed notes are read-only; un-complete them to edit again
- Dragging a note across the divider automatically toggles its done state
- The divider is purely dynamic (not saved to data) and only appears when completed notes exist
- The divider's collapsed state persists per list via `List.completedMin`

### Theming & Visual Design
- Notion/Trello-inspired modern UI with clean, rounded aesthetics
- Light and dark themes via CSS class switching
- CSS custom properties (`--var`) for colors and dimensions
- 5 font choices with adjustable size, line height, and list width
- All preferences persisted in AppConfig

#### Design System
| Element | Light | Dark | Radius |
|---------|-------|------|--------|
| Body background | `#f5f5f4` | `#191919` | — |
| Board head | `#fff` with subtle shadow | `#252525` | 10px |
| Lists | `#ebebea` | `#252525` | 10px |
| Notes | `#fff` with soft shadow | `#2f2f2f` | 8px |
| Modals | `#fff` with backdrop blur | — | 12px |
| Accent color | `#2383e2` (Notion blue) | `#fc2` (gold) | — |
| Focus ring | `0 0 0 2px #2383e2` | `0 0 0 2px #d4a72c` | — |

#### Per-List Color Palette
Each list can have its own pastel background color from a curated 11-color palette:

| Name | Light | Dark |
|------|-------|------|
| Polar Water | `#dae9f1` | `#1c2a33` |
| Blue | `#dbeafe` | `#1c2433` |
| Pine | `#d8dce8` | `#1e2030` |
| Sky | `#d2e8f5` | `#1a2733` |
| Foam | `#d5e3e5` | `#1c2828` |
| Lichen | `#d5ddd0` | `#222820` |
| Yellow | `#f2e6d0` | `#2a2618` |
| Sakura | `#e8d5d0` | `#2b201e` |
| Red | `#f2d0d5` | `#2d1c22` |
| Purple | `#e4d5e8` | `#251e33` |
| Lavender | `#dddaf5` | `#1e1c33` |

- New lists are auto-assigned a random color
- Color picker (dots) available in each list's `≡` dropdown menu
- Color is stored in `List.color` and persisted with board data
- "None" option reverts to default list background

#### Typography
- Base font size: 15px (up from original 11px)
- Line-height multiplier: 1.35
- Board heading: 17px
- Menu/ops icons: font-weight 600–700
- Note text: generous padding (8px 12px)

### Export / Import
- Format: `.nbx` files (JSON)
- Supports single board or batch export/import
- Blob version validation on import

---

## Backup Agent Protocol

Nullboard can sync data to external backup agents over HTTP REST:

### Endpoints

| Method | Path | Body | Purpose |
|--------|------|------|---------|
| `PUT` | `/config` | `{ self, conf }` | Save app configuration |
| `PUT` | `/board/{id}` | `{ self, data, meta }` | Save board data + metadata |
| `DELETE` | `/board/{id}` | - | Delete a board |

All requests include `X-Access-Token` header if authentication is configured.

### Default Agent Configuration
```javascript
[
  { base: 'http://127.0.0.1:10001', auth: '' },  // Local agent
  { base: '', auth: '' }                          // Remote agent (unconfigured)
]
```

### Known Backup Agents
- **Nullboard Agent** - Windows native app (default port 10001)
- **nullboard-agent-express** - Node.js/Express port ([GitHub](https://github.com/justinpchang/nullboard-agent-express))
- **nbagent** - Python/Unix agent ([GitHub](https://github.com/luismedel/nbagent))

Backup requests are queued and processed asynchronously. Each agent tracks status: `busy`, `ready`, or `error`.

### GitHub Gist Backup

Nullboard can also back up all boards to a single GitHub Gist via the `GistBackup` class:

- **Authentication:** Requires a GitHub Personal Access Token (PAT) with the `gist` scope
- **Storage model:** One gist contains all boards as separate files:
  - `nullboard-config.json` — app configuration
  - `nullboard-board-{id}.json` — board data (one per board)
  - `nullboard-meta-{id}.json` — board metadata (one per board)
- **Auto-creation:** If no Gist ID is provided, a new secret gist is created automatically on first save
- **Versioning:** GitHub tracks gist revision history automatically
- **Restore:** All boards can be restored from a gist by providing the Gist ID (or full gist URL) and token. Supports restoring onto a new browser/device
- **URL support:** The Gist ID field accepts both raw IDs and full `gist.github.com` URLs

Configuration is managed alongside the existing Local/Remote backup agents in the Auto-backup dialog.

---

## Deployment

No build step is required. Deployment options:

1. **Static file server** - Serve `nullboard.html` and `extras/` from any HTTP server
2. **Local file** - Open `nullboard.html` directly in a browser (`file://` protocol)
3. **Docker** - Community Docker image available at [rsoper/nullboard](https://github.com/rsoper/nullboard)

### Browser Support
- Primary: Firefox, Chrome
- Compatible: Safari, Edge
- Requirements: localStorage, ES6, Flexbox

---

## Version Tracking

| Version | Value | Purpose |
|---------|-------|---------|
| `codeVersion` | 20231105 | Tracks UI/application code changes |
| `blobVersion` | 20190412 | Tracks board data format for migration compatibility |

---

## JavaScript Patterns

- **Zero dependencies** - all DOM manipulation uses vanilla JS (no jQuery or other libraries)
- **ES6 classes** for Storage, BackupStorage, SimpleBackup, and GistBackup
- **Prototypal patterns** for UI classes (Drag2, VarAdjust)
- **Native DOM APIs** - `querySelector`/`querySelectorAll`, `classList`, `addEventListener`
- **Custom `delegate()` helper** for event delegation (replaces jQuery's `.on(event, selector, fn)`)
- **Promise-based animation helpers** - `animateEl()`, `slideUp()`, `slideDown()` using CSS transitions and `requestAnimationFrame`
- **`fetch()` API** for backup agent HTTP communication
- **Template cloning** from `<tt>` elements via `cloneNode(true)` for dynamic UI generation
- **Async queue** pattern for backup request batching
- **Object.assign()** for object composition and config merging
