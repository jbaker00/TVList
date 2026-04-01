# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Running the App

No build step required. Open `index.html` directly in a browser, or serve it with any static HTTP server:

```bash
python -m http.server
```

## Architecture

The entire application is a **single file** (`index.html`) with embedded CSS and JavaScript — no framework, no build toolchain, no dependencies.

### Structure within index.html

- **CSS** (lines ~7–573): Dark-themed design system with CSS variables. Accent colors: purple `#6c63ff`, pink `#ff6b9d`. Mobile breakpoint at 600px.
- **HTML** (lines ~575–642): Skeleton markup for header/search, tabs, card grids, modal, and toast.
- **JavaScript** (lines ~644–1016): All app logic.

### State

Two arrays persisted to `localStorage` (`tvtracker_watched`, `tvtracker_watchlist`):

```javascript
watched[]    // shows the user has seen
watchlist[]  // shows the user wants to watch
```

Each show entry: `{ id, name, image, year, genres, rating, network, status, addedAt }`.

### Data Flow

1. User types in the search box → debounced (350ms) fetch to **TVMaze API** (`https://api.tvmaze.com/search/shows?q=...`)
2. Selecting a result fetches full details (`/shows/{id}`) and opens the modal
3. Modal actions (Add to Watched / Add to Watchlist / Move / Remove) mutate the state arrays and write to `localStorage`
4. Every mutation calls `renderLists()`, which re-renders both card grids from state

### Key Functions

| Function | Purpose |
|---|---|
| `renderLists()` | Re-renders both tabs from state; call after any state mutation |
| `cardHTML(show, list)` | Returns HTML string for a show card |
| `addShow(list)` | Adds `currentShow` to the given list |
| `moveShow(fromList, toList, id)` | Moves a show between lists |
| `removeShow(id)` | Removes from both lists |
| `sorted(arr)` | Returns sorted copy of a show array (sort order from `#sort-select`) |
| `esc(str)` | HTML-escapes a string — use this on all external data before inserting into innerHTML |

### XSS Protection

All data from TVMaze (show names, network names, etc.) must be passed through `esc()` before being placed in `innerHTML`. Never bypass this.
