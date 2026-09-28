# Tafheem Skill — Design Spec

Date: 2026-09-26
Status: approved for drafting

## 1. What Tafheem is

Tafheem (تفہیم, "understanding") is a hierarchical system-understanding methodology.
Scope, stated exactly:

> Tafheem helps the learner understand a topic by constructing a hierarchical mental
> model of it, from the whole down to its components, relationships, mechanisms, and
> first principles.

A topic can be a concept, technology, system, theory, architecture, or model. Claude
builds the model from its own knowledge, using any material the user supplies as source.
Reading a codebase or a document with a dedicated procedure is out of scope.

Two things are kept strictly separate throughout:

- **Learning methodology** — the nine stages. Lives in `references/methodology.md`. Never
  redefined by the interaction design.
- **Skill architecture** — how Claude exposes the methodology to a learner. Lives in
  `SKILL.md`, the input references, assets, scripts, and evals.

## 2. Decisions made

| Question | Decision |
|---|---|
| Primary job | Teach the user (not Claude's own internal analysis) |
| Flow | Hybrid: compact full map first, then user-driven drill-down |
| Inputs | A topic named by the user (concept, technology, system, theory, architecture, model). Supplied material is used as source, not routed separately |
| Outputs | Structured markdown in chat · saved learning file (.md) · ASCII diagrams when structural. No rendering step |
| Depth | User decides. Every layer ends with a next-layer menu. Claude never ends the session on its own |
| Diagrams | Conditional: include when a layer describes structure, hierarchy, or relationships |

## 3. Layout

```text
tafheem/
├── SKILL.md                 # ~250 lines, loaded every session
├── references/
│   └── methodology.md       # the nine stages + four-concern distinction; one file, with TOC
├── assets/
│   └── learning-file.md     # skeleton for tafheem-<topic>.md
└── evals/
    └── evals.json           # 3 prompts, one per input type, with assertions
```

Rules applied from the research: everything one hop from SKILL.md; only content that is
not needed on every turn gets its own file (methodology detail); SKILL.md must be operable
without reading anything else.

## 4. Session flow (interaction design)

**Calibration (only if unknown).** Two questions: the learner's current level and their
goal. Skipped when the request already states them.

**Phase 1 — Map.** One response covering the whole subject at one level of depth.
Invokes stages Define, Bound, Decompose, Component Analysis, Relationship Analysis.
Fixed sections, ~400 words plus a structure diagram. Ends with the menu.

**Phase 2 — Drill.** Repeats. The user picks a component; Claude treats it as a new
system one abstraction rung deeper and invokes Re-define, Re-bound, Re-decompose,
Component Analysis, Relationship Analysis, Mechanism Analysis. Once mechanisms are
reached the menu also offers First Principles. Ends with the menu.

**Menu format (every layer).** "Go deeper into: A · B · C" plus "mechanisms of the whole"
or "first principles of this" as applicable.

**Learning file.** Created at the map, appended at every drill. Records the tree, each
explored layer with its diagram, an exploration tracker (explored / unexplored per
component), and the current menu. Enables resuming in a later session.

**Explanation and representation.** Chosen per layer from the menus in methodology.md,
based on learner level and what the content needs. They never change the stage order.

## 5. File contents

### SKILL.md
Frontmatter (name `tafheem`, third-person description with trigger phrases and a
"Do NOT use for" clause). Overview and scope. Knowledge-source rules (see below). Phase 1
contract. Phase 2 contract. Menu format. Learning-file rules. Calibration.
Gotchas. Pointer to methodology.md.

Draft description:
> Builds a hierarchical mental model of any unfamiliar topic, concept, technology,
> system, theory, or architecture using the Tafheem method: whole, boundary, components,
> relationships, mechanisms, first principles, then deeper on demand. Use when the user
> says tafheem, wants to understand or learn how something works, or asks for a breakdown
> or mental model of a subject. Do NOT use for quick factual lookups, debugging a
> specific error, reading a codebase, or writing code.

Knowledge-source rules (folded in from the former input-concept reference): the boundary
comes from the learner's stated goal; cross-check the decomposition against how the field
itself divides the subject; mark uncertainty explicitly; prefer canonical component names
over invented ones; when the user supplies material, treat it as the source and decompose
by the system it describes, not by its headings.

Initial gotchas (to be replaced by observed ones after evals): explaining past the point
asked; listing components without relationships; inventing components or mechanisms
with false confidence; decomposing supplied material by its headings; ending a layer
without a menu.

### references/methodology.md
TOC. Four-concern distinction (what / how explained / how represented / how deep).
One section per stage, each stating only: what it produces, the questions it must
answer, the stop condition. Abstraction ladder (system → component → sub-component →
mechanism → implementation; one drill = one rung). Six explanation approaches with
"use when". Six representation methods with "use when" and the conditional diagram
rule. Recursive-deepening protocol.

### assets/learning-file.md
Frontmatter: topic, input type, date, learner level, goal. Sections: Map (tree),
one per explored layer (with diagram), Exploration tracker, Current menu.

### evals/evals.json
Three topic prompts at different levels and domains, e.g. "help me understand how TCP
works, I know basic networking"; a non-technical system ("how does the central bank
control inflation"); an abstract theory ("explain attention in transformers, I know
linear algebra"). Assertions: boundary section present; every layer ends with a menu;
diagram present for the map; learning file created; a drill goes exactly one rung
deeper on the chosen component.

## 6. Order of work
1. Draft all files in one pass.
2. Run the three evals with and without the skill (skill-creator loop).
3. Review outputs; fix what the runs expose; fill gotchas from real failures.
4. Optimize the description against 20 should/shouldn't-trigger queries.
5. Package as a plugin, `claude plugin validate --strict`, publish via marketplace.json.

## 7. Out of scope for v1
Codebase reading and document/paper reading procedures (removed for focus); HTML
rendering of the learning record (removed, markdown + ASCII only); Claude-internal
analysis mode; hooks; flashcards/quiz. Revisit after evals.
