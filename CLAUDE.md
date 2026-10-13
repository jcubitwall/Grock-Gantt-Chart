# Cubit Tasker — project notes for Claude

Internal Gantt / task-tracker for Cubit Home Inc. (Nova Scotia modular homes). Owner: Johnny Wall (Chairman).
Single file: `index.html` (vanilla JS, `<script type="module">`, Supabase JS client). Hosted on GitHub Pages from `main`.

## Working rules
- Edit `index.html` directly in this repo, then commit + push to `main` (that deploys). Johnny has authorised Claude to push updates.
- ALWAYS bump the footer build stamp: `Cubit Tasker · Build YYYY-MM-DD.N` (currently 2026-09-06 series). Johnny verifies deploys by reading it.
- Syntax-check the module script with `node --check` after extracting it, before pushing.
- Commit messages: start with the build number + plain-English summary.
- Supabase changes cannot be made from here. If a change needs SQL, give Johnny the SQL to run in the Supabase SQL editor and say so clearly.
- Johnny often works from his phone; keep instructions short and plain.

## Backend (Supabase)
Tables: `profiles` (roles: pending/member/editor/admin), `projects` (has `doc_url`), `project_board` (rows jsonb, start_date, skip_weekends…), `project_config` (units, templates, roster, `materials_catalog`), `messages`. Realtime channels + RLS. Auth users are separate from `profiles` (deleting a profile does not free the email; delete in Supabase Dashboard → Authentication → Users).

## Key code concepts
- Sync: `save()`/`doSave()`, `mergeIncoming()`, `hasUnsyncedEdit`, recency guard on realtime payloads (`updated_at`), `boardData`/`configData` updated after each successful write (fixes lane bounce-back), `refreshBoardSnapshot()` on visibilitychange.
- Lock is local-only: `sessionLocked` (never persisted); unlocking affects only that user.
- Lanes/drag: `packLanes`, `laneOverride`, `onBarMouseDown`/`onDragMove`/`onDragEnd`, `canDrag` (touch on a pill scrolls when locked/non-editor), `cascadeSchedule` (linked cascade moves every pill starting same day column or later).
- Progress: `computeProgress()` / `progressForRows(rows)` — checklist items count individually, else 1 per row via `r.done`; headers/todo rows excluded.
- Projects list: `projectsCache`, `projectBoardsCache`, `renderProjectsList()` (completion pill: blue track, green fill, % label), `computeTaskNameStats`, `buildPieSvg`, `openCategoryPieModal` (opened from gear menu → "Task Time Pie Chart"; uses `.panel-overlay`, z-index 300).
- Messaging/notes: `messages` table, inbox panel, unread badge; admins see all.
- Materials: catalog in `state.materialsCatalog`; per-row `materials[]` + `noMaterialsUsed`; `materialsLoggedFor(row)` gates completion; CSV report with summary + detail; bulk import parser; searchable combo box.
- Gear menu (admin only, `#adminMenuList`): Units & Generate, Task Templates, Materials Catalog, Task Time Pie Chart, Materials Report (CSV), Project Layouts, Users, Settings, Print/PDF.

## Related repos (Johnny's GitHub: jcubitwall)
- Cubit-Site: hub/landing site (Production/Accounting/Gallery pages) — on hold, not deployed.
- Others (Cubit-Task-Gantt, Cubit-Gantt, Cubit-Management-App, cubit-flow) are older versions; this repo is the live Tasker.

## Parked ideas
Department-scoped access matrix for the hub; Edge Function for full user deletion; push notifications for home-screen badge; full-year stats; multi-tenancy; Cloudflare custom domain + Cloudflare Access.

## Where we left off
- 2026-10-13: Build 2026-09-06.18 pushed — completion % pill on Projects list; pie chart moved into gear menu. No Supabase migration needed.
- Pending: Johnny wants to rename this repo to something like "cubittasker" (note: renaming changes the GitHub Pages URL and breaks bookmarks/home-screen icons; Supabase Auth Site URL / redirect URLs may need updating).
