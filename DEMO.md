# OpenSpec Demo: Run of Show

**Goal of this demo:** show how OpenSpec turns "vibe prompting a coding agent" into a reviewable, versioned, spec-first workflow. The audience should leave able to answer three questions:

1. Where does the *current truth* about the system live? (`openspec/specs/`)
2. Where does a *proposed change* live before it is real? (`openspec/changes/<change-id>/`)
3. What makes a change "real"? (implement, then `archive`, which folds the delta into the specs)

**Important scope note:** we are **not** writing any application code in this demo. We only write specifications and walk the OpenSpec lifecycle. Every time someone asks "but where's the app?", that is the point being made: the spec is the artifact under review, not the code.

**Demo app:** **Splitr**, an expense splitter (Splitwise-lite). Three capabilities: `group-management`, `expense-recording`, `settlement`. The demo specs two changes against it:

1. **`add-expense-splitting`** — create a group, record an expense, split it evenly, see who owes whom.
2. **`add-multi-currency`** — expenses can be recorded in any currency, which **modifies** the existing settlement requirement rather than merely adding to it.

Alternates, if this audience is wrong for it, are in [Appendix A](#appendix-a-candidate-demo-apps).

**Target runtime:** 15 to 20 minutes, plus 5 for questions.

---

## Pre-demo checklist

Do all of this the day before, not live.

- [ ] Node.js installed (`node --version`).
- [ ] Decide install path: `npm i -g @fission-ai/openspec` or run via `npx`. Confirm which one you will type on stage and use it consistently.
- [ ] Run `openspec --help` and **reconcile every command in this file against the actual help output.** Command names and flags below are written from memory of the tool's workflow and may have drifted. Fix this document, then rehearse.
- [ ] Claude Code (or your assistant of choice) open in this directory, with the OpenSpec slash commands available after `init`.
- [ ] Terminal font size bumped. Two panes: terminal on the left, file tree / editor on the right.
- [ ] A throwaway copy of this repo so you can reset with `git checkout .` between rehearsals.
- [ ] Rehearse the deliberate-failure beat (Beat 6). It is the best beat and the easiest to fumble.

**Reset command between runs:**

```bash
rm -rf openspec && git checkout . 2>/dev/null; ls -la
```

---

## The one-slide mental model (say this before touching the keyboard)

> Two directories. `specs/` is **what is true today**. `changes/` is **what someone is proposing**. A proposal is a *delta* against the specs, not a rewrite of them. You review the delta like a pull request. When the work is done, you archive the change, and the delta gets folded into `specs/`, which becomes the new truth.

Draw it on the whiteboard if you have one:

```
openspec/
├── project.md              # conventions, stack, house rules
├── specs/                  # CURRENT truth: what IS built
│   └── <capability>/spec.md
└── changes/                # PROPOSED: what SHOULD change
    └── <change-id>/
        ├── proposal.md     # why, what, impact
        ├── tasks.md        # implementation checklist
        ├── design.md       # (optional) for non-obvious technical calls
        └── specs/<capability>/spec.md   # the DELTA
```

---

## Beat 1: The empty room (1 min)

**Say:** "This is a completely empty directory. No code, no README, nothing."

**Run:**

```bash
ls -la
```

**Point at:** nothing. That is the point. We are going to establish what we are building before anything exists to argue about.

**Landmine:** do not let this beat run long. It is a setup beat.

---

## Beat 2: `init` (1 min)

**Say:** "One command bootstraps the workflow and wires up my AI assistant."

**Run:**

```bash
openspec init
```

**Point at:**

- The generated `openspec/` tree.
- The assistant instruction file it writes or updates (`AGENTS.md` / `CLAUDE.md`). Open it for five seconds and say: "This is the part that matters. The rules of the workflow are now in the agent's context on every single turn. I am not re-explaining my process in each prompt."

**Landmine:** if `init` prompts for which assistants to configure, know your answer in advance.

---

## Beat 3: Teach it the project (2 min)

**Say:** "Before any feature, I tell it about the project once."

**Open and fill:** `openspec/project.md` with the chosen app's purpose, stack, and conventions. Keep it to ten lines on screen. Read two lines aloud, not all of them.

**The line to land:** "Everything in here is context the agent stops guessing about. This is the file that stops it from inventing a Redis dependency at 2am."

---

## Beat 4: The first proposal (3 min)

**Say:** "Now I ask for a feature. Watch what I get back: not code."

**Run (in Claude Code):**

```
/openspec:proposal Users can create a group, add members, record an expense paid by one member, and split it evenly across the group. Show each member's net balance.
```

Let it work. While it runs, narrate what it is doing: reading `project.md`, checking existing specs (there are none yet), and drafting a change folder.

**Then run:**

```bash
openspec list
```

**Point at:** the change ID now sitting in `changes/`.

**Landmine:** the agent may produce a change ID different from what you rehearsed. Do not fight it. Copy the actual ID into the following commands.

---

## Beat 5: Read the proposal out loud (3 min)

This is the heart of the demo. Slow down here.

**Open, in this order:**

1. `changes/<id>/proposal.md` — Why / What Changes / Impact. Say: "This is the part a product person can review."
2. `changes/<id>/specs/<capability>/spec.md` — the delta. Say: "This is the part an engineer reviews."
3. `changes/<id>/tasks.md` — the checklist. Say: "This is the part the agent will work through."

**Point at the delta format specifically.** Read one requirement and one scenario aloud:

```markdown
## ADDED Requirements

### Requirement: <Name>
The system SHALL <behavior>.

#### Scenario: <Name>
- **WHEN** <condition>
- **THEN** <observable outcome>
```

**The line to land:** "`SHALL` plus a scenario is the whole trick. Every requirement carries at least one concrete, checkable scenario. That is what makes this reviewable instead of a wishlist, and it is what the agent tests against later."

Also point at the header: it says `## ADDED Requirements`. Say: "The file is a *diff*, not a document. That is why archiving can merge it mechanically."

---

## Beat 6: Break it on purpose (2 min)

**Say:** "How do I know the agent didn't just write pretty prose?"

**Run:**

```bash
openspec validate <change-id> --strict
```

It passes. Now **delete the `#### Scenario:` block** from one requirement and run it again.

**Point at:** the validation error naming the requirement with no scenario.

**Say:** "A requirement with no scenario is an opinion. The tooling rejects it. This is the guardrail that stops spec rot."

Put the scenario back and re-run to green before moving on.

**Landmine:** confirm the exact `validate` invocation and its failure output during rehearsal. If strict validation does not catch this specific omission in your installed version, find one it *does* catch and break that instead. Do not improvise this live.

---

## Beat 7: The read-only tour (1 min)

Quick fire, roughly fifteen seconds each:

```bash
openspec list
openspec list --specs
openspec show <change-id>
openspec diff <change-id>
```

**The line to land:** "Everything is plain markdown in git. The CLI is a convenience, not a database. If OpenSpec disappeared tomorrow, I would still have my specs."

---

## Beat 8: "Implementation" (1 min)

**Say:** "Normally I would run `/openspec:apply` here and the agent works the task list. We are skipping the code today, because the code is the least interesting part of this story."

**Do:** tick the boxes in `tasks.md` by hand, on screen. It takes five seconds and it visually completes the loop.

**The line to land:** "The tasks came from the spec. The code would come from the tasks. Nobody is prompting from a blank page."

---

## Beat 9: Archive (2 min) — the money beat

**Say:** "The work is done. Now I promote the proposal to truth."

**Run:**

```bash
openspec archive <change-id>
openspec list --specs
```

**Point at, side by side:**

- `changes/` no longer holds an active change (it moves to the archive).
- `specs/<capability>/spec.md` **now exists**, and the `## ADDED Requirements` header is gone. The requirements are just... requirements now.

**The line to land:** "That is the whole lifecycle. Propose, review, build, archive. The spec directory is now the living description of the system, and it got there through a reviewed change, not a well-intentioned doc update someone forgot."

---

## Beat 10: The second change, where it earns its keep (3 min)

The first change proves the format. The second change proves the *value*, because now there is existing truth to change.

**Run (in Claude Code):**

```
/openspec:proposal Expenses can be recorded in any currency. Balances are tracked per currency, and settlement must state what happens when a group's expenses span more than one.
```

This is chosen deliberately: it **changes an existing requirement** rather than only adding new ones. The settlement rule ("minimize the number of transactions needed to settle up") no longer means anything until you say what a balance across two currencies even is.

**Point at:** the delta now containing a `## MODIFIED Requirements` section, restating the settlement requirement **in full** with the new behavior, not as a patch fragment.

**Say:** "Notice it rewrote the whole requirement, not a diff hunk. That is on purpose. When this archives, the new text replaces the old text wholesale, and there is never a moment where the spec is half-updated."

**Then run:**

```bash
openspec diff <change-id>
```

**The line to land:** "This is the thing you cannot get by prompting an agent feature by feature. It knew what the old rule was, because the old rule was written down in a place it reads. The proposal explains what is changing and why, before a line of code moves."

**Optional flourish, if the room is engaged:** ask the audience for a third feature on the spot and generate a proposal live. It is low risk, since nothing has to compile.

---

## Beat 11: Close (1 min)

Three sentences, then stop talking:

1. "Specs are the source of truth, and they live in git next to the code."
2. "Changes are proposals with a lifecycle, so review happens before implementation instead of at PR time."
3. "The agent reads all of it on every turn, so context stops being something I retype."

**Anticipated question: "Isn't this just a lot of ceremony?"**
Answer: "For a one-file script, yes. For anything a team maintains, you are already writing this down somewhere: in tickets, in Slack, in someone's head. This puts it one directory away from the code and makes the agent read it."

**Anticipated question: "What if the spec and the code drift?"**
Answer: "Nothing prevents that automatically. What OpenSpec gives you is a review point and a diff. Drift becomes visible instead of invisible. That is an honest limitation, not a solved problem."

**Anticipated question: "Does it work with agents other than Claude Code?"**
Answer: check the tool's supported-assistant list during prep and answer specifically. Do not guess on stage.

---

## Timing sheet

| Beat | Content | Target | Cumulative |
|---|---|---|---|
| 1 | Empty room | 1:00 | 1:00 |
| 2 | `init` | 1:00 | 2:00 |
| 3 | `project.md` | 2:00 | 4:00 |
| 4 | First proposal | 3:00 | 7:00 |
| 5 | Read the delta | 3:00 | 10:00 |
| 6 | Break validation | 2:00 | 12:00 |
| 7 | Read-only tour | 1:00 | 13:00 |
| 8 | "Implementation" | 1:00 | 14:00 |
| 9 | Archive | 2:00 | 16:00 |
| 10 | Second change | 3:00 | 19:00 |
| 11 | Close | 1:00 | 20:00 |

**If you are running long, cut in this order:** Beat 7, then Beat 3 (pre-fill `project.md` instead), then Beat 8. **Never cut Beats 5, 9, or 10.**

---

## Appendix A: Candidate demo apps

Each candidate is sized so the whole spec fits on a few screens, and each ships with a deliberate second change that **modifies** an existing requirement, which is what Beat 10 needs.

### 1. Expense splitter (Splitwise-lite) — **SELECTED**

- **Capabilities:** `group-management`, `expense-recording`, `settlement`
- **First change:** record an expense against a group and split it evenly.
- **Second change (modifies):** multi-currency support. This forces the settlement requirement to be restated: balances now carry a currency, and the "minimum transactions to settle" rule has to say what happens across currencies.
- **Why it demos well:** everyone in the room has argued about a dinner bill. The multi-currency change is an obviously *behavioral* change to an existing rule, not an addition, so `## MODIFIED Requirements` lands without explanation.

### 2. Feature-flag service

- **Capabilities:** `flag-management`, `evaluation`, `targeting`, `audit-log`
- **First change:** boolean flags with per-environment on/off evaluation.
- **Second change (modifies):** percentage rollouts. The evaluation requirement must be restated to include deterministic bucketing by user ID.
- **Why it demos well:** a developer audience already thinks in terms of specs here, and "what happens on a cache miss" style scenarios write themselves. Slightly more jargon than option 1.

### 3. Habit tracker CLI

- **Capabilities:** `habit-management`, `streak-tracking`, `reporting`
- **First change:** define habits, check in daily, compute a streak.
- **Second change (modifies):** pausing a habit (vacation mode). The streak requirement must be restated: paused days neither break nor extend a streak.
- **Why it demos well:** the smallest of the four, and the streak rule has genuinely fiddly edge cases (timezones, backfilled check-ins), so the scenarios are visibly earning their place.

### 4. Async standup bot

- **Capabilities:** `standup-collection`, `scheduling`, `digest-delivery`
- **First change:** collect three answers from each team member and post a digest at a fixed time.
- **Second change (modifies):** per-member timezone windows. The scheduling requirement is restated: one team-wide cron becomes a per-member window with a digest cutoff.
- **Why it demos well:** the change is a real architectural shift expressed purely in spec language, which sells the "review before you build" argument hard.

---

## Appendix B: Corrections log

Anything below that turned out to be wrong during rehearsal, fix in place above and note here.

- [ ] Verified every CLI command against `openspec --help`.
- [ ] Verified the slash command names after `openspec init`.
- [ ] Verified that Beat 6's deliberate break actually fails validation.
