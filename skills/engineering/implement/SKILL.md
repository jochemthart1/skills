---
name: implement
description: "Implement a piece of work based on a spec or set of tickets."
disable-model-invocation: true
---

Implement the work described by the user in the spec or tickets.

If the user passes a ticket reference, fetch it from the issue tracker and state its title before starting. If the reference is ambiguous, ask.

Call the Skill tool with "tdd" where possible, at pre-agreed seams.

Run typechecking regularly and single test files regularly. At the end, run the tests the ticket's acceptance criteria name:

- **Related tests pass**: the tests you wrote or changed, plus the existing tests covering the code you touched. Not the full suite.
- **Full test suite passes**: the full suite. Fix failures here; when a fix won't fit in this context, create a new ticket for it, record the failure on this ticket, and stop without committing.
- No tests named (or no ticket): the full suite.

Once done, call the Skill tool with "code-review" to review this ticket's diff. Skip it on a full-tests ticket that changed no code.

Commit your work to the current branch, one commit per ticket, referencing the ticket (`Closes #<n>` on a real tracker). Then close the ticket the way the issue tracker closes work.
