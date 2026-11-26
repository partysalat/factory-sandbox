# Issue tracker: factory CLI

Work items live in the factory CLI. The project key is `FS`. The GitHub repo is `partysalat/factory-sandbox`.

## Rules
- Run `factory help agents` first in each session.
- Pass `--json` to read commands. Pass `--project FS` to `add` and `list`.
- Read freely: `status`, `list`, `show`, `manifest`, `doctor`, `project list`, `add`.
- Guarded: `refine`, `answer`, `approve`, `unblock`, `cancel`. These spend tokens, merge a pull request, or end work. Run one only when the user names the item and the action.

## Operations
- Create an item: `factory add --project FS --title "<title>" --body "<body>"`. This adds a backlog item and starts nothing.
- List items: `factory list --project FS --json`. Add `--needs-you` for items that wait for the user.
- Read an item: `factory show <ID> --json`.
- Start work: `factory refine <ID>`.
- Answer an agent question: `factory answer <ID>`.
- Merge a ticket pull request or approve a story: `factory approve <ID>`.

## Stages
backlog, refining, spec_review*, planning, plan_review*, tickets, story_approval*, done. A star means the stage waits for `factory approve`.
Each ticket ends at awaiting_merge, where `approve` merges its pull request.

## States
active, awaiting_input, blocked, failed, cancelled, done.

## PRs as a request surface
Off.
