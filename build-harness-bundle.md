---
name: build-harness
description: >
  Bootstraps a personal agent harness: global ~/.claude structure (agents,
  commands, hooks, rules, skills), a second-brain command center, per-project
  memory (.claude/memory/), and SOUL.md/BUSINESS.md identity files. Detects
  what already exists and only fills gaps — safe to re-run to update or
  extend an existing harness. Also handles optional remote destination setup
  (GitHub, cloud-synced folder, or Notion for second-brain content). Trigger
  when the user says "build my harness", "set up my agent harness",
  "bootstrap my Claude setup", "update my harness", or "/build-harness".
allowed-tools: Read, Write, Glob, Grep, Bash, AskUserQuestion
---

# /build-harness — Personal Agent Harness Bootstrapper

Walks a user from zero (or from a partial setup) to a complete, personalized
agent harness: global `~/.claude` structure, a second-brain command center,
per-project memory, identity files (SOUL.md/BUSINESS.md), and — optionally —
a wired remote destination. Re-runnable: always detects first, only fills gaps.

## Role Boundaries

- IN SCOPE: detecting existing harness state, interviewing the user, scaffolding
  folders/files from templates, wiring at most one remote destination if asked,
  writing SOUL.md/BUSINESS.md from session answers, reporting personalized next steps.
- OUT OF SCOPE: installing a vector-DB/MCP memory backend, building bidirectional
  cloud sync, writing actual project code, judging the *content* of the user's
  business (only reflecting back what they said).
- If the user's answers are ambiguous or a target already exists, stop and ask —
  never guess a folder name, a cloud path, or overwrite silently.

## Progressive Disclosure

1. Keep this file as the phase-by-phase procedure.
2. Load `references/second-brain-schema.md` only during Phase 1(b)/Phase 3.
3. Load `references/agent-graph-patterns.md` only during Phase 1(c)/Phase 2 (writing AGENTS.md).
4. Load `references/memory-architecture.md` only during Phase 1(d)/Phase 4 (writing `.claude/memory/`).
5. Load `references/remote-sync-options.md` only during Phase 1(e)/Phase 5 (destination wiring).
6. Do not load any reference file speculatively — only when its phase is active.

## Feedback

Read `feedback.log` (in this skill's own directory) at the start of every run, if
present. Append general, future-applicable corrections or preferences the user
gives about how the wizard should behave — not one-off answers about a single run.

## Procedure

**Phase 0 — Detect.**
Glob/check for `~/.claude/{agents,commands,hooks,rules,skills}`, any existing
`SOUL.md`/`BUSINESS.md`, and the current project's `.claude/` folder. Ask the
user where their command-center / second-brain lives if not obvious — never
assume a name like "ops," that is one user's naming, not a default. Report
what's found vs. missing before touching anything. If everything already
exists, say so and ask whether they want to extend/update rather than restart
the full wizard.

**Phase 1 — Interview.**
Use `AskUserQuestion` for:
- (a) command-center location/name (where the global harness root should live).
- (b) second-brain taxonomy — offer the default shape from
  `references/second-brain-schema.md`, allow edits.
- (c) which agent roles to seed — offer the default role set from
  `references/agent-graph-patterns.md` as recommended, allow pruning/extending.
- (d) whether to wire per-project memory now for the current project.
- (e) where the harness should live — **Local only** (recommended) / **Local +
  GitHub repo** / **Local + a cloud-synced folder** (Google Drive, Dropbox,
  etc.) / **Local + Notion** (second-brain content only). Exactly one
  destination, not several — see `references/remote-sync-options.md`. Tell the
  user plainly that Obsidian is not a separate destination: it is a viewer
  that works automatically once the second-brain folder exists on disk,
  wherever that is.

**Phase 2 — Scaffold global harness.**
Create only missing folders/files under the confirmed command-center root
(`agents/`, `commands/`, `hooks/`, `rules/`, `skills/`, `plugins/`). Write
`AGENTS.md` from `assets/templates/AGENTS.md.template` using the chosen roles,
encoding explicit handoff edges and validation gates (see
`references/agent-graph-patterns.md`) rather than a flat list. Never overwrite
an existing file — if a target exists, show a short diff-style summary and ask
per-file whether to skip or replace.

**Phase 3 — Scaffold second brain.**
Create the agreed folder taxonomy at the chosen location (idempotent — skip
folders that already exist), with a minimal `index.md` dashboard stub linking
to each top-level folder.

**Phase 4 — Per-project memory.**
If confirmed in Phase 1(d), create `.claude/memory/` in the current project
mirroring the shape in `references/memory-architecture.md`: a `MEMORY.md`
index (pointer-only, one line per entry, kept under 200 lines) plus one typed
entry file per topic (`type: user | feedback | project | reference |
procedural`), cross-linked with `[[name]]`, and referencing the second
brain's daily/weekly notes and priorities for this project. If the project
has a `CLAUDE.md`, add a one-line pointer telling it to read
`.claude/memory/MEMORY.md` at session start; otherwise offer to create a
minimal `CLAUDE.md` from `assets/templates/CLAUDE.md.template`.

**Phase 5 — Wire the chosen destination (only if not Local-only).**
Handle strictly per `references/remote-sync-options.md`:
- **GitHub**: ask explicit confirmation before any `git init` / remote add /
  push — this is a visible-to-others action. Add a `.gitignore` that excludes
  `.env*` and other secret-shaped files before the first commit. Never
  auto-push, never force-push.
- **Cloud-synced folder**: ask the user for their actual local sync-folder
  path — never guess or auto-detect one. If the path doesn't exist, stop and
  say so rather than creating it blind. Document the OS-appropriate step
  (symlink or direct placement) rather than silently moving files.
- **Notion**: only offered for second-brain *content*, never for the
  `~/.claude` harness itself. Requires a live Notion MCP connection; if none
  is found, explain what's needed and skip — never claim it worked.

State plainly that this is one-time setup, not continuous bidirectional sync.
Reconciling a diverged remote copy later is a manual, human decision, not
something this skill automates.

**Phase 6 — SOUL.md + BUSINESS.md.**
Always last of the content-writing phases, and only if missing. Ask targeted
questions (voice/tone/ethics/identity for SOUL; active projects/focus/goals
for BUSINESS) and write from `assets/templates/SOUL.md.template` and
`assets/templates/BUSINESS.md.template`. Content comes only from this
session's answers — never pre-filled with example or placeholder data.

**Phase 7 — Report + personalized next steps.**
Summarize what was created vs. skipped vs. already-present. Confirm
`/build-harness` is safe to re-run later to extend or update. Then, grounded
specifically in what the user said in Phase 6 (their stated objective, focus
area, audience) — not generic filler — give 2-3 concrete "what to do next"
suggestions: e.g. which second-brain folder to populate first, which seeded
agent role to exercise first, what the first `procedural` memory entry should
probably capture. If the Phase 6 answers are too sparse to support a specific
suggestion, say so explicitly rather than inventing one.

## Guardrails

1. Never overwrite an existing file without explicit per-file confirmation.
2. Never assume folder names/locations (command center, second-brain root) —
   always confirm in Phase 1.
3. Detection (Phase 0) always runs before any write, on every invocation.
4. Idempotent by design — safe to re-run; only fills gaps, never duplicates
   folders, files, or memory entries.
5. Templates ship with no personal/business content baked in — SOUL.md,
   BUSINESS.md, rules, and memory entries are generated purely from the
   current session's answers.
6. Default is local filesystem scaffolding only. If (and only if) the user
   picks a remote destination in Phase 1(e), confirm explicitly before any
   push — never auto-push, never force-push, never commit `.env`/secrets.
7. Never fabricate a cloud path (Drive/Dropbox sync folder, Notion workspace)
   — ask for it or detect a live MCP connection; if neither is available,
   tell the user and skip that destination rather than pretending it worked.
8. If asked to run against a directory that already has a full harness, this
   skill only offers to add genuinely missing pieces — it never re-interviews
   or rewrites what is already there without being asked to update it.

## Output Format

End every run with a short report: a bulleted "Created" / "Skipped (already
present)" / "Needs your input" list, followed by the Phase 7 personalized
next-steps section.


---

# FILE: references/agent-graph-patterns.md

````markdown
# Agent Graph Patterns (default role set)

Offer this as the recommended starting set during Phase 1(c). The user can
prune or extend it. When writing `AGENTS.md` in Phase 2, encode explicit
handoff edges and validation gates — not a flat list of roles with no
relationship to each other.

## Default roles

- **orchestrator** — entry point for any non-trivial build or task request.
  Breaks work into steps, gets user approval on the plan, delegates to
  builder roles, then routes to the validator role before declaring done.
- **researcher** — gathers external or internal information the orchestrator
  or builder needs before acting. Read-only; never writes production output.
- **builder** (name per stack, e.g. "fullstack") — does the actual
  implementation work: writes and edits files, runs code. Only invoked by
  the orchestrator or directly by the user for small isolated changes.
- **reviewer / validator** — checks builder output before it's marked done:
  runs tests, re-reads the diff, verifies the change actually does what was
  asked. Never the same role that did the building.
- **logger** (optional) — records what ran and what changed, for
  observability across sessions. Cheap model, minimal tool access.

## Handoff edges (encode these explicitly in AGENTS.md)

```
user request
   -> orchestrator (plan + get approval)
        -> researcher   (if external/unknown info needed first)
        -> builder       (implements)
        -> reviewer       (validation gate — must pass before "done")
        -> logger         (optional, records the run)
```

Rules to encode:
- The orchestrator is the only role allowed to invoke the builder for
  multi-step work — this keeps a single place where scope is agreed before
  code gets written.
- The reviewer is a gate, not a suggestion: builder output is not "done"
  until the reviewer has run against it. Don't let the builder self-certify.
- Only add roles the user actually described a need for — do not invent a
  role (e.g. a "designer" agent) with no matching request.

````


---

# FILE: references/memory-architecture.md

````markdown
# Per-Project Memory Architecture

Used in Phase 4 when the user confirms wiring `.claude/memory/` for the
current project.

## Shape

```
.claude/memory/
  MEMORY.md               # index only — one line per entry, <200 lines total
  user_<topic>.md          # type: user
  feedback_<topic>.md      # type: feedback
  project_<topic>.md       # type: project
  reference_<topic>.md     # type: reference
  procedural_<topic>.md    # type: procedural
```

## Entry file format

```markdown
---
name: <short-kebab-case-slug>
description: <one-line summary, specific enough to judge future relevance>
metadata:
  type: user | feedback | project | reference | procedural
---

<content — for feedback/project: lead with the rule/fact, then **Why:** and
**How to apply:** lines. Cross-link related entries with [[name]].>
```

## MEMORY.md format

One line per entry, pointer-only, no content duplicated from the entry file:

```markdown
# Memory Index

- [Title](file.md) — one-line hook
```

Keep this file under 200 lines total — if it's approaching that, that's a
signal to consolidate or archive stale entries, not to keep appending.

## The `procedural` type

This project-memory type is distinct from the four described in the global
CLAUDE.md memory instructions (user/feedback/project/reference). Use it for
repeatable how-to knowledge specific to this project's workflows — e.g. "how
to deploy this app," "how the test suite is invoked," "the sequence for
cutting a release" — things a new session would otherwise have to rediscover
by reading code or asking the user again.

## Wiring into the project

- If a `CLAUDE.md` already exists at the project root, add one line near the
  top pointing to `.claude/memory/MEMORY.md` so it's read at session start.
  Don't restructure the rest of the file.
- If no `CLAUDE.md` exists, offer to create a minimal one from
  `assets/templates/CLAUDE.md.template` — ask first, don't assume the
  project wants one.
- Reference the second brain's daily/weekly notes and priorities for this
  project where relevant, rather than duplicating that content into project
  memory.

````


---

# FILE: references/remote-sync-options.md

````markdown
# Remote Destination Options

Used in Phase 1(e) and Phase 5. Exactly one destination may be chosen, not
several. Default is Local only.

## Local only (recommended default)

No remote wiring. Everything lives on disk under the confirmed command-center
root and second-brain root. This is the safe default — recommend it unless
the user has a specific reason to want a remote copy.

## Local + GitHub repo

- Confirm explicitly before `git init`, `remote add`, or any push — this
  makes content visible to others (even a private repo has collaborators
  with access, and mistakes are harder to fully retract).
- Add a `.gitignore` excluding `.env*` and other secret-shaped files before
  the first commit.
- Never auto-push. Never force-push. If the user wants continuous sync,
  explain this is one-time setup — they push manually going forward, or ask
  again each time.

## Local + cloud-synced folder (Google Drive, Dropbox, iCloud, etc.)

- Ask the user for their actual local sync-folder path. Never guess a
  default (e.g. `~/Google Drive`) — sync clients vary by install and
  account.
- If the path doesn't exist, stop and say so. Do not create it blind — that
  risks writing into a folder the sync client isn't actually watching.
- Document the OS-appropriate way to place the harness/second-brain there
  (symlink vs. direct placement) rather than silently moving files — moving
  files out from under an existing setup can break relative paths elsewhere.

## Local + Notion (second-brain content only)

- Never offered for the `~/.claude` harness itself — Notion is not a
  substitute for the agent configuration layer, only for human-readable
  second-brain content.
- Requires a live Notion MCP connection. If none is found, explain what
  connecting one requires and skip this destination — never claim it
  succeeded when it didn't.

## Obsidian is not a destination

Tell the user plainly: Obsidian is a viewer/editor that works automatically
once the second-brain folder exists as plain Markdown on disk, regardless of
which of the above options (or Local only) was chosen. It does not need its
own setup step here.

## General rule

State plainly, whichever option is picked, that this is one-time setup, not
continuous bidirectional sync. If a remote copy and the local copy diverge
later, reconciling them is a manual human decision — this skill does not
automate that.

````


---

# FILE: references/second-brain-schema.md

````markdown
# Second-Brain Schema (default taxonomy)

Offer this as the default shape during Phase 1(b); let the user rename,
prune, or add top-level folders before scaffolding in Phase 3.

```
<second-brain-root>/
  index.md              # dashboard stub, links to each folder below
  daily/                # one note per day, freeform capture
  weekly/                # weekly review / planning notes
  projects/              # one subfolder per active project or initiative
  areas/                 # ongoing responsibilities with no end date
  reference/              # durable reference material, not time-bound
  people/                 # notes per collaborator/contact, if useful
  inbox/                  # unsorted raw capture, triaged periodically
```

Notes:
- This mirrors a PARA-style split (Projects / Areas / Resources / Archive)
  loosely adapted with `reference/` standing in for Resources and no
  dedicated Archive folder by default — offer adding one if the user has a
  habit of archiving.
- `inbox/` exists so capture never blocks on filing; it should get triaged
  into the other folders, not left to grow indefinitely.
- Do not invent domain-specific folders (e.g. a specific business name) —
  those come from the user's own answers in Phase 1(b).
- `index.md` should be a short dashboard: one line per top-level folder,
  linking to it, plus a one-line description of what belongs there.

````


---

# FILE: assets/templates/AGENTS.md.template

````markdown
# AGENTS.md

Generated by /build-harness. Describes the seeded agent roles and how work
flows between them. Edit freely as roles evolve — this file is not
regenerated automatically after initial setup.

## Roles

<!-- One subsection per role chosen in Phase 1(c). For each: name, one-line
     purpose, tools it needs, and what it is NOT responsible for. -->

### <role name>
- Purpose: <one line>
- Tools: <tool list>
- Not responsible for: <boundary>

## Handoff graph

<!-- Encode explicit edges, not a flat list. See
     references/agent-graph-patterns.md for the default pattern this was
     built from. Redraw this if roles were pruned or added. -->

```
user request
   -> <role> (...)
        -> <role> (...)
        -> <role> (validation gate)
```

## Validation gates

<!-- State explicitly which role(s) must sign off before work is "done," and
     what they check. A builder role should never self-certify its own
     output as complete. -->

````


---

# FILE: assets/templates/BUSINESS.md.template

````markdown
# BUSINESS.md

<!-- Written only from this session's Phase 6 answers. Never pre-filled with
     example or placeholder content — if an answer wasn't given, leave the
     section header with no content rather than inventing one. -->

## Active projects

## Focus area

## Goals

## Audience


````


---

# FILE: assets/templates/CLAUDE.md.template

````markdown
# CLAUDE.md

<!-- Minimal project CLAUDE.md, offered in Phase 4 only if the project has
     none. Fill in from what the user actually says about this project —
     do not invent content. -->

## What this project is

<!-- one or two lines -->

## Memory

Read `.claude/memory/MEMORY.md` at session start for prior context on this
project (user context, feedback on approach, active work, references).

## Notes

<!-- anything else the user wants a fresh session to know immediately -->

````


---

# FILE: assets/templates/SOUL.md.template

````markdown
# SOUL.md

<!-- Written only from this session's Phase 6 answers. Never pre-filled with
     example or placeholder content — if an answer wasn't given, leave the
     section header with no content rather than inventing one. -->

## Voice / tone

## Identity

<!-- who is speaking, in what capacity -->

## Ethics / boundaries

<!-- what this agent should never do or say, per the user -->

## Values


````
