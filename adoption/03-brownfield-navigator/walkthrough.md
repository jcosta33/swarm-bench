# Adoption walkthrough 3 — brownfield refactor, inventory + change plan

> **A verified illustration**, not a field audit. The product/persona are fictional; the `swarm check`
> output below was **really executed** against a filled change plan in a throwaway workspace and pasted
> verbatim. "Documented gaps" are checkable surface facts.

## Premise

**Persona.** Priya, an engineer on an existing ~80k-LOC service. The next job is a structural refactor —
extract the pricing math out of the checkout controller — done by an agent, with **no change to observable
behavior**. She needs the agent boxed in by *preservation guarantees*, not a loose "refactor this."

**Adopts.** The core loop **+ the advanced brownfield tier**: an **inventory** (map the code first) and a
**change plan** (enumerate what must survive, in waves). The `large-pr-review` shape from
`docs/examples/large-pr-review.md`.

## The walkthrough

### 1. Inventory before touching anything

Priya writes an `inventory/` doc from `templates/inventory.md` — current modules, interfaces, behavior,
tests, unknowns. It *observes, never judges* (the auditor discipline). This anchors the change plan to what
the code actually does today.

### 2. The change plan — preservation as a table

From `templates/change-plan.md`, the load-bearing section is **Behavioral preservation guarantees**: a
table binding each behavior to the spec AC it must keep and the command that proves it.

```markdown
## Behavioral preservation guarantees
| ID | Behavior | Verify with |
|---|---|---|
| SPEC-checkout-discount#AC-001 | a valid code still reduces the total by its percentage | npm test -- discount.applies |
| SPEC-checkout-discount#AC-002 | an invalid/expired code still leaves the total unchanged | npm test -- discount.rejects |
```

plus `## Transformation waves` (each wave leaves the suite green), `## Cutover conditions`, and
`## Rollback criteria`.

### 3. Check it mechanically (the one CLI touch)

Priya runs the check on the change plan:

```
$ swarm check change-plans/extract-pricing.md
change-plans/extract-pricing.md  ✓ clean  0 errors, 0 warnings
EXIT: 0
```

The check validates the change-plan contract — that the `preserves:` refs resolve (C010) and the waves are
present (C011). A change plan that gestured at preservation without resolvable AC refs, or shipped no
waves, would not pass clean.

### 4. The two-tier review

When the agent's ~40-file PR lands, the review packet carries **two** coverage tables — requirement
coverage *and* change-plan coverage (each preservation guarantee, its Verify result) — so the reviewer
confirms both "the feature is right" and "nothing else moved." This is the `large-pr-review.md` punchline:
read the table rows + exceptions, open a handful of files, not 40.

## Documented gaps (checkable surface facts only)

1. **`swarm check` does validate the change-plan contract** (C010 preserves-refs, C011 waves) and passed
   the filled plan clean. *(Checkable: re-run step 3; a plan with an unresolvable `preserves:` ref or no
   `## Transformation waves` would not be clean.)* Recorded as a *capability*, not a defect.
2. **Inventory + change plan are advanced-tier, discovered only if you read `docs/05`.** They are not in
   the core three guides; an adopter who never opens `docs/05-brownfield-and-change-plans.md` won't know
   the brownfield path exists. *(Observation of the core-vs-advanced tiering, ADR-0064.)*

## What it illustrates

The brownfield adoption shape: the agent is boxed by *preservation guarantees the CLI can mechanically
check*, and the review proves both the change and the non-change — the case where Swarm's structure earns
the most over an ad-hoc "refactor this" prompt.

## To make this a real demo (Phase 2 seed)

This is the natural **Yap** slot: Yap is a real existing product, so a real inventory → change plan → agent
PR → two-tier review on a Yap refactor is the credible brownfield demo (the field-grounded counterpart to
this fictional illustration).
