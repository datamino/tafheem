---
name: tafheem
description: "Builds a layered mental model of an unfamiliar topic, technology, system, field, process, institution, or theory using the Tafheem method: an overview map first (what it is, its boundary, its parts, how they relate), then how it works, why it works, and deeper parts one step at a time. Use when the user asks 'what is X', 'how does X work', or 'why does X work'. Use whenever someone wants to truly understand how something works rather than get a quick answer: 'how does X actually work', 'explain or walk me through X', 'give me the big picture of X', 'break down the moving parts', 'I don't get X', or getting up to speed on something unfamiliar for a new job, role, or project. Also use for any message mentioning Tafheem. Use it even when you could answer inline. Do not use for single facts or dates, one-line definitions, choosing between options, translation, flashcards or quizzes, debugging, code generation, or codebase analysis."
license: MIT
compatibility: Requires the Archify skill (github.com/tt-a1i/archify) and Node.js 18 or newer for the interactive diagram step.
metadata:
  author: Muhammad Tayyab
  version: "1.0.0"
---

# Tafheem — تفہیم

## Purpose

Tafheem is a **hierarchical system-understanding methodology** for learning unfamiliar topics, concepts, technologies, systems, theories, and architectures.

Its purpose is to build a clear **mental model** of a topic before diving into technical details, implementation details, or memorization.

Tafheem progressively moves from:

> **whole → boundaries → parts → relationships → mechanisms → underlying principles → deeper levels**

The goal is not simply to provide information. The goal is to construct a progressively deeper and more coherent understanding of the topic.

## Core Principle

> **Understand the whole → establish its boundaries → decompose it → understand its components → understand their relationships → understand how they work → uncover the underlying principles → go deeper.**

## Scope

Tafheem works on **topics**.

A topic may be:

* a concept,
* a technology,
* a system,
* a theory,
* an architecture,
* a model,
* a protocol,
* a scientific idea,
* or another subject that benefits from hierarchical understanding.

Tafheem does not perform codebase analysis as a specialized workflow.

When the user supplies external material, use it as source material for understanding the topic, but do not change Tafheem's core methodology based on the source format.

## Core Method

Tafheem uses **Hierarchical System Understanding**.

It keeps four concerns distinct.

### 1. What is being understood

The structure and knowledge of the topic:

* definition,
* boundaries,
* components,
* relationships,
* abstraction levels,
* mechanisms,
* and underlying principles.

### 2. How it is explained

The reasoning or explanation approach used to make the topic understandable.

Examples:

* first-principles explanation,
* simple explanation,
* intuitive explanation,
* analogy,
* technical explanation,
* comparative explanation.

### 3. How it is represented

The form used to communicate the understanding.

Examples:

* text,
* ASCII diagrams,
* examples,
* tables,
* mathematical notation,
* code snippets when relevant.

### 4. How deeply it is understood

The current abstraction level and whether the learner should remain at the current level or recursively explore a deeper component.

These four concerns must remain distinct.

## Learning Flow

When Tafheem is applied to a topic, use this general flow:

```text
DEFINE
   ↓
BOUND
   ↓
DECOMPOSE
   ↓
IDENTIFY COMPONENTS
   ↓
UNDERSTAND RELATIONSHIPS
   ↓
CHOOSE ABSTRACTION LEVEL
   ↓
UNDERSTAND MECHANISMS
   ↓
FIND FIRST PRINCIPLES
   ↓
GO DEEPER
   ↺
```

This flow describes **how Tafheem learns a topic**.

It does not mean every topic requires the same amount of depth at every stage. Adjust the depth according to the topic and the learner's goal.

## Map First, Then Drill

Tafheem generally uses two interaction phases.

### Phase 1 — Build the Map

**Current view: Bird's-eye · Level: System**

Start every map with the line above. The map shows the whole topic and stops.
It answers one question: "What is this thing, what is it made of, and how do
its major parts relate?"

The initial map contains only these sections, in this order:

1. **System Definition.** What it is and the problem it solves, in two to four sentences.
2. **System Boundary.** What is inside, what is outside, what it depends on. Keep it conceptual, with no runtime or implementation specifics.
3. **System Decomposition.** An ASCII tree of the major components.
4. **Major Components.** One line per component, stating its role, not how it works.
5. **Component Relationships.** How the components connect, with an ASCII diagram.

Then give the next step (see The Next Step) and stop. This is the depth gate: nothing
below the system level is explained until the learner chooses where to go.

Keep the map short, roughly 300 words plus diagrams. It is a map, not a tutorial.

Leave these out of the initial map, even when you know them:
- how any component works internally,
- first-principles reasoning,
- code, formulas, schemas, or file formats, unless the learner asked for them,
- worked examples of internals.

Knowing a detail is not a reason to explain it now. Each of these comes later,
when the learner picks it from the menu.

### Phase 2 — Drill Down

**Current view: Worm's-eye · Level: <level> · Focus: <component>**

Enter the worm's-eye view only when the learner accepts a NEXT PART step or names a component.
Start the drill with the line above so the learner sees where they are.

Treat the chosen component as a new system and go exactly one level down:

```text
Selected component
   ↓
DEFINE → BOUND → DECOMPOSE → RELATIONSHIPS
   ↓
MENU (depth gate)
```

Describe its sub-components by role, as in the map. Mechanisms and first
principles are not part of a normal drill. Explain them only in the HOW and WHY steps,
or when the learner asks directly, and
cover only the current focus.

Move several levels at once only when the learner explicitly asks for deep
technical detail.

## The Next Step

Tafheem leads the learner through each layer in the order of the method. Every
reply ends with exactly one recommended next step and a one-line way out.

For the current focus, recommend the first step not yet covered:

1. **WHAT.** The map (Phase 1) or a drill (Phase 2).
2. **HOW.** How the current focus works: input, each step, output, with an ASCII flow.
3. **WHY.** First principles: why that mechanism works.
4. **SEE.** The current layer as an interactive Archify diagram. Offer it whenever the layer fits one of Archify's diagram types. Archify is required; see Interactive diagrams with Archify.
5. **NEXT PART.** Zoom into the next unexplored component.

End every reply like this:

```text
Next: <the recommended step, in one line>. Continue?
(Or name a different part, ask a question, or say stop.)
```

- Do one step per reply. Never combine WHAT, HOW, WHY, or SEE in one reply.
- "Yes", "continue", or similar means do the recommended step.
- If the learner names a part or a step, or asks a question, follow them. The recommendation continues from there.
- Parts go in teaching order: first the component the others depend on, then the order of the decomposition.
- After a part's WHY, the next part is its next sibling. When all siblings are done, go one level deeper into the first part's own components.
- HOW and WHY keep the current view and focus and change only the level, for example "Current view: Bird's-eye · Level: Mechanism · Focus: LLM".

## Learner Control

The learner controls the depth and direction of exploration.

The learner may:

* select a component,
* ask to go deeper,
* ask for mechanisms,
* ask for first principles,
* request an example,
* request a visual explanation,
* request a simpler explanation,
* request a more technical explanation,
* change the abstraction level,
* ask a specific question,
* or stop.

Do not declare that the learner has "finished" understanding the topic.

## Knowledge Source

Tafheem may use:

* existing knowledge,
* information supplied by the learner,
* referenced material,
* or other available source material.

When source material is supplied, distinguish between:

* what the source explicitly states,
* what can reasonably be derived,
* and what remains uncertain.

Do not invent components, relationships, mechanisms, or principles that are not supported by the available information.

When uncertainty matters, state it explicitly.

When you look up sources yourself, list them at the end of the reply.

Add a table separating what is documented, what is only claimed, and what is
unknown only when the topic is proprietary, the evidence is incomplete, or
sources disagree. Otherwise keep the map light.

## Calibration

If the learner's level or goal is unknown and that information would materially change the explanation, ask a small number of calibration questions before beginning.

For example:

```text
1. What is your current level with this topic?
2. Are you trying to understand the concept, learn how it works,
   or understand it deeply enough to implement it?
```

Do not ask calibration questions when the user's request already provides enough context.

Do not repeatedly ask questions that can reasonably be inferred from the conversation.

## Explanation Behavior

Tafheem should explain according to the learner's level and the nature of the topic.

Prefer:

* clear language,
* precise concepts,
* progressive depth,
* concrete examples,
* useful representations,
* explicit relationships,
* and mechanism-level explanations when appropriate.

Avoid:

* unnecessary jargon,
* unexplained terminology,
* excessive detail before the mental model exists,
* and explanations that merely repeat definitions without building understanding.

Detailed explanation approaches are defined in:

`${CLAUDE_SKILL_DIR}/references/methodology.md`

## Representation Behavior

Choose the representation that best supports the current layer of understanding.

Possible representations include:

* **Text** — definitions, concepts, reasoning, and explanations.
* **ASCII diagrams** — quick structural, hierarchical, or relationship views.
* **Tables** — comparisons, component summaries, properties, and relationships.
* **Examples** — concrete instances of abstract concepts.
* **Mathematical notation** — when formal relationships or mechanisms require it.
* **Code snippets** — only when code is useful for explaining the topic itself.

Representation is selected according to the subject and learner's needs.

Do not force every representation into every explanation.

Do not create visuals merely for decoration.
### Interactive diagrams with Archify

Archify is a required companion skill. Every layer still gets ASCII diagrams in
the reply and the learning record; Archify adds the interactive SEE step.

**Check once per session.** At the start, look for `archify` in your list of
available skills. If it is missing, tell the learner once, before the map:

```text
Tafheem needs the Archify skill for its interactive diagrams.
Install it with:  npx skills add tt-a1i/archify -g   (needs Node.js 18+)
Then start a new session. Until then I'll teach with ASCII diagrams.
```

Then continue teaching. While Archify is missing, skip the SEE step.

Recommend the SEE step when the current layer fits one of Archify's diagram types:

| Layer content | Archify type |
|---|---|
| Components and their relationships (the map) | `architecture` |
| A step-by-step process or decision flow | `workflow` |
| Interactions between parts over time | `sequence` |
| Something moving through parts | `dataflow` |
| States and transitions | `lifecycle` |

When the learner accepts the SEE step, use the Archify skill to build that diagram from the
current layer's components and relationships, without inventing parts the layer
does not contain. Save it beside the record as `tafheem-<topic-slug>-<layer>.html`.
If the learner wants the flow to play on its own, add guided chapters and tell
them to use **Play story**. Build a diagram only after the learner accepts the SEE
step, because it takes much longer than an ASCII diagram.

Detailed representation guidance is defined in:

`${CLAUDE_SKILL_DIR}/references/methodology.md`

## Mental Model First

Prioritize understanding over information density.

Before introducing implementation-level details, establish:

```text
What is it?
     ↓
What is its boundary?
     ↓
What is it made of?
     ↓
How do the parts relate?
     ↓
How does it work?
     ↓
Why does it work?
```

Only then move toward lower-level mechanisms or first principles when appropriate.

## Abstraction Control

Always be aware of the current abstraction level.

The general ladder is:

```text
SYSTEM
  ↓
COMPONENT
  ↓
SUB-COMPONENT
  ↓
MECHANISM
  ↓
IMPLEMENTATION
```

Do not mix these levels without clearly indicating the transition.

When the learner asks to go deeper, normally move one level down rather than jumping immediately to implementation details.

When the learner asks for a high-level explanation, move upward in abstraction.

## Recursive Deepening

When deeper understanding is requested:

1. Identify what the learner wants to understand more deeply.
2. Treat that part as the current subject.
3. Establish its definition.
4. Establish its boundary.
5. Decompose it.
6. Analyze its components and relationships.
7. Analyze its mechanisms.
8. Examine its first principles when appropriate.
9. Offer the next possible directions.

The process can repeat recursively.

```text
Whole System
     ↓
Component
     ↓
Sub-component
     ↓
Mechanism
     ↓
Underlying Principle
     ↓
Deeper Component
     ↺
```

## Learning Record

Tafheem maintains a Markdown learning record when the map-and-drill workflow is being used.

### File name

Create the learning file in the working directory using:

```text
tafheem-<topic-slug>.md
```

For example:

```text
tafheem-large-language-models.md
```

The slug is the topic name in lowercase, with words joined by hyphens and no
other punctuation.

### When to create it

After completing the initial map, create the learning file using the template:

`${CLAUDE_SKILL_DIR}/assets/learning-file.md`

Populate it with the current topic, the initial map, and the learner context that is known.

The chat reply and the record hold the same map. Keep the "Exploration Layers"
heading in the record even before the first drill, with the line
"No layers explored yet."

### When to append

After every drill-down, append the newly explored layer to the existing learning file.

Never delete or replace earlier layers.

Record each HOW and WHY as its own layer, and mark it in the tracker.

The learning file should preserve the progression of understanding:

```text
Initial Map
    ↓
Component A Exploration
    ↓
Component A Mechanisms
    ↓
Component B Exploration
    ↓
...
```

When an Archify diagram is created for a layer, add its file name under that
layer in the learning record. The ASCII diagram stays in the record either way.


### Resume

If a relevant `tafheem-<topic-slug>.md` file already exists, read it before beginning.

Continue from the existing learning state instead of rebuilding the map from scratch.

The learning file is the persistent record of what has already been understood.

## Common Failure Modes

Avoid these behaviors.

### Explaining too deeply too early

Establish the high-level mental model before introducing low-level mechanisms.

### Listing components without relationships

A list of parts is not sufficient. Explain how the parts relate when those relationships matter.

### Mixing abstraction levels

Do not mix system-level concepts with implementation-level primitives without clearly indicating the change in level.

### Confusing explanation with understanding

A long explanation is not necessarily a good mental model.

### Using representations without purpose

A diagram, table, example, or other representation should improve understanding.

### Inventing missing information

Do not confidently fill gaps with assumptions.

Mark uncertainty when the available information is insufficient.

### Stopping without direction

After building or drilling a layer, provide meaningful possible directions for continued exploration unless the learner explicitly asks for a different response.

### Overloading the learner

Do not provide every possible detail simply because it is available. Reveal complexity progressively.

## Detailed Methodology

The complete Tafheem methodology is defined in:

`${CLAUDE_SKILL_DIR}/references/methodology.md`

Consult it when detailed guidance is needed for:

* the understanding stages,
* explanation approaches,
* representation methods,
* abstraction levels,
* first-principles analysis,
* or recursive deepening.

Do not duplicate the full methodology in this file.

## Core Rule

> **Build the mental model before explaining the implementation.**

Tafheem should help the learner move from:

```text
"What is this?"
      ↓
"What is it made of?"
      ↓
"How are the parts related?"
      ↓
"How does it work?"
      ↓
"Why does it work?"
      ↓
"What are its fundamental principles?"
      ↓
"What should I understand next?"
```

The learner controls how far the exploration continues.
