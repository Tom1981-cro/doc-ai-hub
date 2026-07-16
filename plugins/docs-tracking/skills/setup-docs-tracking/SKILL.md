---
name: setup-docs-tracking
description: Bootstrap the self-maintaining docs-tracking system in the CURRENT repo — generate a source-derived ROUTE_REFERENCE map, seed a canonical SYSTEM_DOCS, and wire the read-before / update-after loop via CLAUDE.md plus the /hey-claude and /sync-route-reference commands. Run once per repo. Stack-agnostic. Use when the user wants "docs tracking" / "self-maintaining docs" set up in a repo.
---

# Setup Docs Tracking

Install the self-maintaining documentation system into the current repository so
the coding agent reads accurate route/system context before each task and keeps it
fresh after. Everything is generated from THIS repo's real source.

Run from the repo root. Read-only except the files this skill writes; never edit
product source. Prefer a blank/"unverified" field over a wrong one.

## Step 0 — Detect the stack (read-only)
Identify from the repo:
- **Frontend routing** (if any): Next.js app/pages router, expo-router (`app/**`
  file routes — scan `.tsx` AND `.jsx`), React Router config, SvelteKit, Rails
  views, etc. Note dynamic/group segments.
- **Backend / API**: framework + where routes live (Express, FastAPI `@router`,
  Rails routes.rb, Django urls.py, Go handlers…) and the URL prefix.
- **Data layer**: SQL tables (schema/migrations/ORM) OR NoSQL collections
  (`db.<collection>`) OR none — this is the "right table/collection" source.
- **Access model**: how auth/roles gate routes (middleware, guards, redirects).

If the repo has none of these (e.g. a library), say so and offer a lighter
SYSTEM_DOCS-only setup.

## Step 1 — Generate ROUTE_REFERENCE.md
Thorough read-only investigation (use Explore/Task subagents for breadth). Write
`ROUTE_REFERENCE.md` at the repo root, three sections, each entry:
`Route / Component (file) / Access / Backend API (method+path) / Data (tables or
collections) / Known issues (blank) / Last audited (blank)`. Also a **Backend API**
section (every route file → endpoints → data) and a **Data** section (deduped
tables/collections → files using them). Start with a "How routing works here"
section and a `_Last generated:_` line. Scan ALL route file extensions the stack
uses (don't miss `.jsx`, `.js`). Preserve any pre-existing Known-issues /
Last-audited values on re-runs.

## Step 2 — Seed SYSTEM_DOCS.md
Write `SYSTEM_DOCS.md`: Architecture overview · Known bugs (by priority — seed
only what you can verify) · Data notes · Security/TODO · **Recent Changes Log**.
Every section ends with a **Last audited:** date. Mark human-context gaps as such.

## Step 3 — Wire CLAUDE.md
If `CLAUDE.md` exists, append the section in **Template A**; else create `CLAUDE.md`
starting with it, adapted to the detected stack.

## Step 4 — Install the commands
Write `.claude/commands/hey-claude.md` (**Template B**) and
`.claude/commands/sync-route-reference.md` (**Template C**), adapting the
stack-specific wording (route file extensions, backend framework, data layer).

## Step 5 — Report + remind to commit
List what was created + totals, and remind the user to git-track it all
(`git add CLAUDE.md SYSTEM_DOCS.md ROUTE_REFERENCE.md .claude/commands/`) — if it's
not in version control it drifts between environments.

---

## Template A — CLAUDE.md "Documentation" section

```markdown
## Documentation — source of truth (read before, update after)

Two git-tracked living docs are the source of truth for how this app is wired:

- **SYSTEM_DOCS.md** — canonical: architecture, known bugs (by priority), data
  notes, security TODOs, and the Recent Changes Log. Human-curated + agent-updated.
- **ROUTE_REFERENCE.md** — generated map: every route → component file → backend
  API → data (tables/collections) → access.

**Every implementation task follows this loop (`/hey-claude` wraps it):**
1. Before writing code — read SYSTEM_DOCS.md (relevant section) + ROUTE_REFERENCE.md
   (the exact route → component file → endpoint → data). Edit the file the
   reference names — not a similarly-named dead one — and query the table/collection
   it names. Note any Known Issues.
2. After the change is verified — update SYSTEM_DOCS.md (Known Issues + Last audited
   + Recent Changes Log); update ROUTE_REFERENCE.md if routes/screens/endpoints/data
   were added/removed/renamed **or rewired**.
3. Only update sections you actually verified. A confidently-wrong doc update is
   worse than a stale one. If code and docs disagree, trust the code and fix the doc.

ROUTE_REFERENCE.md is generated from source, not hand-maintained. Regenerate it with
the committed `/sync-route-reference` command after larger drift.
```

## Template B — .claude/commands/hey-claude.md

```markdown
---
description: Run an implementation task with the docs loop (read docs before, update after).
---

# /hey-claude — implementation task with the docs loop

$ARGUMENTS

## PRE-IMPLEMENTATION: READ DOCS
1. Read SYSTEM_DOCS.md — the section for the area being changed; note Known Issues.
2. Read ROUTE_REFERENCE.md — the exact route, component file, endpoint(s), and
   data tables/collections. Edit the file it names; query the table it names.
3. If docs and code disagree, trust the code — and fix the doc in this task.

## IMPLEMENT
Make the change. Verify it (typecheck / run the affected flow) before touching docs.

## POST-IMPLEMENTATION: UPDATE DOCS
1. Update SYSTEM_DOCS.md: Known Issues for the touched section, its Last audited
   date, and a Recent Changes Log line.
2. Update ROUTE_REFERENCE.md if routes/screens/endpoints/data were added, removed,
   renamed, or rewired. For larger drift run /sync-route-reference.
3. Only update sections you actually verified.
```

## Template C — .claude/commands/sync-route-reference.md

```markdown
---
description: Regenerate ROUTE_REFERENCE.md from the current source.
---

# /sync-route-reference — regenerate the route map

ROUTE_REFERENCE.md is generated by pointing Claude at the source (the
route → API → data mapping needs to read code semantically). This command is that
generator, committed so regeneration is reproducible. Run when the map has drifted.

Read-only investigation; rewrite ROUTE_REFERENCE.md in place (do not edit source):
- **Frontend routes** — every route file (scan ALL extensions the stack uses) →
  component → access → the /api endpoints it calls → the data it touches.
- **Backend API** — every route file → endpoints (method+path→handler) → data.
- **Data** — deduped tables/collections → the files that use each.
Preserve human-added Known-issues / Last-audited values for routes that still
exist. Update the _Last generated:_ line. Report totals.
```
