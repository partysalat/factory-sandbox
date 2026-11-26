# Blog: DynamoDB repository

Needs merged first: "Blog: core domain crate"

## Goal

Implement `PostRepository` on DynamoDB.

## In scope

- DynamoDB implementation in `api/core` (or a separate crate): conditional create (`409` on clash), update, publish and unpublish with sparse GSI attributes, delete, list published, list all.
- `#[ignore]` integration tests on DynamoDB Local. `make api-test-integration` starts it with `podman`, runs `cargo test -- --ignored`, and stops it.

## Decisions (read these files)

- `docs/wayfinder/blog/tickets/03-dynamodb-data-model-and-access-patterns.md`
- `docs/wayfinder/blog/tickets/08-test-strategy.md`

Vocabulary: `GLOSSARY.md`. Map: `docs/wayfinder/blog/map.md`. Definition of done: `make ci` is green and new logic has tests at the lowest fitting layer (see the test strategy ticket).

## Acceptance

- `make ci` is green. Integration tests are not part of `make ci`.

## Verify after merge (by the user)

Run `make api-test-integration`.
