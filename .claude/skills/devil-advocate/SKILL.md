---
name: devil-advocate
description: Build the case against an agreed grilling session, spec, or ticket set — technical findings the agent resolves itself; only business findings go to the Creator, one at a time; rulings fold back into the artifact. Mandatory between the end of grilling and the epic draft (gate 1).
argument-hint: "Spec or ticket reference to attack — blank for the current session"
---

# Devil's Advocate

Assume something already agreed is wrong somewhere, and build the case against it: the design from the current grilling session, a spec from `/to-spec`, or a ticket set from `/to-tickets`. The deliverable is the amended artifact — never implementation code.

In this repo the step is **mandatory right after a grilling session closes and before the epic draft goes to the Creator**: an open finding blocks gate 1 (see "Contro-esame obbligatorio" in `docs/agents/workflow.md`).

## 1. Load the target

The issue tracker vocabulary lives in `docs/agents/issue-tracker.md` — read it if you haven't already.

- An argument (path, issue number, URL) → fetch full body and comments; for tickets, every sibling with its blocking edges.
- No argument → the design agreed in the current conversation.

Read the whole artifact before attacking any part — the strongest flaws live between distant sections.

## 2. Build the case

First read the domain glossary (`CONTEXT.md`) and the ADRs touching the area (`docs/adr/`, see `docs/agents/domain.md`; files that don't exist yet are fine — proceed silently): the case is argued in the App's terms, never in the abstract.

Two sweeps, both mandatory, on the decisions as written: the artifact needn't be implemented, and existing code is evidence, never the boundary of the hunt.

**War-game the design** — run the mechanism mentally under hostile conditions; does it still serve its purpose?

- **Self-defeating mechanism** — its failure mode is the very thing it exists to eliminate (the watchdog that dies silently), or its pressure lands on an actor who cannot act (alarm fatigue).
- **Promise vs platform** — a stated guarantee the substrate cannot honor (exactly-once wording on an at-least-once pipeline, ordering nowhere enforced).
- **Hostile input and scale** — untrusted values reaching trusted surfaces; unbounded repetition (loops, storms) with no ceiling.

**Read the artifact against itself** — hunt each category across the whole artifact:

- **Contradiction** — two decisions that cannot both hold: within the artifact, against an ADR or the glossary, or two tickets shaping the same seam incompatibly.
- **Gap** — a case silently assumed away: failure paths, concurrency, existing data, the empty state, permissions.
- **Ambiguity** — a line two implementers would ship two different ways.
- **Untested assumption** — a decision resting on an unverified "fact": check the checkable yourself; only the unverifiable becomes a finding.
- **Broken sequencing** (tickets) — a blocking-edge cycle, an undeclared dependency, a slice that cannot land green under its declared blockers.

A candidate becomes a **finding** only if no existing resolution turns up in the artifact or codebase — an answered objection is noise that spends the patience real findings need. Every finding carries its evidence.

The case is complete when every mechanism is war-gamed and every category hunted across every section, distant pairings included. An artifact that survives is a legitimate outcome: say so plainly.

## 3. Resolve the findings

Sort each finding by the kind of decision it needs:

- **Technical finding** — the fix is a matter of implementation (a seam, a contract, a failure path, a data-model detail, a sequencing edge). **You resolve it yourself**: adopt your recommended resolution and fold it into the artifact. Do **not** take it to the Creator — a technical question put to a non-technical Creator is friction, and the recommended resolution is the one to take.
- **Business finding** — the fix changes *what the App does* (a product behavior, a scope call, a trade-off the Creator owns). **This goes to the Creator**, phrased in the Creator's terms with your preferred resolution as the recommended answer. Each ruling is a decision; "accepted risk" is legitimate — record it and move on.

Run the business findings as a `/grilling` session — one finding per question, the one most likely to change the design first. Business rulings come from the Creator, not from you: if the Creator is unavailable, the *business* findings stay open and the process stops here. Technical findings never block on Creator availability — you have already resolved them.

## 4. Amend the artifact

Fold every ruling back into where it lives:

- Spec or tickets → amend the affected sections on the tracker (or file); accepted risks land in Further Notes or the ticket body.
- Session → restate the amended decisions in the conversation: they are the basis for the epic draft.
- A ruling overturning an ADR or glossary term → `/domain-modeling`.

Done when every finding is folded into the artifact or recorded there as accepted risk. Only then may the epic draft go to the Creator — an unadjudicated finding means gate 1 is not reachable yet.
