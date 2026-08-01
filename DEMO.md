# OpenSpec: Lifecycle and Command Walkthrough

**Verified against `@fission-ai/openspec` v1.7.0 on 2026-07-31.** Every command, output, and path below was executed in this repository. Where the tool surprised me, the surprise is written down rather than smoothed over. Where I have not verified something, it is listed as unverified rather than asserted.

This document does two jobs:

- **Part 1 to 4** walk the lifecycle stage by stage, showing which commands belong at each stage and why.
- **Part 5 onward** is reference: every command, when to reach for it, and the sharp edges.

The demo app is **Splitr**, a shared-expense tracker. We write **no application code**. Only specifications. Every time someone asks "but where's the app?", that is the point being made: the spec is the artifact under review.

---

## Part 1: The model

Two directories, one idea.

> `specs/` is **what is true today**. `changes/` is **what someone is proposing**. A proposal is a *delta* against the specs, not a rewrite of them. You review the delta like a pull request. When the work is done you archive it, the delta folds into `specs/`, and that becomes the new truth.

```
openspec/
├── config.yaml             # schema choice, project context, per-artifact rules
├── specs/                  # CURRENT truth: what IS built
│   └── <capability>/spec.md
└── changes/                # PROPOSED: what SHOULD change
    ├── <change-id>/
    │   ├── .openspec.yaml  # schema, created date, optional skip_specs
    │   ├── proposal.md     # why & what
    │   ├── specs/<capability>/spec.md   # the DELTA
    │   ├── design.md       # how (conditional)
    │   └── tasks.md        # implementation checklist
    └── archive/
        └── 2026-07-31-<change-id>/      # date-stamped on archive
```

### The five stages

| Stage | What exists | Commands that matter here |
|---|---|---|
| **0. Setup** | nothing | `init`, then fill `config.yaml` |
| **1. Proposed** | `proposal.md` only | `new change`, `status`, `instructions` |
| **2. Specified** | delta specs written | `validate`, `show --deltas-only` |
| **3. In progress** | design + tasks, boxes being ticked | `list`, `status`, `/opsx:apply` |
| **4. Archived** | folded into `specs/` | `archive`, `list --specs` |

A change moves forward only. There is no "unarchive".

---

## Part 2: This repo is the fixture

Rather than a single happy path, this repo is parked with a change at **every** stage, so each command has something real to show. Run this first:

```bash
openspec list
```

```
Changes:
  refactor-split-engine      5/15 tasks    <relative time>
  add-settlement-history     5/17 tasks    <relative time>
  add-expense-editing        No tasks      <relative time>
```

```bash
openspec list --specs
```

```
Specs:
  expense-recording     requirements 7
  group-management      requirements 5
  settlement            requirements 4
```

| Change | Stage | Why it's here |
|---|---|---|
| `add-expense-editing` | **Proposed** | Proposal only. Deliberately fails validation. |
| `add-settlement-history` | **In progress** | All four artifacts, 5/17 tasks. Has a `MODIFIED` delta. |
| `refactor-split-engine` | **In progress, spec-exempt** | `skip_specs: true`. Zero deltas, still valid. |
| `2026-07-31-add-expense-splitting` | **Archived** | Created all three capabilities. |
| `2026-07-31-add-multi-currency` | **Archived** | `MODIFIED` + `REMOVED` against existing truth. |

The git history is a second fixture, one commit per stage:

```bash
git log --oneline
```

---

## Part 3: Walking the lifecycle

### Stage 0: Setup

```bash
openspec init --tools claude
```

```
OpenSpec Setup Complete
Created: Claude Code
6 skills and 6 commands in .claude/
Config: openspec/config.yaml (schema: spec-driven)
```

**`init` does not scaffold `specs/` or `changes/`.** It creates exactly one file under `openspec/`: `config.yaml`. The directories appear when they have something to hold. If you promised the audience a tree, you will be standing in front of a single YAML file.

`--tools` accepts `all`, `none`, or a comma-separated list of 33 assistants (cursor, codex, gemini, github-copilot, cline, kiro, zcode, and so on), plus `windsurf` as an accepted alias for `devin`. The `openspec/` directory is identical whichever you pick; only the generated command files differ.

**Then teach it the project once.** There is **no `project.md`** — context lives in `openspec/config.yaml`:

```yaml
schema: spec-driven

context: |
  Splitr is a shared-expense tracker...
  Money is stored as integer minor units (cents), never floats.

# rules:          per-artifact constraints, e.g. "always include a Non-goals section"
# operations:     per-operation guidance for apply and archive
```

**When to revisit:** whenever the agent gets a convention wrong twice. That is a `config.yaml` gap, not a prompting problem.

### Stage 1: Proposed

```bash
openspec new change add-expense-editing
openspec status --change add-expense-editing
```

```
Progress: 1/4 artifacts complete

[x] proposal
[ ] specs
[ ] design
[-] tasks (blocked by: specs, design)
```

Markers: `[ ]` ready, `[-]` blocked, `[x]` done, `[~]` skipped.

**This is the most under-appreciated thing OpenSpec does.** The workflow is a dependency graph, not a checklist. You physically cannot write the task list before writing down what the thing should do. That ordering is the whole argument for spec-driven development, and it is enforced rather than suggested.

Now validate a proposal-only change:

```bash
openspec validate add-expense-editing --strict
```

```
✗ [ERROR] file: Change must have at least one delta. No deltas found.
  ... If this change intentionally modifies no specs (pure refactor, tooling,
  docs), set "skip_specs: true" in the change's .openspec.yaml instead.
```

**A change at Stage 1 is supposed to fail validation.** Failing here means "not finished", not "broken". Do not wire `openspec validate --all` into CI expecting green while changes are in flight.

Before writing any artifact, see exactly what the agent will receive:

```bash
openspec instructions proposal --change add-expense-editing
```

This prints `<task>`, `<project_context>`, `<output>`, `<instruction>`, `<template>`, `<unlocks>`. The `<project_context>` block contains **the exact text from `config.yaml`**, with an explicit note that it is guidance, not content. That is the proof that Stage 0 mattered.

### Stage 2: Specified

Write the delta. The format is strict:

```markdown
## ADDED Requirements

### Requirement: Net balance per member

The system SHALL compute each group member's net balance as the total they paid
minus the total of their shares across all expenses in the group.

#### Scenario: Balances always sum to zero

- **WHEN** balances are computed for any group
- **THEN** the sum of all member balances is exactly 0
```

Four delta operations: `## ADDED`, `## MODIFIED`, `## REMOVED`, `## RENAMED Requirements`.

- **ADDED** — new behavior. New capabilities should open with a `## Purpose` section, which archive copies into the main spec it creates. Omit it and archive silently writes `TBD - created by archiving change <id>. Update Purpose after archive.` instead. Nothing warns you: see Part 7.
- **MODIFIED** — copy the **entire** existing requirement block and edit it. Partial content loses detail at archive time.
- **REMOVED** — requires `**Reason**` and `**Migration**`.
- **RENAMED** — `FROM:` / `TO:` format.

```bash
openspec validate add-settlement-history --strict
openspec show add-settlement-history --json --deltas-only
```

**Use `--deltas-only` before trusting any large delta.** Part 7 explains why this is not optional.

### Stage 3: In progress

```bash
openspec list
```

```
add-settlement-history     5/17 tasks    <relative time>
```

Note the split between two notions of "done":

```bash
openspec status --change add-settlement-history
```

```
Progress: 4/4 artifacts complete
All artifacts complete!
```

**`status` reports file existence. `list` reports ticked checkboxes.** A change can be "all artifacts complete" with 5 of 17 tasks done. Neither is wrong; they answer different questions. `status` asks "is it planned?", `list` asks "is it built?".

#### The spec-exempt path

Some changes genuinely alter no behavior. `refactor-split-engine` sets `skip_specs: true` in its `.openspec.yaml`:

```bash
openspec status --change refactor-split-engine
```

```
Progress: 3/3 artifacts complete (1 skipped)

[x] proposal
[~] specs (skipped: change declares skip_specs)
[x] design
[x] tasks
```

```bash
openspec validate refactor-split-engine --strict
# Change 'refactor-split-engine' is valid
```

Zero deltas, and it passes. The denominator drops to 3, and `tasks` is now blocked by `design` alone.

**When to use `skip_specs`:** pure refactors, tooling, docs. **When not to:** anything where behavior changes, even slightly. The tool's own instruction is blunt about it: *"Do not invent a requirement just to satisfy validation."* Reaching for `skip_specs` because writing the spec is annoying is how the spec directory stops being true.

### Stage 4: Archived

```bash
openspec archive add-expense-splitting -y
```

```
Task status: ✓ Complete

Specs to update:
  expense-recording: create
  group-management: create
  settlement: create
Applying changes to openspec/specs/settlement/spec.md:
  + 4 added
Totals: + 13, ~ 0, - 0, → 0
Specs updated successfully.
Change 'add-expense-splitting' archived as '2026-07-31-add-expense-splitting'.
```

What archive does:

1. Resolves every `MODIFIED` / `REMOVED` / `RENAMED` header against the real spec.
2. Applies the deltas, dropping the `## ADDED Requirements` header for a plain `## Requirements`.
3. Carries `## Purpose` into new specs and adds a `# <capability> Specification` title.
4. Moves the change to `changes/archive/<date>-<id>/`. Nothing is deleted.

**Flags:** `-y` skips prompts (it is interactive otherwise). `--skip-specs` archives without touching specs. `--no-validate` exists and should be treated as a mistake.

**Archive is the real safety net, not validate.** See Part 7.

The payoff is a behavior changelog you did not have to write:

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

---

## Part 4: A 20-minute live demo path

If you are presenting rather than reading, run this subset:

| # | Do | Say |
|---|---|---|
| 1 | `ls -la` on empty dir | "Nothing here yet. We decide what to build before there is anything to argue about." |
| 2 | `openspec init --tools claude` | "One command. Note it creates one config file, not a tree." |
| 3 | Show `config.yaml` `context:` | "Written once. Read on every artifact." |
| 4 | `openspec status --change ...` | "A dependency graph. You cannot write tasks before specs." |
| 5 | `openspec instructions proposal --change ...` | "Here is my context, injected. That is the proof." |
| 6 | Read a delta, then **break it twice** | The single best beat. See Part 7. |
| 7 | `openspec list` / `list --specs` | "Plain markdown in git. The CLI is a convenience, not a database." |
| 8 | Tick tasks by hand | "Tasks came from specs. Code would come from tasks." |
| 9 | `openspec archive <id> -y` | "Propose, review, build, archive. Now it's truth." |
| 10 | Second change, `MODIFIED` + `git diff` | "It knew the old rule, because the old rule was written down." |

**Never cut 6, 9, or 10.** With only ten minutes, run 4, 6, 9, 10 and nothing else.

---

## Part 5: Complete command reference

### Core lifecycle

| Command | When to use it |
|---|---|
| `openspec init [--tools <list>]` | Once per repo. `--tools all\|none\|cursor,codex,...` |
| `openspec new change <name>` | Start a change. `--description`, `--goal`, `--schema` |
| `openspec status --change <id>` | "What artifact do I write next, and what is blocking it?" The single most useful command mid-change. `--json` for scripting |
| `openspec instructions <artifact> --change <id>` | Before writing an artifact by hand, to see the template and injected context |
| `openspec validate <id> [--strict]` | Before asking anyone to review. `--all`, `--changes`, `--specs`, `--json` |
| `openspec archive <id> -y` | Tasks are done and the change should become truth. `--skip-specs`, `--no-validate` |

### Reading

| Command | When to use it |
|---|---|
| `openspec list` | Active changes with task progress |
| `openspec list --specs` | Current capabilities and requirement counts |
| `openspec show <id>` | A change or spec, auto-detected. `--type change\|spec` to disambiguate |
| `openspec show <id> --json --deltas-only` | **Verify what the parser actually saw.** Not optional for large deltas |
| `openspec show <spec> --json --requirements` | Requirements without scenario bodies |
| `openspec view` | Interactive dashboard. Know you are entering a TUI before typing it on stage |

**There is no `openspec diff`.** Use `show`, `--deltas-only`, or `git diff` after archiving.

### Configuration and schemas

| Command | When to use it |
|---|---|
| `openspec config list \| get \| set \| unset \| path \| edit` | **Global** config at `~/.config/openspec/config.json`, not per-repo |
| `openspec config profile [preset]` | Switch which workflows are generated |
| `openspec schemas` | List workflow schemas. `spec-driven` is the default: proposal → specs → design → tasks |
| `openspec templates [--schema <name>]` | Resolve template file paths for a schema |
| `openspec schema which\|validate\|fork\|init` | **Experimental.** Fork the schema to customize artifacts |
| `openspec update [--force]` | Refresh generated instruction files after upgrading the CLI |

### Multi-repo and housekeeping

| Command | When to use it |
|---|---|
| `openspec store setup\|register\|list\|doctor\|unregister\|remove` | Specs living outside the code repo. Most commands then take `--store <id>` |
| `openspec context [--json]` | Print the working context for the resolved root |
| `openspec doctor` | Check relationship health of the resolved root |
| `openspec workset create\|list\|open\|remove` | Purely local named working views. Never shared |
| `openspec completion install [shell]` | Shell tab-completion |
| `openspec feedback <message>` | File feedback upstream |

### Deprecated

`openspec spec ...` and `openspec change ...` still work but print:

> Warning: The "openspec spec ..." commands are deprecated. Prefer verb-first commands.

Use `openspec show` / `openspec validate --specs` instead. Note `openspec change list` is deprecated in favour of `openspec list`, but **`openspec new change`** is current — the deprecation is on the `change` noun-first group, not the `new` verb.

---

## Part 6: The slash commands

`init --tools claude` generates six, in `.claude/commands/opsx/` with backing skills in `.claude/skills/`. They are `/opsx:*`, **not** `/openspec:*`. Restart your IDE after `init` or they will not appear.

| Command | Stage | What it does |
|---|---|---|
| `/opsx:explore` | before Stage 1 | Thinking partner. Explicitly forbidden from writing code. May create artifacts if asked |
| `/opsx:propose` | Stage 1 → 2 | Creates the change and generates **all** artifacts in dependency order |
| `/opsx:apply` | Stage 3 | Works the task list |
| `/opsx:update` | Stage 2 or 3 | Revises existing artifacts and keeps them coherent. Never edits code |
| `/opsx:sync` | Stage 3 | Folds delta specs into main specs **without archiving**. Agent-driven, so it can merge intelligently (add one scenario rather than replace a whole requirement) |
| `/opsx:archive` | Stage 4 | Wraps `openspec archive` |

**`/opsx:sync` versus `archive`:** sync updates `specs/` while the change stays active. Use it when a long-running change should land its spec early. Archive is the terminal move. Sync is the escape hatch, and like most escape hatches it is where specs and changes drift apart if you lean on it.

The skills are plain markdown you can read and edit. `.claude/skills/openspec-propose/SKILL.md` is the one worth reading first: it is the whole propose algorithm in about 120 lines.

---

## Part 7: Sharp edges, all verified

### Validation catches less than you think

| Defect | `validate` | `--strict` | `archive` |
|---|---|---|---|
| Requirement with zero scenarios | **caught** | **caught** | n/a |
| Change with zero deltas and no `skip_specs` | **caught** | **caught** | n/a |
| `###` instead of `####` on a scenario | missed | missed | n/a |
| `MODIFIED` header matching no existing requirement | missed | missed | **caught, aborts atomically** |
| `REMOVED` without `**Reason**` / `**Migration**` | missed | missed | not caught |
| New capability with `## Purpose` under 50 chars | missed | missed | not caught |
| New capability with no `## Purpose` at all | missed | missed | writes a `TBD` placeholder |

**`--strict` did not differ from plain `validate` in any of the seven cases above.** Treat it as decoration until you find a case where it bites.

### The silent one, in detail

Change one `#### Scenario:` to `### Scenario:` and validation still reports the change as valid, exit 0. But every scenario under that requirement disappears from the parsed delta, because the `###` line ends the requirement block early. Not one lost scenario: all of them.

Measured twice in this repo. On the `settlement` delta of `add-expense-splitting`, one demotion took a requirement from **4 scenarios to 0**. On `add-settlement-history`, one demotion took the whole change from **16 scenarios to 7** — nine gone, still reported valid.

```bash
openspec show <change> --json --deltas-only   # the only way to see it
```

The tool's own instructions warn about this in exactly these words: *"Scenarios MUST use exactly 4 hashtags. Using 3 hashtags or bullets will fail silently."*

**As a demo beat this is gold.** Everyone in the room has been sold a tool that only worked in the happy path. Showing the sharp edge and the mitigation buys more credibility than a clean run does.

### Archive is the real safety net

Break a `MODIFIED` header so it matches nothing:

```
openspec validate add-multi-currency --strict
Change 'add-multi-currency' is valid          ← missed

openspec archive add-multi-currency -y
settlement MODIFIED failed for header "### Requirement: Settlement plans" - not found
Aborted. No files were changed.               ← caught, atomically
```

No partial write. Your specs are never left half-merged. Validation is a linter; archive is the gate.

### Other things that will bite

- **`openspec update` mutates global config.** It flipped `profile: core (default)` to `profile: custom (explicit)` in `~/.config/openspec/config.json`. Same six workflows, no repo files touched, but it is a global side effect from a repo-local-looking command.
- **`archive` prompts without `-y`.** Fine interactively, hangs in CI.
- **Archived changes are date-stamped** (`2026-07-31-add-expense-splitting`). Two changes archived the same day keep distinct names only because the change id differs.
- **A proposal warning at >10 deltas** is non-blocking: *"Consider splitting changes with more than 10 deltas."* Good advice, easy to ignore, and ignoring it is how a change becomes unreviewable.
- **The tool's own instruction text is wrong about `Purpose`.** `openspec instructions specs` claims a Purpose under 50 characters is reported by `--strict`. It is not, at either validation level. Trust the table above over the instruction text.
- **`.DS_Store` inside `.git/`.** Not an OpenSpec issue, but it bit this repo twice: browsing or moving the folder in Finder writes `.git/refs/.DS_Store`, which git parses as a malformed ref (`badRefName`) and which `.gitignore` cannot prevent. If `git fsck` starts complaining, run `find .git -name '.DS_Store' -delete`.

---

## Part 8: Unverified

Do not assert these on stage. They are read from help text and skill files, not exercised.

- `/opsx:apply` end to end. We wrote no code, so the implementation loop is untested here.
- `/opsx:sync` and `/opsx:explore` behavior in practice.
- `openspec store` and `--store` beyond reading help output.
- `openspec schema fork` / `init`, and whether a custom schema round-trips.
- `openspec workset`, `openspec completion`, `openspec feedback`.
- `openspec view`, the interactive dashboard.
- `RENAMED Requirements`. It is documented and I have not exercised it; every other delta operation here has been.

---

## Appendix: Alternate demo apps

Each is sized so the spec fits on a few screens, and each has a second change that **modifies** an existing requirement, which is what the Stage 4 payoff needs.

1. **Expense splitter** — *selected, built out in this repo.* `group-management`, `expense-recording`, `settlement`. Second change: multi-currency.
2. **Feature-flag service** — `flag-management`, `evaluation`, `targeting`, `audit-log`. Second change: percentage rollouts, restating evaluation with deterministic bucketing. Strongest for a purely developer audience, more jargon.
3. **Habit tracker CLI** — `habit-management`, `streak-tracking`, `reporting`. Second change: vacation mode, so paused days neither break nor extend a streak. Smallest option, genuinely fiddly edge cases.
4. **Async standup bot** — `standup-collection`, `scheduling`, `digest-delivery`. Second change: per-member timezone windows. A real architectural shift expressed purely in spec language.
