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
- `PROGRAMS` — the six certificate programs and their courses; each course has a `done` *default* and optional `note`.
- `WEEKS` — the 14-week plan (hardcoded ISO dates in the Jun 29–Oct 2, 2026 window) with per-week `items`.
- `LINKEDIN` — the visibility-milestone checklist.

**State model (the non-obvious part).** Persisted to `localStorage` under key `aicert_v1` as `{courses, logs, linkedin, notify}`. Course completion is stored *sparsely as overrides*, not as full state:
- `cKey(programId, courseIndex)` → `"programId::index"` is the storage key.
- `isDone(p, i, default)` returns the override in `state.courses` if present, else the `done` default from `PROGRAMS`.
- So flipping a checkbox writes one override key; untouched courses fall back to their `PROGRAMS` default (e.g. the pre-completed Google course).

**Render pipeline.** `renderAll()` calls every `render*` function. Any state mutation pattern is: mutate `state` → `save()` → `renderAll()`. Keep that order when adding features.

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
