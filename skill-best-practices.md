# Building a Public, Advanced Claude Skill — Research Digest

Compiled 2026-09-26 from Anthropic's official docs, the open Agent Skills spec, the
`skill-creator` and `writing-skills` skills installed locally, and community
write-ups. Sources at the bottom.

---

## 1. The hard rules (spec + Anthropic)

| Item | Rule |
|---|---|
| `name` | 1–64 chars, `a-z 0-9 -` only, no leading/trailing/double hyphen, **must equal the folder name**, must not contain `anthropic` or `claude` |
| `description` | 1–1024 chars, non-empty, third person, no XML tags. In Claude Code `description` + `when_to_use` ≤ 1536 combined |
| `SKILL.md` body | < 500 lines, < ~5000 tokens. Split beyond that |
| References | One hop from SKILL.md. No `a.md → b.md → c.md` chains |
| Long reference files | > 100 lines → put a table of contents at the top |
| Paths | Forward slashes only. Relative to skill root |
| Frontmatter | Must start on line 1 of the file. Field names lowercase, case-sensitive |
| Optional spec fields | `license`, `compatibility` (≤ 500 chars, only if real env needs), `metadata` (string→string map, e.g. author, version), `allowed-tools` (experimental) |

Validate with `skills-ref validate ./my-skill` (agentskills.io) or
`claude plugin validate --strict ./plugin` if packaged as a plugin.

---

## 2. Directory layout

```
my-skill/
├── SKILL.md            # required: frontmatter + instructions
├── scripts/            # executable code (run, not read → only output costs tokens)
├── references/         # docs loaded on demand; one file per domain/variant
├── assets/             # templates, schemas, images used in output
├── evals/              # test cases (see §7)
└── LICENSE             # for a public skill
```

Progressive disclosure is the whole design:
1. **Metadata** (name + description) — always in context, ~100 tokens
2. **SKILL.md body** — loaded when triggered
3. **Bundled files** — zero cost until read or executed

Organize `references/` by *mutually exclusive* context (e.g. `aws.md`, `gcp.md`)
so Claude reads only the relevant one.

---

## 3. The description is the product

It's the only thing Claude sees at startup and decides triggering from among 100+ skills.

- **Third person.** "Extracts…", never "I can…" or "You can…".
- **What + when.** Include concrete trigger phrases users actually type, file types, and
  synonyms. Claude *undertriggers*, so be a little pushy: "Use whenever the user mentions
  X, Y, or Z, even if they don't say 'dashboard'."
- **Exclusion clause.** End with "Do NOT use for …" naming near-miss cases. Community
  practitioners call this the single most valuable line.
- **Don't summarise the workflow.** Superpowers testing found agents follow the description
  instead of reading the body when it describes the process. Triggers only.
- Test with 5+ different phrasings; run the `skill-creator` description optimizer
  (`scripts/run_loop.py`) with 20 realistic should/shouldn't-trigger queries.

Template:
```yaml
description: >-
  <Does X, Y, Z>. Use when the user <situation>, mentions <keywords/filetypes>,
  or asks to <verbs>, even if they don't say "<term>". Do NOT use for <near-misses>.
```

---

## 4. Writing the body

**Assume Claude is smart.** Only add what it doesn't already know. Challenge every
paragraph: "Does this justify its token cost?"

**Explain the why, not MUST.** All-caps imperatives are a yellow flag. Reasoning
generalises to edge cases; rigid rules get rationalised around.

**Match freedom to fragility:**
- High freedom (prose heuristics) — many valid approaches, e.g. code review
- Medium (template/pseudocode with params) — preferred pattern, some variation
- Low (exact script, no flags) — fragile/destructive ops, e.g. migrations

**One default, one escape hatch.** Don't list five libraries. "Use pdfplumber. For
scanned PDFs use pdf2image + pytesseract."

**Consistent terminology.** Pick one word per concept and stick to it.

**No time-sensitive text.** Put deprecated approaches in a collapsed "Old patterns" section.

**Highest-signal section: Gotchas.** Concrete failure modes observed in real runs
("Scanned PDFs return `[]` silently"). Anthropic's own best skills "began as a few lines
and a single gotcha". Grow it from observed failures, never from imagination.

**Patterns that work:**
- *Checklist workflow* — a copyable `- [ ]` list for 3+ step tasks; prevents premature "done"
- *Feedback loop* — draft → validate (script or style guide) → fix → repeat until clean
- *Plan-validate-execute* — emit a JSON plan, validate it with a script, only then act.
  For batch, destructive, or high-stakes operations
- *Templates* — strict for data contracts, flexible ("sensible default") for documents
- *2–3 input/output examples* spanning the expected variation; one excellent example
  beats five mediocre ones, and never port it to five languages
- *Conditional routing* — "Creating? → section A. Editing? → section B."

**Match the form to the failure** (superpowers research):

| Baseline failure | Right form |
|---|---|
| Knows the rule, skips it under pressure | Prohibition + rationalization table + red-flags list |
| Complies but output is wrong shape | Positive recipe: state what the output IS, parts in order |
| Omits a required element | Structural slot in the template, not a prose reminder |
| Should depend on a condition | Conditional keyed to an observable predicate |

Prohibition lists ("don't restate", "never narrate") measurably *backfire* on shaping
problems. Nuance clauses ("unless it matters") reopen negotiation. Don't add them.

---

## 5. Scripts

- **Solve, don't defer.** Handle `FileNotFoundError`, permissions, etc. inside the script
  instead of letting it crash for Claude to figure out.
- **No voodoo constants.** Every number gets a comment explaining it.
- **Verbose errors.** "Field `x` not found. Available: a, b, c" lets Claude self-correct.
- **State intent:** "Run `scripts/x.py`" (execute) vs "See `scripts/x.py`" (read as reference).
- **List dependencies explicitly** with install commands. Claude API sandbox has no network;
  claude.ai can pip/npm install.
- **MCP tools:** always fully qualified `Server:tool_name`.
- **Find script candidates** by running 3 test prompts and noticing helper code the agent
  rewrote each time. Bundle that.
- Use `${CLAUDE_SKILL_DIR}` (skill) / `${CLAUDE_PLUGIN_ROOT}` (plugin) — never absolute
  paths. Hardcoded paths and cached IDs are the #1 thing that leaks from personal skills
  into public ones.

---

## 6. Claude Code-specific frontmatter (beyond the spec)

| Field | Use for |
|---|---|
| `when_to_use` | Extra trigger context appended to description |
| `disable-model-invocation: true` | Side-effect skills (`/deploy`, `/commit`) — user-only |
| `user-invocable: false` | Background knowledge Claude auto-loads, hidden from `/` menu |
| `allowed-tools` | Pre-approve e.g. `Bash(git add *) Bash(git commit *)`. Grants, doesn't restrict. Review carefully — applies even in untrusted repos |
| `disallowed-tools` | Block tools while active |
| `paths` | Glob patterns → auto-activate only when touching matching files |
| `context: fork` + `agent:` | Run in an isolated subagent (no conversation history) |
| `model`, `effort` | Override per skill |
| `arguments`, `argument-hint` | Named `$args`; `$ARGUMENTS`, `$0`, `$1` positional |
| `hooks` | Register lifecycle hooks when the skill fires |
| `` !`cmd` `` in body | Dynamic context injection; non-zero exit aborts the invocation (add `\|\| true`) |

Locations: `~/.claude/skills/` (personal), `.claude/skills/` (project),
`<plugin>/skills/` (namespaced `/plugin:skill`). Skills supersede `commands/`.

---

## 7. Testing (non-negotiable for a public skill)

Anthropic: **build evals before writing extensive docs.** Superpowers: **no skill
without a failing test first** — same as TDD.

1. **Baseline (RED):** run 3+ realistic prompts *without* the skill. Record exactly what
   goes wrong and the agent's verbatim rationalisations.
2. **Write the minimum** that fixes those specific failures.
3. **Re-run with the skill (GREEN).** Compare against baseline.
4. **Refactor:** close new loopholes; move repeated helper code into `scripts/`.
5. **Test on Haiku, Sonnet, and Opus.** What's enough for Opus may be too thin for Haiku.
6. **Watch how Claude navigates** the skill: unexpected read order, ignored files, or
   repeatedly re-read files all signal a structure problem.

Tooling:
- **`skill-creator`** (installed): `evals/evals.json`, with-skill vs without-skill
  subagent runs, `eval-viewer/generate_review.py` for human review, blind A/B comparison,
  and the description-optimizer loop.
- **`claude plugin eval`** (if packaged as a plugin): `evals/<case>/` dirs with a prompt
  and graders (`regex`, `tool_used`, `llm` rubric, `file`). Each case runs 3× with and 3×
  without the plugin; gate CI with `--threshold` and pin `--model`. `claude plugin eval init`
  scaffolds the suite interactively.
- Eval prompts must be *realistic and substantive* — file names, backstory, typos. Simple
  one-step prompts never trigger skills regardless of description quality.
- Should-not-trigger cases must be near-misses, not obviously irrelevant.

---

## 8. Packaging & publishing

**Standalone skill:** the folder itself, or `python -m scripts.package_skill <dir>` from
skill-creator produces a `.skill` file. Installs to `~/.claude/skills/<name>/`.
Cross-runtime: Codex reads `.agents/skills/`, Copilot `.github/skills/`; the SKILL.md
format is shared, so symlinks work.

**Plugin (recommended for public distribution):**
```
my-plugin/
├── .claude-plugin/plugin.json        # only this file goes in .claude-plugin/
├── .claude-plugin/marketplace.json   # optional: self-hosted marketplace, source "./"
├── skills/<name>/SKILL.md
├── README.md
└── LICENSE
```
`plugin.json`: `name` (kebab-case, **permanent** — renaming orphans every install),
`description`, `version` (bump every release or omit and let git SHA drive updates),
`author`, `homepage`, `repository`, `displayName`.

Dev loop: `claude --plugin-dir ./my-plugin` → `/reload-plugins` → `claude plugin validate --strict`.
Ship: users run `claude plugin marketplace add owner/repo` then
`claude plugin install name@marketplace`. Or submit to Anthropic's directory at
claude.ai/directory/manage (paid plan; extra review rules beyond CLI validation).

**Keep plugins focused.** One plugin with 15 skills, 7 commands and 12 hooks should be
split. The marketplace rewards specificity. Every enabled plugin's descriptions sit in
context on *every* turn, so bloat is a real cost to users.

**Pre-publish audit** (Victor Da Luz's lesson: only a handful of 24 personal skills were
publishable as-is):
- Read every line imagining a machine that isn't yours
- Strip hardcoded paths, cached IDs, private integrations, personal knowledge bases
- Rule: if scrubbing personal references leaves the logic intact, ship it; if the personal
  wiring *is* the logic, keep it private
- Remove optimisation shortcuts that won't exist elsewhere
- README must explain each component and how to invoke it, not just "install and use"

---

## 9. Security (users will read your SKILL.md before trusting it)

Snyk found 36.8% of public skills had at least one flaw; 13.4% critical. A malware
campaign shipped 341 malicious skills through one marketplace. Reviewers now:

- Read `SKILL.md`, `hooks/hooks.json`, `.mcp.json`, and `bin/` before installing
- Run `claude plugin details` for a component inventory
- Flag remote downloads, `curl | sh`, credential prompts, and out-of-tree path references

So: no network fetches at load time, no requests for passwords, no code outside the plugin
root, minimal `allowed-tools`, no secrets, pin dependencies, sign releases (marketplace
`sha256` for archives). Principle of least surprise: the skill must do exactly what its
description says.

---

## 10. Anthropic's ranked skill categories (from "How we use skills")

1. Product verification (behavioural tests + assertions + recordings) — highest impact
2. Library/API reference with gotchas
3. Data fetching & analysis
4. Business-process automation (with log-file memory)
5. Code scaffolding
6. Code-quality review
7. CI/CD & deployment
8. Incident runbooks
9. Infra ops with guardrails

One skill, one verb. A skill that restates what Claude already does adds cost and no value.

---

## 11. Pre-ship checklist

- [ ] Name matches folder, kebab-case, no reserved words
- [ ] Description: third person, what + when, trigger keywords, exclusion clause, no workflow summary
- [ ] SKILL.md < 500 lines; references one hop deep; long refs have a TOC
- [ ] Every paragraph justified its tokens; "why" over "MUST"
- [ ] Gotchas section populated from real runs
- [ ] Scripts handle errors, document constants, list deps, use forward slashes and `${CLAUDE_SKILL_DIR}`
- [ ] No hardcoded personal paths/IDs/secrets
- [ ] ≥ 3 evals with baseline comparison; tested on Haiku/Sonnet/Opus
- [ ] Description optimised against 20 should/shouldn't-trigger queries
- [ ] `claude plugin validate --strict` passes; installed once from a local marketplace
- [ ] README, LICENSE, `version`, `author`, `repository` set
- [ ] Security self-review done as if you were a stranger installing it

---

## Sources

Official
- [Skill authoring best practices — Claude Platform Docs](https://platform.claude.com/docs/en/agents-and-tools/agent-skills/best-practices)
- [Agent Skills specification — agentskills.io](https://agentskills.io/specification)
- [Skills — Claude Code docs (frontmatter reference)](https://code.claude.com/docs/en/skills)
- [Create a plugin — Claude Code docs](https://code.claude.com/docs/en/plugins/create)
- [Publish and distribute a plugin — Claude Code docs](https://code.claude.com/docs/en/plugins/publish)
- [Test plugins with evals — Claude Code docs](https://code.claude.com/docs/en/plugin-evals)
- [Plugin security and trust — Claude Code docs](https://code.claude.com/docs/en/plugins/security)
- [Equipping agents for the real world with Agent Skills — Anthropic Engineering](https://www.anthropic.com/engineering/equipping-agents-for-the-real-world-with-agent-skills)
- [Lessons from building Claude Code: how we use skills — claude.dev](https://claude.dev/blog/lessons-from-building-claude-code-how-we-use-skills/)
- [anthropics/skills — GitHub (examples + template)](https://github.com/anthropics/skills)
- [skill-creator SKILL.md — anthropics/skills](https://github.com/anthropics/skills/blob/main/skills/skill-creator/SKILL.md)

Community
- [Skill Authoring Patterns from Anthropic's Best Practices — Generative Programmer](https://generativeprogrammer.com/p/skill-authoring-patterns-from-anthropics)
- [obra/superpowers writing-skills (TDD for skills, form-matches-failure)](https://github.com/obra/superpowers/blob/main/skills/writing-skills/anthropic-best-practices.md?plain=1)
- [Publishing my Claude Code skills as a plugin marketplace — Victor Da Luz](https://vdaluz.com/blog/publishing-claude-code-skills-plugin-marketplace)
- [Agent Skills Guide 2026: Build, Share & Secure — Termdock](https://www.termdock.com/en/blog/agent-skills-guide)
- [Claude Code Plugin Marketplace guide 2026 — The Prompt Shelf](https://thepromptshelf.dev/blog/claude-code-plugin-marketplace-2026/)
- [SKILL.md Frontmatter Reference — tonsofskills](https://tonsofskills.com/docs/reference/skill-frontmatter/)
- [Agent Skills Cheat Sheet — Webfuse](https://www.webfuse.com/agent-skills-cheat-sheet)
- [Skill Authoring Guide gist — lipex360x](https://gist.github.com/lipex360x/3a1a662525e88a3e856b7fda02ab8ce3)
- [Claude Code Skills: Progressive Disclosure — Daniel Avila](https://medium.com/@dan.avila7/claude-code-skills-progressive-disclosure-step-by-step-3ca02a4a9f60)
- [How to Publish a Claude Code Plugin — systemprompt.io](https://systemprompt.io/guides/publish-plugin-claude-marketplace)
