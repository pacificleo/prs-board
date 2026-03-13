# SimpleBoard

SimpleBoard is a minimalist take on a kanban board / a task list manager, designed to be compact, readable and quick in use.

https://github.com/pacificleo/prs-board

![SimpleBoard](images/simpleboard-example-alt.png)

## Dead simple

* Single-page web app - just one HTML file and a webfont pack.
* Can be used completely offline. In fact, it's written exactly with this use in mind.

## Locally stored

* All data is stored locally, for now using [localStorage](https://developer.mozilla.org/en/docs/Web/API/Window/localStorage).
* The data can be exported to- or imported from a plain text file in a simple JSON format.
* The data can also be automatically backed up to a local disk with the help of:
  * [Nullboard Agent Express Port](https://github.com/justinpchang/nullboard-agent-express) - an express.js-based portable app
  * [nbagent](https://github.com/luismedel/nbagent) - a version for Unix systems, in Python
* The data can also be backed up to a **GitHub Gist** - all boards are stored as files in a single secret gist, with automatic versioning via GitHub's gist revision history. Restore onto any device by providing the Gist ID and a GitHub token.

## Beta

Still very much in beta. Caveat emptor and all that.

## UI & UX

The whole thing is largely about making it convenient to use.

Everything is editable in place, all changes are saved automatically and last 50 revisions are kept for undo/redo:

![In-place editing](images/simpleboard-inplace-editing.gif)

New notes can be quickly added directly where they are needed, e.g. before or after existing notes:

![Ctrl-add note](images/simpleboard-ctrl-add-note.gif)

Notes can also be dragged around, including to and from other lists:

![Drag-n-drop](images/simpleboard-drag-n-drop.gif)

Nearly all controls are hidden by default to reduce visual clutter to its minimum:

![Hidden controls](images/simpleboard-hidden-controls.gif)

Longer notes can be collapsed to show just the first line, for even more compact view of the board:

![Collapsed notes](images/simpleboard-collapsed-notes.gif)

The default font is [Barlow](https://tribby.com/fonts/barlow/) - it's both narrow *and* still very legible. Absolutely fantastic design!

![Barlow speciment](images/barlow-specimen.png)

Notes can also be set to look a bit different. This is useful for partitioning lists into sections:

![Raw notes](images/simpleboard-raw-notes.gif)

Notes can be marked as complete via the ✓ icon. Completed notes get strikethrough text and are moved below a "Completed (N)" divider. The divider can be collapsed to hide completed notes. Dragging notes across the divider toggles their completed state.

Links starting with https:// and http:// are recognized and open in new tabs. Markdown-style titled links are also supported — write `[My Doc](https://example.com/long-url)` to display a clean clickable "My Doc" link instead of a long URL.

![Links on hover](images/simpleboard-links-on-hover.gif)

Pressing CapsLock will highlight all links and make them left-clickable.

![Links reveal](images/simpleboard-links-reveal.gif)

Lists can be moved around as well, though not as flashy as notes:

![List swapping](images/simpleboard-list-swap.gif)

The font can be changed; its size and line height can be adjusted:

![Theme and zoom](images/simpleboard-ui-preferences.gif)

The color theme can be inverted:

![Dark theme](images/simpleboard-dark-theme.gif)

Each list can have its own background color from a curated 11-color pastel palette, selectable via the list menu.

Also:

* Support for multiple boards with near-instant switching
* Undo/redo for 50 revisions per board (configurable in the code)
* Keyboard shortcuts, including Tab'ing through notes

## Caveats

* Written for desktop and keyboard/mouse use
* Essentially untested on mobile devices and against tap/touch input
* Works in Firefox, tested in Chrome, should work in Safari and may work in Edge (or what it's called now)
* Uses localStorage for storing boards/lists/notes, so be careful around [clearing your cache](https://stackoverflow.com/questions/9948284/how-persistent-is-localstorage)

You spot a bug, file an issue.

## Background

SimpleBoard is something that handles ToDo lists in the way that works really well. For *me* that is.

Tried a lot of options, some were almost *it*, but none was 100%.

**Trello** wasn't bad, but never was comfortable with the idea of storing my data in cloud without any actual need.

**Wekan** looked promising, but ultimately too heavy and had no offline usage support or a local storage option.

**Things** was beautiful, but not the right tool for the job.

**Inkscape** - I kid you not - with a laundry list of text items was actually OK, but didn't scale well.

Ditto for the plain **text files**.

Pieces of **paper** were almost there, but rearranging items can be quite a hassle.

So finally got annoyed enough to sit down and write exactly what I wanted.

And, voilà, SimpleBoard came out => https://github.com/pacificleo/prs-board

## License

The [2-clause BSD license](https://opensource.org/licenses/BSD-2-Clause/) with the [Commons Clause](https://commonsclause.com/).

That is, you can use, change and re-distribute it for as long as you don't try and sell it.
