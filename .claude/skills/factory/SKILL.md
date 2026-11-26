---
name: factory
description: Factory CLI. Work items (IDs like DP-31), the backlog, and what the factory waits on the user for.
---

# Factory

`factory help agents` holds the rules, the stages and the states. Run it first in each session. `factory --help` lists the commands.

## Read freely

`status`, `list`, `show`, `manifest`, `doctor`, `project list`, and `add` (adds a backlog item, starts nothing).
Pass `--json`. Pass `--project <KEY>` where required; take the key from `project list`.

## Guarded

`refine`, `answer`, `approve`, `unblock` and `cancel` each spend tokens, merge a pull request (`approve` on a ticket) or end work for good.
Run one when the user names the item and the action in this conversation.

## Steps

1. Orient: run `factory status --json`, then `factory show <ID> --json` for each item with `needsYou`.
   Done when you can state, for every such item, the reason and the next command.
2. Report: give the user the ID, the reason and the command you would run. Read out the question, spec or plan in short form.
   Done when every needs-you item has all three.
3. Act: run the guarded command the user chose, once. An agent run takes minutes and the command waits for it.
   Done when `factory show <ID>` shows the new stage.
