# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Overview

A static HTML/CSS/JS tutorial site (JavaScript-Practices): one page per JavaScript topic, covering core JS through DOM/BOM, AJAX, JSON, jQuery comparison, and charting libraries (Canvas, Plotly, Chart.js, D3, Google Charts). No build step, no package manager, no tests — pages are plain HTML opened directly or served as static files.

## Architecture

**Every content page is a full standalone HTML document following one fixed template** (see any `NN_topic.html`, e.g. `1_js_intro.html`): meta/head boilerplate, a top navbar, a left sidebar nav (`<nav class="sidebar">`), a `<header>`, a `<section class="content">` with the actual lesson content, and a shared `js/main.js` include. There is no templating engine or includes mechanism — the entire sidebar markup is duplicated verbatim in every single HTML file.

**Sidebar duplication is the central maintenance hazard.** The sidebar (`<!-- Left-Side-Navigation-Bar Start-->` ... `<!-- Left-Side-Navigation-Bar End-->`) is the same ~240-line block copy-pasted into `index.html` and all ~150 topic pages. When the sidebar structure or links change, **every page must be updated**, not just the one being edited. On a page, its own entry in the sidebar should carry `class="nav_link active"`; no other page should match. `index.html`, `rough.html`, and `temp.html` are not topic pages and have no self-link, so none of their sidebar entries get `active`.

Given the file count, bulk sidebar edits are best done with a script (find the marker block, substitute, re-inject `active` on the matching `href`) rather than 150 manual edits — see git history around 2026-10-10 for a worked example.

## File naming / numbering

Files are numbered to match the sidebar's teaching order (`1_js_intro.html`, `2_howto.html`, ... `147_Google_Chart_JS.html`), but the numbering has gaps and one suffix exception baked in — do not assume strict sequential numbering maps 1:1 to sidebar position:

- `26a_array_search.html` — inserted between `26_array_methods.html` and `27_array_sort.html`. Added via an `a`-suffix rather than renumbering everything after it, since 26–29 were already taken.
- `42_loop_for_of.html`, `76_object_sets.html`, `113_bom_timing_events.html`, `129_json_syntax.html` — exist on disk but have no historical sidebar entry of their own; they were later added as their own sidebar items in later menu sections (JavaScript Basic / Objects / BOM / JSON respectively). Confirm the sidebar actually links them before assuming they're reachable.
- `123_XMLHttpRequest_Object.html` and `124_XMLHttpRequest.html` are numbered in one order but linked from the sidebar in the opposite order (the sidebar lists "XMLHttpRequest" before "XMLHttpRequest Object", matching topic flow, not file number).

When adding a new topic page or resolving a missing link, **check for a free/matching filename before creating one** — grep the sidebar for the exact title text in `index.html`, and verify the target filename doesn't collide with an existing file's topic.

## Content template (per page)

The `<section class="content">` body uses a fixed placeholder skeleton: `<h3>SectionTitle</h3>`, a description paragraph, an optional list/table of code examples, and a closing note. Many pages still have literal `SectionTitle`/`SectionDescription`/`code`/`description` placeholder text — check before assuming a page's content is finished.

## Verifying link integrity

After any sidebar or file-naming change, validate from the project root:

```bash
# No leftover placeholder links
grep -rl 'href="xxx"' *.html

# Every href resolves to a real file
for f in *.html; do grep -oP 'href="\K[^"]+(?=")' "$f" | grep -v '^https\?://\|^#'; done | sort -u | while read l; do [ -f "$l" ] || echo "MISSING: $l"; done

# Exactly one active link per self-referencing page
for f in $(grep -l 'nav_link active' *.html); do c=$(grep -o 'nav_link active' "$f" | wc -l); [ "$c" != "1" ] && echo "$f: $c actives"; done
```

## Shared assets

- `js/main.js` — sidebar collapse/expand, dark-mode toggle, submenu show/hide. Shared across all pages; do not duplicate its logic inline.
- `css/main.css` — primary stylesheet. `css/style.css` is currently empty; `css/mystyle.css` is a small supplementary file.
- Boxicons is pulled from a CDN (`unpkg.com/boxicons`) for nav icons.
