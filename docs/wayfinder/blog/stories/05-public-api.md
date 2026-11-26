# Blog: public API Lambda

Needs merged first: "Blog: infra skeleton (table, hello Lambda, HTTP API)", "Blog: DynamoDB repository"

## Goal

Replace the hello Lambda with the real read endpoints.

## In scope

- `api/public`: `GET /api/posts` and `GET /api/posts/{slug}`. Drafts return `404`.
- `tracing` JSON setup. One request log line. Never log the values listed in the logging ticket.
- Wire the Lambda to the DynamoDB repository. The `public` role has read-only table access.
- Handler tests use the in-memory fake.

## Decisions (read these files)

- `docs/wayfinder/blog/tickets/04-api-contract.md`
- `docs/wayfinder/blog/tickets/02-author-auth-for-admin-writes.md`
- `docs/wayfinder/blog/tickets/08-test-strategy.md`
- `docs/wayfinder/blog/tickets/09-logging-and-alarms.md`

Vocabulary: `GLOSSARY.md`. Map: `docs/wayfinder/blog/map.md`. Definition of done: `make ci` is green and new logic has tests at the lowest fitting layer (see the test strategy ticket).

## Acceptance

- `make ci` is green. Handler tests cover list order, slug lookup, and draft `404`.

## Verify after merge (by the user)

Run `make deploy` and call both routes.
