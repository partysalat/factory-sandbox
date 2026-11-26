# Blog: admin API Lambda and token auth

Needs merged first: "Blog: public API Lambda"

## Goal

Add the write endpoints behind the admin token.

## In scope

- `api/admin`: all admin routes from the API contract. Bearer token check, constant-time compare, token cached per cold start from SSM.
- Generic `401`. Publish needs a non-empty title and body. Body limit 100 KB.
- Infra: `admin` Lambda with read-write table access and `ssm:GetParameter` plus KMS decrypt. Routes wired in the HTTP API.
- `make secret` generates a token, stores it as an SSM `SecureString`, and prints it once.

## Decisions (read these files)

- `docs/wayfinder/blog/tickets/02-author-auth-for-admin-writes.md`
- `docs/wayfinder/blog/tickets/04-api-contract.md`
- `docs/wayfinder/blog/tickets/03-dynamodb-data-model-and-access-patterns.md`
- `docs/wayfinder/blog/tickets/09-logging-and-alarms.md`
- `docs/wayfinder/blog/tickets/08-test-strategy.md`

Vocabulary: `GLOSSARY.md`. Map: `docs/wayfinder/blog/map.md`. Definition of done: `make ci` is green and new logic has tests at the lowest fitting layer (see the test strategy ticket).

## Acceptance

- `make ci` is green. Handler tests cover auth failure, create conflict, publish rules, and the `404` and `409` paths.

## Verify after merge (by the user)

Run `make secret`, `make deploy`, then create, publish, and delete a post with `curl` and the token.
