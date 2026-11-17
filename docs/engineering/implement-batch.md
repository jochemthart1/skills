## What it does

`implement-batch` runs a whole **batch** of `(n/N)` [tickets](https://www.aihero.dev/ai-coding-dictionary/ticket) made by [to-tickets](https://aihero.dev/skills-to-tickets), so you don't have to clear the [context window](https://www.aihero.dev/ai-coding-dictionary/context-window) and type `/implement` once per ticket. The orchestrating [agent](https://www.aihero.dev/ai-coding-dictionary/agent) hands each ticket to a fresh background [subagent](https://www.aihero.dev/ai-coding-dictionary/subagent) that follows [implement](https://aihero.dev/skills-implement)'s rules exactly: the ticket's own tests, `code-review`, one commit, ticket closed.

It keeps your run unattended without making it silent. Implementers decide every question that has a reasonable default and write each one down as a **decision**; only a question that is costly to get wrong comes back to you as `BLOCKED`. The end report numbers every decision across the batch (D1, D2, …), so correcting one is a single line.

## When to reach for it

You invoke this by typing `/implement-batch #n` (any ticket of the batch) or `/implement-batch next`, and the agent won't reach for it on its own.

| Your situation | Reach for |
| --- | --- |
| A `(n/N)` batch you'd otherwise work with `/clear` + `/implement` per ticket | `/implement-batch` |
| One lone ticket | [implement](https://aihero.dev/skills-implement) |
| A spec you want built on an integration branch, each ticket in its own worktree | [implement-spec](https://aihero.dev/skills-implement-spec) |

## Prerequisites

- **An issue tracker** configured by [setup-matt-pocock-skills](https://aihero.dev/skills-setup-matt-pocock-skills), holding the batch's tickets with their blocking edges.
- **A `Parallel-safe: yes` line** on tickets that may run beside others. Without it a ticket runs on its own, which is safe but slower. Your project's agent docs say what counts as parallel-safe.

## The frontier, on one working tree

Blocking edges decide what can start: the **frontier** is every open ticket whose blockers are closed. Unlike `implement-spec`, every implementer works in the same checkout on the current branch, so the batch lands as one commit per ticket, exactly as if you had run them by hand. That is also why parallelism is opt-in per ticket: several parallel-safe tickets can run together, with at most one ticket that isn't parallel-safe among them, and each implementer stages only its own files.

A ticket that fails (red tests, a follow-up ticket, no commit) stops everything that depends on it, including the full-tests ticket, and nothing is retried. Independent tickets keep going.

## Common questions

**What counts as a blocking question?**

One with no reasonable default, or one where a wrong choice is expensive to undo: data loss, an external or public action, or a choice the batch's later tickets build on. Everything else the implementer decides and records, even where your project docs normally say "ask the user". Answering a `BLOCKED` question resumes the same implementer, so it keeps everything it had read.

**I picked up the batch halfway. Does it redo the closed tickets?**

No. Closed tickets count as done, so the run starts at the open ones.

## It's working if

- You type one command and see a `▶` line per ticket start and a `✔` line per commit, with no `/clear` in between.
- Parallel-safe tickets overlap; the others run alone.
- Each decision shows up both in the end report and as a comment on its ticket.
- A blocked ticket asks you once, and the run carries on with the others while you think.

## Where it fits

`implement-batch` is the build step of the main chain, replacing the hand-run loop of [implement](https://aihero.dev/skills-implement):

```txt
grill-with-docs → to-tickets → implement-batch → retro
```

Its neighbours are [to-tickets](https://aihero.dev/skills-to-tickets), which writes the `(n/N)` titles, blocking edges and Parallel-safe lines it reads, and [implement-spec](https://aihero.dev/skills-implement-spec), the worktree-per-ticket alternative. [ask-matt](https://aihero.dev/skills-ask-matt) is the router over the whole set.
