# Blog: infra skeleton (table, hello Lambda, HTTP API)

Needs merged first: "Blog: repo foundation"

## Goal

Deploy the smallest real stack. This proves the Rust Lambda packaging before real code depends on it.

## In scope

- CDK stack `BlogStack` in `eu-west-1` (account from credentials, none stored in the repo).
- DynamoDB table `Posts` and the sparse GSI as in the data model ticket.
- A hello `public` Rust Lambda: `GET /api/posts` returns `{"posts":[]}`. Use `cargo-lambda-cdk`, arm64, `provided.al2023`. Check the open items in the research note (install, synth, `architecture` prop).
- HTTP API with the `/api` routes wired. Explicit log group with 14-day retention. JSON log format.
- Make targets `synth`, `deploy`, `destroy` become real.

## Decisions (read these files)

- `docs/wayfinder/blog/tickets/01-rust-lambda-build-and-cdk-packaging.md`
- `docs/wayfinder/blog/tickets/03-dynamodb-data-model-and-access-patterns.md`
- `docs/wayfinder/blog/tickets/04-api-contract.md`
- `docs/wayfinder/blog/tickets/09-logging-and-alarms.md`
- `docs/wayfinder/blog/tickets/06-monorepo-layout-make-targets-and-ci.md`

Vocabulary: `GLOSSARY.md`. Map: `docs/wayfinder/blog/map.md`. Definition of done: `make ci` is green and new logic has tests at the lowest fitting layer (see the test strategy ticket).

## Acceptance

- `make ci` is green. Infra checks are `tsc --noEmit` and lint only.

## Verify after merge (by the user)

Run `make setup` and `cdk bootstrap` once. Run `make deploy`. Then `curl <api-url>/api/posts` returns `{"posts":[]}`.
