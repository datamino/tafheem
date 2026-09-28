# Tafheem — تفہیم

**Learn any unfamiliar topic the way experts hold it in their heads: see the whole first, then go one level deeper at a time.**

Tafheem (Urdu for *understanding*) is an agent skill for Claude. Ask it to help you understand a topic, and instead of dumping a long explanation it builds a **hierarchical mental model** with you, one step per reply:

```text
WHAT   the map: what it is, its boundary, its parts, how the parts relate
  ↓
HOW    how it works, step by step
  ↓
WHY    the first principles that make it work
  ↓
SEE    interactive diagram, built with Archify
  ↓
NEXT PART   zoom into one part and repeat
```

Every reply ends with one recommended next step. You say **yes** to follow the path, or go wherever you want.

---

## Install

Tafheem needs two things: the Tafheem skill itself and the [Archify](https://github.com/tt-a1i/archify) skill, which draws its interactive diagrams.

**Prerequisite:** [Node.js](https://nodejs.org) 18 or newer, for Archify. Check with `node --version`.

### Step 1: install Archify (required)

```bash
npx skills add tt-a1i/archify -g
```

This works for Claude Code, Codex, Cursor, and OpenCode.

### Step 2: install Tafheem

#### Claude Code (recommended)

```bash
claude plugin marketplace add datamino/tafheem
claude plugin install tafheem@tafheem
```

Start a new session. Tafheem loads automatically when you ask to understand something, or run it directly with `/tafheem:tafheem`.

#### Claude Code, without the plugin system

```bash
git clone https://github.com/datamino/tafheem.git
cp -R tafheem/tafheem ~/.claude/skills/tafheem
```

Run it directly with `/tafheem`.

#### Check it worked

Start a new session and type `/`. You should see both `tafheem` and `archify` in the list. If Archify is missing, Tafheem tells you how to install it and keeps teaching with ASCII diagrams until you do.

#### Other agents

Tafheem uses the open [Agent Skills](https://agentskills.io/specification) format. Copy the `tafheem/` folder into your agent's skills directory, for example `.agents/skills/` for Codex or `.github/skills/` for GitHub Copilot.

---

## Try it

Paste one of these into a new session:

```text
Help me understand how TCP works. I know what an IP address and a router are,
but I have a networking exam in two weeks and TCP still confuses me.
```

```text
Tafheem: how does a central bank control inflation? I'm a business student,
not an economist.
```

```text
Build me a mental model of attention in transformers. I know linear algebra and
basic neural nets, and I eventually want to implement it myself.
```

---

## What a session looks like

**Your first message** gets the map, the bird's-eye view, and nothing deeper:

```text
Current view: Bird's-eye · Level: System

System Definition      what it is, the problem it solves
System Boundary        what's inside, outside, and what it depends on
System Decomposition   an ASCII tree of the major parts
Major Components       one line per part: its role, not its internals
Component Relationships how the parts connect, with a diagram

Next: HOW, trace one request through the whole system. Continue?
(Or name a different part, ask a question, or say stop.)
```

**You say "yes"** and it explains how the whole thing works. **"Yes" again** gets you why it works. Then it recommends the first part to zoom into, in teaching order, and repeats the same path for that part.

Every reply says where you are, for example `Current view: Worm's-eye · Level: Component · Focus: Tokenizer`, so you never lose your place in the hierarchy.

---

## How to use it effectively

1. **Say your level and your goal in the first message.** "I'm a web developer, I want to understand it well enough to use it at work" lets Tafheem skip the calibration questions and draw the right boundary. Without it, Tafheem may ask two quick questions first.
2. **Follow the path when you're new to a topic.** The order WHAT → HOW → WHY exists because each step makes the next one easier to understand. Just reply "yes".
3. **Take control whenever you want.** You can say things like:
   - "go deeper into the tokenizer"
   - "zoom out"
   - "simpler" or "more technical"
   - "show me an example"
   - "skip to why"
   - "compare it to X"
4. **Expect short replies, and treat that as intentional.** Each reply covers one level. If you want more, ask to go deeper. The skill won't pile on everything at once.
5. **Ask questions any time.** Tafheem answers, then offers to continue the path from where you were.
6. **Give it source material** such as a paper, a spec, or your team's docs. Tafheem separates what the source states, what can be derived from it, and what remains uncertain.
7. **Keep the learning record and come back later.** Start a new session in the same folder and say "continue my tafheem on TCP". It reads the record and picks up where you left off.
8. **Stop whenever you like.** Tafheem never declares you "finished". You decide how deep to go.

---

## The learning record

After the first map, Tafheem saves a Markdown file in your current folder, named after the topic, for example `tafheem-large-language-models.md`. Every step you take is appended to it. Nothing is deleted.

The file contains:
- the initial map,
- every layer you explored, with its ASCII diagrams,
- an **exploration tracker** showing which parts have been through WHAT, HOW, and WHY,
- the recommended next step.

It works as your study notes, and it's what makes resuming possible.

---

## Interactive diagrams with Archify

The **SEE** step turns the current layer into an interactive HTML diagram with pan, zoom, relationship tracing, and a guided "Play story" mode. Tafheem builds it with the Archify skill, which is why Archify is a required install.

Things to know:
- **Every layer still gets ASCII diagrams** in the reply and the learning record. Archify adds the interactive version on top.
- **Archify diagrams take a few minutes to build,** because Archify validates and repairs each one. Tafheem builds one only when you accept the SEE step.
- **Archify contacts its author's server each time it runs** to check for updates. It only shows a notice and never installs anything, but it is a network call.
- **SEE is offered when the layer fits a diagram type:** architecture, workflow, sequence, dataflow, or lifecycle. First-principles layers usually don't, so SEE is skipped there.
- **Archify is a separate project** by its own author, under the MIT license. Tafheem uses it; it doesn't ship it.
---

## What Tafheem is not for

- Quick factual lookups or one-line definitions. Just ask directly.
- Debugging a specific error.
- Writing code.
- Analysing a codebase.

---

## How it's built

| File | Purpose | Loaded |
|---|---|---|
| `tafheem/SKILL.md` | The operating instructions: session flow, map contract, next-step rules, learning record | When the skill runs |
| `tafheem/references/methodology.md` | The full Tafheem method: nine stages, abstraction levels, bird's-eye and worm's-eye views, explanation and representation approaches | Only when a stage needs detail |
| `tafheem/assets/learning-file.md` | Template for the learning record | Once, when the record is created |
| `tafheem/evals/evals.json` | Test prompts and pass/fail checks used to develop the skill | Never, during normal use |

Context cost: about 200 tokens in every session for the skill's name and description, and about 5,400 tokens when it runs.

### The method

Tafheem keeps four concerns separate:

- **What** is understood: the structure.
- **How** it is explained: the approach.
- **How** it is represented: text, diagrams, tables, examples.
- **How deeply** it is understood: the abstraction level.

The learning flow is:

```text
DEFINE → BOUND → DECOMPOSE → IDENTIFY COMPONENTS → UNDERSTAND RELATIONSHIPS
   → CHOOSE ABSTRACTION LEVEL → UNDERSTAND MECHANISMS → FIND FIRST PRINCIPLES
   → GO DEEPER ↺
```

The full method is in [`tafheem/references/methodology.md`](tafheem/references/methodology.md).

---

## License

[MIT](LICENSE) © 2026 Muhammad Tayyab
