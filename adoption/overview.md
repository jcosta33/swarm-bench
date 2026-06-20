# Adoption walkthroughs — the pick-and-choose map + collected gaps

## The minimal → full map

Each row adds to the one above; "Executed" is what was really run to ground that walkthrough.

| # | Adopted parts (cumulative shape) | Tooling | Executed grounding |
|---|---|---|---|
| 1 | review packet | none | — |
| 2 | spec · task · review · finding (core formats) | none | — |
| 3 | + inventory · change-plan (brownfield) | swarm-cli (check) | `swarm check` on a change plan (clean) |
| 4 | + the mechanized loop | swarm-cli (full) | init · new · check · worktree · status · review |
| 5 | + agent-queries-scope | + swarm-mcp | real server, 10 tools, get_status / get_task |
| 6 | + conditioning stances/guides | + swarm-skills | npx `--list` + folder install |
| 7 | + hooks + conventions | all | whole-workspace check, hook, MCP, skills |

The cost curve is honest: walkthroughs 1–2 install **nothing** (plain markdown); 3–4 add the CLI; 5–7 add a
separate repo each (mcp, skills) and the integration seams between them.

## Collected checkable gaps (across all 7)

Surfaced by **executing the real surface** — these are checkable facts, not conjecture. The two marked
**FILED** are real bugs worth tracking (a swarm-hq finding each).

| Gap | Severity | Where | Checkable by |
|---|---|---|---|
| **`npm install -g swarm-cli` (README) installs an unrelated project** — the npm name is gritzko's filesystem Swarm client (1.2.2), not the framework CLI (1.0.0); the kit hook says "not yet on npm" | **adoption blocker · FILED** | 4, 7 | `npm view swarm-cli` vs `swarm-cli/README.md` |
| **`swarm_get_task` strips `TASK-` and misses CLI-created tasks** — `get_task("TASK-x")` → `swarm show task x` → `no tasks/x.md` (file is `TASK-x.md`) | **MCP bug · FILED** | 5, 7 | `swarm show task TASK-x` works; `swarm show task x` fails |
| Fresh `init` workspace fails whole-workspace `swarm check` (exit 2) on unfilled `AGENTS.md` placeholders | first-run gate (arguably intended) | 4, 7 | `swarm check` before filling AGENTS.md |
| `swarm worktree create` requires an initial commit | first-run ordering | 4, 7 | `worktree create` on a no-commit repo |
| `ADOPTING.md` documents only copy-whole; no 1-part path, despite README "adopt just the review packet" | doc/promise mismatch | 1 | grep ADOPTING for a partial path |
| The `persona-*` stances are not in the kit core — install per task | by-design tiering | 6 | `ls` a fresh `init` skills dir |
| `npx skills add` emits TUI spinner control characters in agent-detected mode | cosmetic | 6 | re-run `npx skills add … --list` |
| inventory + change-plan are advanced-tier, discoverable only via `docs/05` | discoverability | 3 | core-vs-advanced tiering (ADR-0064) |

## The honest headline

The biggest takeaway is a **validation of the method, not the product**: *executing* the real surface
(rather than narrating from imagination) surfaced **three real bugs** — a documented install that gets the
wrong package, an MCP tool broken for CLI-created tasks, and a fresh-workspace gate — that a hand-written
"happy path" walkthrough would have shown working. The integration seams between the separately-shipped
repos (canon ↔ npm, swarm-cli ↔ swarm-mcp) are where a real adopter snags. None of this is field evidence
of *UX* friction (that's Phase 2) — it's checkable *surface* fact.
