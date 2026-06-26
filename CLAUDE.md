# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## What this is

A single-file, zero-build, zero-dependency static web app: a 14-week AI/AI-security
certification study tracker. Everything lives in `index.html` (HTML + CSS + vanilla JS).
No package.json, no framework, no bundler, no tests.

## Commands

- **Run:** open `index.html` in any browser. For a served copy: `python3 -m http.server` then visit `localhost:8000`.
- **Deploy:** GitHub Pages — Settings → Pages → Deploy from branch → `main` / root.

There is no build, lint, or test step. Edits to `index.html` are live on reload.

## Architecture

**Three data arrays at the top of the `<script>` drive the entire UI** — change content here, not in the render functions:
- `PROGRAMS` — 6 programs → 21 courses → 70 Coursera `modules`. Each course has a `modules` array (the real Coursera module titles) and a `done` *default* (only Google's "Beyond the Chatbot" is `true`).
- `WEEKS` — the 14-week plan (hardcoded ISO dates in the Jun 29–Oct 2, 2026 window) with per-week `items`.
- `LINKEDIN` — the visibility-milestone checklist.

**State model (the non-obvious part).** Persisted to `localStorage` under key `aicert_v2` as `{modules, logs, linkedin, notify}`. **The module is the single source of truth; course and certificate completion are *derived*, never stored:**
- `mKey(programId, courseIndex, moduleIndex)` → `"prog::ci::mi"` is the only completion key written.
- `modDone(p, ci, mi, default)` returns the override in `state.modules` if present, else the course's `done` default. Untouched modules fall back to the default (so Google's pre-completed course shows done).
- `courseStats()` rolls modules up to a course (`full` = all modules done, `part` = some); `progStats()` rolls courses up to a program; `totals()` aggregates everything. Counts and the donut are recomputed from these, not stored.
- Ticking a course checkbox (`checkCourse`) bulk-sets all its module keys. `load()` migrates old `aicert_v1` course-level overrides into module keys.

**Tracker UI.** The homepage `#programs` tree is the only place to mark progress (there is no separate Courses tab). Accordion open/closed state lives in in-memory `Set`s (`openP`, `openC`), reapplied on every re-render. Expand/collapse toggles the `.open` class directly (smooth `grid-template-rows` 0fr↔1fr animation); checking a box does a full `afterCheck()` re-render (open state preserved via the Sets). First render only plays the staggered entrance via the `.intro` class.

**Render pipeline.** `renderAll()` calls every `render*` function. Any state mutation pattern is: mutate `state` → `save()` → re-render (`afterCheck()` for the tracker, or `renderAll()`). Keep that order when adding features. All user strings are run through `esc()` before going into `innerHTML`.

**Design system.** Editorial light theme (see `PRODUCT.md`): OKLCH tokens, one teal accent, ink + paper, Fraunces (display) / Inter (UI) / JetBrains Mono (data). One deliberate dark "masthead" moment is the hero panel. Full `prefers-reduced-motion` alternative.

**Date-driven logic** (all relative to "today"):
- `currentWeek()` finds which `WEEKS` entry contains today and highlights it; returns `-2` if the plan hasn't started yet.
- `weekdaysStreak()` counts consecutive logged Mon–Fri days (weekends skipped, not streak-breaking).
- `weekHours()` sums `state.logs` since the most recent Monday.
- Because `WEEKS` dates are hardcoded to 2026, "current week" / streak behavior only makes sense within that window.

**Reminders.** In-page `Notification` API nudges only fire while the tab is open (`maybeRemind()` on load, weekday + not-yet-logged). Closed-page nudges are handled separately by a scheduled reminder in the Claude desktop app (see README) — not by this code.

## Keep in sync

`STUDY-PLAN.md` is the human-readable mirror of the `WEEKS` array in `index.html`. When you change the plan, update both.

## Do not commit

`ruvector.db` is a local ruflo plugin database (gitignored). Never push it.
