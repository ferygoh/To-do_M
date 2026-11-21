# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Project

A single-file to-do tracker in plain HTML/CSS/JS (no frameworks, no build step, no tests, no package manager). Each task has: item, priority, deadline, requested by (plus optional notes and a done flag). To run it, open `index.html` in a browser.

## Files

- `index.html` is the whole app: the CSS, markup and JS are all inline. Edit it directly.
- `mockup.html` is a **generated** file: two iPhone 16 Pro Max frames (light and dark), each embedding a modified copy of `index.html` in an iframe `srcdoc`. Do not edit it by hand. It goes stale whenever `index.html` changes.
  - It is built by a PowerShell script (`build-mockup.ps1`) and an HTML template (`mockup-template.html`). Both were kept in a session scratchpad and are NOT in this repo, so they may need to be recreated.
  - The build reads `index.html`, swaps the line `var tasks = JSON.parse(load("tasks") || "[]");` for sample tasks (this exact line must stay in `index.html`, or the build throws), forces the theme, escapes the HTML for `srcdoc`, and writes `mockup.html`.

## Architecture notes (index.html)

- **State and storage**: a `tasks` array persisted in `localStorage` under the key `tasks` (theme under `theme`). All storage access goes through the try/catch helpers `load` and `store`, because storage can be blocked (for example in `data:` URL previews). Keep it that way.
- **Task shape**: `{id, item, priority, deadline, requestedBy, notes, done}`. `id` comes from `newId`. Older saved tasks without an `id` are backfilled on load.
- **Rendering**: `render()` rebuilds the table from `tasks`. Sorting (done tasks always last), search, priority and requester filters, and hide-done are all read from the control values at render time, so call `render()` after changing any of them.
- **Dates**: use the local-date helpers `today()` and `daysUntil()`. Do not use `toISOString()` for dates, because it gives UTC and is wrong around midnight.
- **Theming**: CSS custom properties on `:root`, overridden under `[data-theme="dark"]`. A script in `<head>` sets `data-theme` before paint (saved choice, else `prefers-color-scheme`).
- **Mobile layout**: under `@media (max-width: 640px)` the table turns into CSS-grid cards. Cell labels come from `data-label` attributes through `::before`. Inputs stay at 16px so iOS Safari does not zoom, and the page uses `viewport-fit=cover` with safe-area insets.
- **Undo**: delete and "clear completed" show a toast with an Undo button.
