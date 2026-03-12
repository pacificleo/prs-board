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
- Light and dark themes with customizable fonts

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
└── SimpleBackup           # HTTP REST backup to external agents

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
  title: <string>,
  notes: [ Note, ... ]
}
```

### Note
```
{
  text: <string (HTML content)>,
  raw:  <boolean (raw text vs formatted)>,
  min:  <boolean (collapsed state)>
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
- Hold CapsLock or Ctrl to reveal clickable links
- Right-click context menu for link operations

### Theming
- Light and dark themes via CSS class switching
- CSS custom properties (`--var`) for colors and dimensions
- 5 font choices with adjustable size, line height, and list width
- All preferences persisted in AppConfig

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
- **ES6 classes** for Storage, BackupStorage, and SimpleBackup
- **Prototypal patterns** for UI classes (Drag2, VarAdjust)
- **Native DOM APIs** - `querySelector`/`querySelectorAll`, `classList`, `addEventListener`
- **Custom `delegate()` helper** for event delegation (replaces jQuery's `.on(event, selector, fn)`)
- **Promise-based animation helpers** - `animateEl()`, `slideUp()`, `slideDown()` using CSS transitions and `requestAnimationFrame`
- **`fetch()` API** for backup agent HTTP communication
- **Template cloning** from `<tt>` elements via `cloneNode(true)` for dynamic UI generation
- **Async queue** pattern for backup request batching
- **Object.assign()** for object composition and config merging
