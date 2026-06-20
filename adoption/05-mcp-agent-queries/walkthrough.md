# Adoption walkthrough 5 — the agent queries its scope over MCP

> **A verified illustration**, not a field audit. The product/persona are fictional; the MCP server,
> its tool list, and every response below were captured by **really driving the built `swarm-mcp`
> server over stdio** (MCP SDK `Client` + `StdioClientTransport`) against a real workspace and the real
> `swarm` binary. "Documented gaps" are checkable surface facts, never imagined friction.

## Premise

**Persona.** A two-person team building on Claude Desktop / Cursor. They like Swarm's discipline but want
the **agent to query its task scope from a tool** rather than re-read (and possibly mis-summarize) the
packet — and they want the tool to surface *facts only*, never a verdict the agent could rubber-stamp.

**Adopts.** The starter kit **+ swarm-cli + swarm-mcp** (the MCP server is a thin stdio adapter over
`swarm <cmd> --json`). Builds on the same `checkout-discount` workspace as walkthrough 4.

## The walkthrough

### 1. Point the agent at the server

The MCP server is configured in the agent's MCP config (it requires the `swarm` binary on PATH — it
shells out to it):

```json
{ "mcpServers": { "swarm": {
    "command": "swarm-mcp",
    "args": ["--workspace", "/path/to/workspace", "--swarm-bin", "swarm"] } } }
```

On start it announces what it's bound to:

```
swarm-mcp: ready (workspace=/…/04-cli, swarm=/…/swarm-cli/bin/swarm.js)
```

### 2. What the agent can ask — 10 read/reconcile tools

```
=== listTools (10 tools) ===
  swarm_get_status        swarm_check_workspace   swarm_check_file
  swarm_scan_task         swarm_reconcile_review  swarm_validate_review_packet
  swarm_get_task          swarm_get_spec          swarm_get_review
  swarm_get_checks
```

Every tool is read-only or reconcile-only. None writes; none returns a Pass/Fail.

### 3. "What's on the board?" — `swarm_get_status`

```jsonc
{
  "noVerdictIssued": true,
  "noVerdictNote": "swarm-mcp surfaces facts only and issues no verdict. A human or an independent
                    reviewer owns the review result …",
  "source": { "command": "swarm status --json", "exitCode": 0 },
  "ok": true,
  "data": {
    "level": "clean",
    "specs": [ { "id": "SPEC-checkout-discount", "status": "ready",
      "tasks": [ { "id": "TASK-checkout-discount", "status": "ready",
                   "hasReview": false, "reviewStatus": null } ] } ],
    "tasksWithoutReview": [], "needsHuman": []
  }
}
```

The agent gets the board as data — and `noVerdictIssued: true` with a note that the *human* owns the
result. This is the reconcile-only boundary, surfaced in every payload.

### 4. "What's my task scope?" — `swarm_get_task` → a real bug

The status above returned the task id `TASK-checkout-discount`. The agent passes exactly that id to
`swarm_get_task`:

```jsonc
// callTool swarm_get_task { task: "TASK-checkout-discount" }
{
  "source": { "command": "swarm show task checkout-discount --json", "exitCode": 2 },
  "ok": false,
  "data": { "error": "Usage", "message": "no tasks/checkout-discount.md in this workspace" },
  "note": "no tasks/checkout-discount.md in this workspace"
}
```

It **fails** — even though the task exists. See Documented gaps #1.

## Documented gaps (checkable surface facts only)

1. **`swarm_get_task` can't resolve a task created by `swarm new task` (a real bug).** The MCP strips the
   `TASK-` prefix from the id before shelling out — `swarm_get_task("TASK-checkout-discount")` runs
   `swarm show task checkout-discount`, which looks for `tasks/checkout-discount.md`. But `swarm new task`
   writes `tasks/TASK-checkout-discount.md`, so the lookup misses (exit 2). Directly checkable: in the
   workspace, `swarm show task TASK-checkout-discount` **succeeds** while `swarm show task
   checkout-discount` **fails** (`no tasks/checkout-discount.md`). So the agent, handed the exact id
   `swarm_get_status` returned, gets "not found" for a task that exists. *(Filed as a swarm-hq finding.)*
2. **The server requires `swarm` (swarm-cli) on PATH** — it is a thin adapter that shells out to
   `swarm <cmd> --json`. An adopter who configured the MCP server without installing swarm-cli first gets
   no tools that work. (Observation of the `--swarm-bin` requirement + the startup line.)

## What it illustrates

The "agent queries its scope from a tool, and the tool never issues a verdict" adoption: 10 read/reconcile
tools, `noVerdictIssued: true` in every payload, swarm-cli as the only dependency. It also shows why
*executing* the surface matters — the headline use case (`get_task` for the agent's own scope) is broken
for CLI-created tasks, which a written-from-imagination walkthrough would have shown working.

## To make this a real demo (Phase 2 seed)

Wire the real `swarm-mcp` into a Claude Desktop/Cursor config against a real `checkout-discount` workspace,
and record an actual agent session that calls `swarm_get_task` / `swarm_reconcile_review` mid-task — after
the `get_task` prefix bug (gap #1) is fixed, so the demo shows the intended happy path.
