---
name: implement-batch
description: "Implement a whole (n/N) ticket batch from /to-tickets, one fresh background subagent per ticket."
disable-model-invocation: true
---

# Implement Batch

A **batch** is the set of `(n/N)` tickets `/to-tickets` made from one session: `Granary rework (1/3)`, `Granary rework (2/3)`, `Granary rework: full tests (3/3)`. Run the whole batch so the user never has to `/clear` and `/implement` ticket by ticket. Each ticket goes to its own fresh **implementer** subagent; you are the **orchestrator**. You pick the batch, schedule the **frontier**, relay blocking questions and report. The implementers write the code.

The issue tracker should have been provided to you. If not, tell the user to run `/setup-matt-pocock-skills`.

Project specifics (test commands, build steps, which files are risky to touch) live in the project's agent docs (`CLAUDE.md`, `AGENTS.md`) and in the ticket text. The implementers read those themselves. Talk to them through **context pointers** (ticket numbers, file paths); they fetch the ticket text themselves.

## 1. Pick the batch

- **`#n`**: any ticket of the batch. Fetch it, then find its siblings: the tickets whose titles share its leading text (the **stem**) and end in `(k/N)` with the same N. Search closed tickets too.
- **`next`** (or no argument): list the open `ready-for-agent` tickets whose title ends in `(k/N)`, group them by stem and N, and take the group holding the lowest issue number. Then search closed tickets for the rest of that batch.

You have the batch when you hold exactly one ticket for every k from 1 to N. When a k is missing or two tickets claim it, ask the user. Closed tickets count as done, so a partly done batch resumes at its open tickets; a batch with every ticket closed has nothing left to run.

## 2. Read the graph

For every open ticket, read:

- **Blocking edges**: the tracker's native dependencies (GitHub issue dependencies, Linear relations), or the ticket's "Blocked by" section. A blocker outside the batch counts too.
- **Parallel-safe**: the ticket body's `Parallel-safe:` line, as `/to-tickets` writes it: `**Parallel-safe:** yes (reason)`. Markdown emphasis and the reason after the value don't matter. A value other than `yes`, or no such line, means not parallel-safe.

State the batch to the user in a few lines: each ticket's number, title, done or open, and its blockers.

## 3. Work the frontier

The frontier is every open ticket whose blockers are all closed. Start a frontier ticket when it fits beside the tickets already running:

- Nothing running: start it.
- Something running: start it only if, after starting, at most one running ticket is not parallel-safe.

A blocked implementer still counts as running: it holds its place until it finishes.

Start each implementer as a **background** subagent (general-purpose in Claude Code) with a **fresh context window**, never a fork of your conversation, using the prompt in step 4. Print one line as it starts: `▶ #312 (2/4) started`.

Then wait for completion notifications rather than polling. Every report, including one from a resumed implementer, goes through these steps:

1. A report ending in `BLOCKED:` is a blocked ticket, not a failure: print `⏸ #312 blocked: <question>` and go to step 5.
2. Otherwise check the outcome against the tracker and git: the ticket is closed and its commit exists. Trust that over the report's wording.
3. Print one line: `✔ #312 committed abc1234, 2 decisions` or `✘ #312 failed: <reason>`, and handle a failure (step 6).
4. Recompute the frontier and start whatever now fits.

The batch ends when every ticket is closed, or when nothing is running and nothing more may start.

## 4. The implementer prompt

`implement` is user-invoked, so a subagent can't reach it through the Skill tool. Hand it the file instead: `implement/SKILL.md` is a sibling of this skill's own directory (`../implement/SKILL.md` from here); resolve that to an absolute path, and stop and tell the user if the file isn't there. Fill in this prompt:

<implementer-prompt>

Implement ticket #<n> (<title>). Read `<absolute path to implement/SKILL.md>` and follow it as if the user had run `/implement #<n>`. Call the Skill tool for the model-invoked skills it names (`tdd`, `code-review`).

Other implementers may be working in this same working tree at the same time, and it may hold the user's own uncommitted changes. The git index is shared too, so commit only the paths this ticket changed, each by name: `git add <paths>`, then `git commit -m "..." -- <paths>`. Leave every other change in the tree as you found it; `git add -A`, `git add .` and a bare `git commit` would sweep it into your commit.

Run the tests the ticket's acceptance criteria name. On "Related tests pass", run only the tests this ticket touched, of every test kind, even where the project docs say to run a suite in full at the end of each issue: only the full-tests ticket, `(N/N)`, runs full suites.

The user is not watching this run. For its duration, this rule replaces every "ask the user" rule in the skills and the project's docs:

- **Non-blocking question**: decide it yourself. Record it as question / options / choice / why in a "Decisions" section of your final report, and as a comment on #<n>.
- **Blocking question**: no reasonable default exists, or a wrong choice is expensive to undo (data loss, an external or public action, a choice that later tickets of this batch build on). Stop, and end your report with `BLOCKED: <question>`, the options, and your recommendation. You will be resumed with the answer.

Final report, short: the commit hash (or "not committed" and why), whether #<n> is closed, any follow-up tickets you created, the test result, what the ticket delivered in one or two lines, and the Decisions section.

</implementer-prompt>

## 5. Blocked questions

Relay a `BLOCKED` question to the user with its options and the implementer's recommendation (AskUserQuestion in Claude Code). Resume the **same** implementer with the answer (SendMessage in Claude Code) so its context stays. The other running implementers carry on meanwhile; the blocked ticket's dependents wait until it closes.

## 6. Failure

A ticket **fails** when it ends with failing tests, creates a follow-up ticket, or stops without committing for any reason other than `BLOCKED`. Then:

- Start no ticket that depends on it, directly or through other tickets, and never start the full-tests ticket (the `(N/N)` one) for the rest of this run.
- Let running tickets finish, and keep starting frontier tickets that don't depend on it.
- Leave the failed ticket as it is: the failure goes in the end report for the user to decide, and a retry is theirs to ask for.

## 7. End report

One chat message, grouped per ticket in batch order:

- `#n (k/N) <title>`: the commit hash, or failed / blocked / not started and why.
- What it delivered, in one or two lines.
- Its decisions, numbered **D1, D2, …** across the whole batch, each with the choice and the alternatives it beat, so the user can answer "D3 wrong, use X".

End with the follow-up tickets created and the tickets left open.
