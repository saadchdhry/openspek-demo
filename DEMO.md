# OpenSpec Demo: Run of Show

**Verified against `@fission-ai/openspec` v1.7.0 on 2026-07-31.** Every command, path, and output in this document was executed in this repository, and the git history is the recording of that run. Where the tool surprised me, the surprise is written down rather than smoothed over.

**Goal of this demo:** show how OpenSpec turns "vibe prompting a coding agent" into a reviewable, versioned, spec-first workflow. The audience should leave able to answer three questions:

1. Where does the *current truth* about the system live? (`openspec/specs/`)
2. Where does a *proposed change* live before it is real? (`openspec/changes/<change-id>/`)
3. What makes a change real? (finish the tasks, then `openspec archive`, which folds the delta into the specs)

**Important scope note:** we write **no application code**. Only specifications. Every time someone asks "but where's the app?", that is the point being made: the spec is the artifact under review, not the code.

**Demo app:** **Splitr**, an expense splitter. Three capabilities: `group-management`, `expense-recording`, `settlement`. Two changes:

1. **`add-expense-splitting`** — groups, expenses, even splits, balances, settlement. Creates all three capabilities.
2. **`add-multi-currency`** — expenses carry a currency, which **modifies** four existing requirements and **removes** one.

Alternates, if this audience is wrong for it, are in [Appendix A](#appendix-a-alternate-demo-apps).

**Target runtime:** 18 to 22 minutes, plus questions.

---

## Pre-demo checklist

- [ ] `npm install -g @fission-ai/openspec` and confirm `openspec --version` reports 1.7.x. **If it reports a different major or minor, re-verify this document.** The command surface moved noticeably between versions.
- [ ] Terminal font size bumped. Two panes: terminal left, file tree right.
- [ ] Rehearse Beat 6 twice. It is the best beat and the easiest to fumble.
- [ ] Decide whether you are running the live path or the replay path (below).

### Live path vs replay path

The git history of this repo *is* the demo, one commit per beat:

```bash
git log --oneline
```

```
9c24e77  Archive add-multi-currency          ← Beat 10, second half
fc4ed6a  Propose add-multi-currency          ← Beat 10, first half
d08ef13  Archive add-expense-splitting       ← Beat 9
1f136e5  Set up OpenSpec and propose ...     ← Beats 2-8
b6a2e9f  Add OpenSpec demo run of show       ← the empty room (Beat 1)
```

Useful diffs to have queued up:

```bash
git show d08ef13 --stat        # what archiving creates
git diff fc4ed6a 9c24e77 -- openspec/specs/   # what archiving a MODIFIED delta changes
```

**Replay path (safer):** check out each commit and talk over the result. Zero risk of the agent going sideways on stage.

**Live path (better):** reset to empty and actually run it. Use this if you are confident and the room is small.

**Reset to empty:**

```bash
git checkout b6a2e9f -- . && rm -rf openspec .claude && git status
```

---

## The one-slide mental model (say this before touching the keyboard)

> Two directories. `specs/` is **what is true today**. `changes/` is **what someone is proposing**. A proposal is a *delta* against the specs, not a rewrite of them. You review the delta like a pull request. When the work is done you archive the change, the delta folds into `specs/`, and that becomes the new truth.

```
openspec/
├── config.yaml             # schema choice + project context + per-artifact rules
├── specs/                  # CURRENT truth: what IS built
│   └── <capability>/spec.md
└── changes/                # PROPOSED: what SHOULD change
    ├── <change-id>/
    │   ├── .openspec.yaml  # schema + created date
    │   ├── proposal.md     # why & what
    │   ├── specs/<capability>/spec.md   # the DELTA
    │   ├── design.md       # how (conditional)
    │   └── tasks.md        # implementation checklist
    └── archive/
        └── 2026-07-31-<change-id>/      # date-stamped after archiving
```

---

## Beat 1: The empty room (1 min)

**Say:** "Completely empty directory. No code, no README."

```bash
ls -la
```

**Point at:** nothing. That is the point. We establish what we are building before there is anything to argue about.

Keep this beat short. It is setup.

---

## Beat 2: `init` (2 min)

```bash
openspec init --tools claude
```

Output:

```
OpenSpec Setup Complete

Created: Claude Code
6 skills and 6 commands in .claude/
Config: openspec/config.yaml (schema: spec-driven)

Getting started:
  Start your first change: /opsx:propose "your idea"
```

**Correction worth knowing:** `init` does **not** scaffold `specs/` or `changes/`. It creates exactly one file under `openspec/`: `config.yaml`. The directories appear when they have something to hold. If you promised the audience a tree, you will be standing in front of a single YAML file.

```bash
find openspec .claude -type f | sort
```

```
.claude/commands/opsx/{apply,archive,explore,propose,sync,update}.md
.claude/skills/openspec-{apply-change,archive-change,explore,propose,sync-specs,update-change}/SKILL.md
openspec/config.yaml
```

**Point at:** `.claude/skills/openspec-propose/SKILL.md`. Open it for ten seconds.

**The line to land:** "This is the part that matters. The workflow rules are now in the agent's context automatically. I am not re-explaining my process in every prompt. And notice it is a plain markdown file I can read and edit."

**Note:** the slash commands are `/opsx:*`, not `/openspec:*`. Restart your IDE after `init` or they will not appear.

---

## Beat 3: Teach it the project once (2 min)

There is **no `project.md`**. Project context lives in `openspec/config.yaml` under a `context:` key.

Open `openspec/config.yaml` and show the filled-in block:

```yaml
schema: spec-driven

context: |
  Splitr is a shared-expense tracker: a small group records who paid for what,
  and the app works out who owes whom.

  Tech stack: TypeScript, Node.js, PostgreSQL. HTTP API only, no UI in scope.
  Money is stored as integer minor units (cents), never floats.
  ...
```

Read two lines aloud, not all of them. Then point out the commented-out `rules:` and `operations:` blocks below it.

**The line to land:** "Context, plus per-artifact rules like 'always include a Non-goals section', plus per-operation guidance. Written once. The agent reads it on every artifact it writes."

You will prove that claim in Beat 5.

---

## Beat 4: Propose (2 min)

**Live path:**

```
/opsx:propose Users can create a group, add members, record an expense paid by one member, and split it evenly across the group. Show each member's net balance.
```

**Or drive the CLI directly**, which is what the skill does under the hood and is more legible on stage:

```bash
openspec new change add-expense-splitting
openspec status --change add-expense-splitting
```

```
Progress: 0/4 artifacts complete

[ ] proposal
[-] specs   (blocked by: proposal)
[-] design  (blocked by: proposal)
[-] tasks   (blocked by: specs, design)
```

Markers: `[ ]` ready, `[-]` blocked, `[x]` done.

**This is the beat I did not know existed and it is one of the best.** Do not skip it.

**The line to land:** "The workflow is a dependency graph, not a checklist. Specs are blocked until the proposal exists. Tasks are blocked until both specs and design exist. You physically cannot write the task list before you have written down what the thing should do. That ordering is the entire argument for spec-driven development, and it is enforced rather than suggested."

The `spec-driven` schema is one of several; `openspec schemas` lists them.

---

## Beat 5: The instruction envelope (2 min)

```bash
openspec instructions proposal --change add-expense-splitting
```

This prints an XML-ish envelope containing `<task>`, `<project_context>`, `<output>`, `<instruction>`, `<template>`, and `<unlocks>`.

**Point at two things:**

1. `<project_context>` contains **the exact text you wrote into `config.yaml` in Beat 3**. That is the proof.
2. The envelope carries a warning: *"This is background information for you. Do NOT include this in your output."*

**The line to land:** "This is what the agent actually receives. Not a vague system prompt: a per-artifact instruction with your project context injected, a template to fill, and the explicit note about what is guidance versus what is content. That is why the output is consistent across artifacts and across sessions."

---

## Beat 6: Read the delta, then break it (4 min) — the most important beat

### 6a. Read it

Open `openspec/changes/add-expense-splitting/specs/settlement/spec.md`. Read one requirement and one scenario aloud:

```markdown
## ADDED Requirements

### Requirement: Net balance per member

The system SHALL compute each group member's net balance as the total they paid
minus the total of their shares across all expenses in the group.

#### Scenario: Balances always sum to zero

- **WHEN** balances are computed for any group
- **THEN** the sum of all member balances is exactly 0
```

**The line to land:** "`SHALL` plus at least one scenario. That is the whole trick. A requirement with no scenario is an opinion. Every scenario here is a test that has not been written yet."

Point at the header: `## ADDED Requirements`. "This file is a *diff*, not a document. That is why archiving can merge it mechanically."

### 6b. Break it loudly

Delete the scenarios under one requirement, then:

```bash
openspec validate add-expense-splitting --strict
```

```
✗ [ERROR] settlement/spec.md: ADDED "Settlement does not move money" must include at least one scenario
Next steps:
  - Each requirement MUST include at least one #### Scenario: block
  - Debug parsed deltas: openspec show add-expense-splitting --json --deltas-only
```

Good error. Names the file, the requirement, and the fix. Note it fails **without** `--strict` too.

### 6c. Break it quietly (this is the beat people remember)

Restore the scenarios. Now change a single character: one `#### Scenario:` becomes `### Scenario:`.

```bash
openspec validate add-expense-splitting --strict
```

```
Change 'add-expense-splitting' is valid
```

It passes. Now show what was actually parsed:

```bash
openspec show add-expense-splitting --json --deltas-only
```

The "Settlement plan" requirement went from **4 scenarios to 0**. Not one lost scenario: all four, because the `###` line ended the requirement block early. And validation reports the change as valid.

**The line to land:** "One hashtag. Four scenarios gone, silently, and the tool says everything is fine. I am showing you this because you will hit it. The lesson is not that OpenSpec is broken. It is that `validate` checks structure it can see, and you should read `--deltas-only` before you trust a large delta. The tool's own instructions warn about this in exactly these words: *using 3 hashtags will fail silently.*"

Restore before continuing, and confirm green.

**Why include a flaw in a demo:** because everyone in the room has been sold a tool that only worked in the happy path. Showing the sharp edge and the mitigation buys more credibility than a clean run does.

---

## Beat 7: The read-only tour (1 min)

Quick fire:

```bash
openspec list
openspec list --specs
openspec show add-expense-splitting
```

**There is no `openspec diff`.** I expected one; it does not exist in 1.7. Use `openspec show`, or `--json --deltas-only`, or plain `git diff` after archiving (Beat 10 does exactly that).

**The line to land:** "Everything is plain markdown in git. The CLI is a convenience, not a database. If OpenSpec disappeared tomorrow I would still have my specs."

`openspec view` opens an interactive dashboard. Know whether you want to be in a TUI on stage before you type it.

---

## Beat 8: "Implementation" (1 min)

**Say:** "Normally `/opsx:apply` here and the agent works the task list. We are skipping the code, because the code is the least interesting part of this story."

Tick the boxes in `tasks.md` on screen:

```bash
sed -i '' 's/^- \[ \]/- [x]/' openspec/changes/add-expense-splitting/tasks.md
openspec list
```

```
Changes:
  add-expense-splitting     ✓ Complete    just now
```

**The line to land:** "The tasks came from the specs. The code would come from the tasks. Nobody is prompting from a blank page, and nobody is asking the agent to guess what 'done' means."

---

## Beat 9: Archive (2 min) — the money beat

```bash
openspec archive add-expense-splitting -y
```

```
Proposal warnings in proposal.md (non-blocking):
  ⚠ Consider splitting changes with more than 10 deltas
Task status: ✓ Complete

Specs to update:
  expense-recording: create
  group-management: create
  settlement: create
Applying changes to openspec/specs/expense-recording/spec.md:
  + 5 added
...
Totals: + 13, ~ 0, - 0, → 0
Specs updated successfully.
Change 'add-expense-splitting' archived as '2026-07-31-add-expense-splitting'.
```

**Point at, side by side:**

- The **added / modified / removed / renamed totals**. Archiving reports itself as a diff.
- `openspec/specs/` **now exists**, populated. `openspec list --specs` shows 13 requirements across three capabilities.
- Open `openspec/specs/settlement/spec.md`: the `## ADDED Requirements` header is gone, replaced by `## Requirements`. The `## Purpose` block was carried across. A `# settlement Specification` title was added.
- The change moved to `changes/archive/2026-07-31-add-expense-splitting/`, date-stamped. Nothing was deleted.

**The line to land:** "That is the whole lifecycle. Propose, review, build, archive. The spec directory is now the living description of the system, and it got there through a reviewed change, not a doc someone forgot to update."

**Non-obvious detail worth a sentence:** the `-y` matters. Without it, archive prompts interactively. There is also `--skip-specs`, for changes that genuinely alter no behavior (tooling, docs, refactors). Validation *rejects* a change with zero deltas unless its `.openspec.yaml` sets `skip_specs: true`, which is a deliberate guard against inventing a fake requirement to satisfy the tool.

---

## Beat 10: The second change, where it earns its keep (4 min)

The first change proved the format. The second proves the value, because now there is existing truth to change.

```
/opsx:propose Expenses can be recorded in any currency. Balances are tracked per currency, and settlement must state what happens when a group's expenses span more than one.
```

Chosen deliberately: it **invalidates an existing requirement**. Splitr's settlement spec says, in as many words:

> ### Requirement: Single currency across a group
> The system SHALL treat all amounts within a group as the same, unspecified currency.

That requirement is now false. It cannot be quietly reinterpreted, because it is written down.

**Open the delta** at `changes/add-multi-currency/specs/settlement/spec.md` and point at three sections:

- `## MODIFIED Requirements` — the requirement restated **in full**, not as a patch fragment.
- `## REMOVED Requirements` — with mandatory `**Reason**` and `**Migration**` fields.
- `## ADDED Requirements` — a new "No currency conversion" requirement, drawing the boundary explicitly.

**Say:** "It rewrote the whole requirement, not a diff hunk. On purpose: at archive time the new text replaces the old wholesale, so the spec is never half-updated."

### The archive-time safety net

Before archiving, break it: change the `MODIFIED` header to `### Requirement: Settlement plans` (plural), so it matches nothing in the main spec.

```bash
openspec validate add-multi-currency --strict
```

```
Change 'add-multi-currency' is valid
```

Validation does not catch it. Now archive:

```bash
openspec archive add-multi-currency -y
```

```
settlement MODIFIED failed for header "### Requirement: Settlement plans" - not found
Aborted. No files were changed.
```

**The line to land:** "Two things worth noticing. `validate` missed it, so validation is not the safety net. **Archive** is: it resolves every MODIFIED header against the real spec, and when one does not match it aborts atomically. No partial write. Your specs are never left half-merged."

Fix the header, then archive for real:

```
Totals: + 4, ~ 5, - 1, → 0
Change 'add-multi-currency' archived as '2026-07-31-add-multi-currency'.
```

### The payoff

```bash
git diff HEAD~1 -- openspec/specs/settlement/spec.md
```

```diff
-The system SHALL compute each group member's net balance as the total they paid
-minus the total of their shares across all expenses in the group.
+The system SHALL compute each group member's net balance **per currency** as the
+total they paid in that currency minus the total of their shares in that currency.
+The system SHALL NOT combine balances across currencies.

-### Requirement: Single currency across a group
-The system SHALL treat all amounts within a group as the same, unspecified currency.
```

**Close on this:** "That diff is the thing you cannot get by prompting an agent feature by feature. It knew what the old rule was, because the old rule was written down where it reads. The requirement that became false was deleted with a stated reason and a migration path, in a reviewable commit. This is a changelog for behavior, and nobody had to remember to write it."

---

## Beat 11: Close (1 min)

Three sentences, then stop talking:

1. "Specs are the source of truth, and they live in git next to the code."
2. "Changes are proposals with an enforced dependency order, so review happens before implementation instead of at PR time."
3. "The agent reads all of it on every turn, so context stops being something I retype."

**"Isn't this a lot of ceremony?"** For a one-file script, yes. For anything a team maintains, you are already writing this down in tickets, Slack, or someone's head. This puts it one directory from the code and makes the agent read it.

**"What stops the spec and code from drifting?"** Nothing automatic, and I would not claim otherwise. What you get is a review point and a diff. Drift becomes visible instead of invisible. `/opsx:sync` exists to reconcile specs against reality, but a human still decides what is true.

**"Does it work with other assistants?"** Yes. `openspec init --tools` lists roughly thirty, including Cursor, Codex, Gemini, Copilot, Cline, and Zed. The `openspec/` directory is identical regardless; only the generated command files differ.

**"What is a store?"** A standalone OpenSpec repo registered on your machine, so specs can live outside the code repo. `openspec store`, and `--store <id>` on most commands. Out of scope today.

---

## Timing sheet

| Beat | Content | Target | Cumulative |
|---|---|---|---|
| 1 | Empty room | 1:00 | 1:00 |
| 2 | `init` | 2:00 | 3:00 |
| 3 | `config.yaml` context | 2:00 | 5:00 |
| 4 | Propose, dependency graph | 2:00 | 7:00 |
| 5 | Instruction envelope | 2:00 | 9:00 |
| 6 | Read delta, break it twice | 4:00 | 13:00 |
| 7 | Read-only tour | 1:00 | 14:00 |
| 8 | "Implementation" | 1:00 | 15:00 |
| 9 | Archive | 2:00 | 17:00 |
| 10 | Second change | 4:00 | 21:00 |
| 11 | Close | 1:00 | 22:00 |

**If running long, cut in this order:** Beat 7, then Beat 5, then Beat 1 (start with `init` already run). **Never cut 6, 9, or 10.** If you have only ten minutes, run 4, 6a, 9, 10 and nothing else.

---

## Appendix A: Alternate demo apps

Each is sized so the spec fits on a few screens, and each has a second change that **modifies** an existing requirement, which is what Beat 10 needs.

### 1. Expense splitter — **SELECTED, and built out in this repo**

`group-management`, `expense-recording`, `settlement`. Second change: multi-currency, which forces the settlement requirement to be restated. Everyone has argued about a dinner bill, and the change is obviously behavioral rather than additive.

### 2. Feature-flag service

`flag-management`, `evaluation`, `targeting`, `audit-log`. Second change: percentage rollouts, restating evaluation with deterministic bucketing. Strongest for a purely developer audience, more jargon.

### 3. Habit tracker CLI

`habit-management`, `streak-tracking`, `reporting`. Second change: vacation mode, restating the streak rule so paused days neither break nor extend a streak. Smallest option, genuinely fiddly edge cases.

### 4. Async standup bot

`standup-collection`, `scheduling`, `digest-delivery`. Second change: per-member timezone windows, turning one team-wide cron into per-member windows. The change is a real architectural shift expressed purely in spec language.

---

## Appendix B: Findings from the verification run

Corrections against what this document originally assumed, all confirmed by execution.

**Wrong in the first draft:**

| Assumed | Actual |
|---|---|
| `/openspec:proposal` | `/opsx:propose` (also `apply`, `archive`, `explore`, `sync`, `update`) |
| `openspec/project.md` | `context:` key in `openspec/config.yaml` |
| `init` scaffolds `specs/` and `changes/` | `init` creates only `config.yaml`; directories appear on use |
| `init` writes `AGENTS.md` / `CLAUDE.md` | Writes `.claude/commands/opsx/*` and `.claude/skills/openspec-*` |
| `openspec diff <change>` | Does not exist. Use `show`, `--json --deltas-only`, or `git diff` |
| `design.md` optional side file | A first-class artifact in the dependency graph, conditional but ordered before `tasks` |
| `openspec archive <change>` | Prompts interactively; needs `-y` to run unattended |

**Right in the first draft:** the delta format (`## ADDED Requirements`, `### Requirement:`, `#### Scenario:` with WHEN/THEN), SHALL wording, the propose → review → implement → archive lifecycle, and the claim that archive merges deltas into `specs/`.

**Validation behavior, tested directly:**

| Defect | `validate` | `validate --strict` | `archive` |
|---|---|---|---|
| Requirement with zero scenarios | **caught** | **caught** | n/a |
| `###` instead of `####` on a scenario | missed | missed | n/a |
| `MODIFIED` header matching no existing requirement | missed | missed | **caught, aborts atomically** |
| `REMOVED` without `**Reason**` / `**Migration**` | missed | missed | not caught |

The `###`-instead-of-`####` case is the dangerous one: it silently discarded **all four** scenarios of the affected requirement while reporting the change as valid. Read `openspec show <change> --json --deltas-only` before trusting a large delta.

**Still unverified, do not assert on stage:**

- `/opsx:apply` end to end (we wrote no code).
- `/opsx:sync` and `/opsx:explore`.
- `openspec store` and `--store`, beyond reading the help text.
- Whether `--strict` differs from plain `validate` at all. In every case tested here, the two behaved identically.
