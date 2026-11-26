# Blog: core domain crate

Needs merged first: "Blog: repo foundation"

## Goal

Build the domain logic with no AWS code.

## In scope

- `api/core`: post model, slug and field validation, state transitions (draft, published, unpublish), API error types and codes.
- `PostRepository` trait and an in-memory fake.
- `contract/fixtures/*.json`: one example per response and error shape. Tests assert serialized output equals the fixtures.

## Decisions (read these files)

- `docs/wayfinder/blog/tickets/03-dynamodb-data-model-and-access-patterns.md`
- `docs/wayfinder/blog/tickets/04-api-contract.md`
- `docs/wayfinder/blog/tickets/08-test-strategy.md`

Vocabulary: `GLOSSARY.md`. Map: `docs/wayfinder/blog/map.md`. Definition of done: `make ci` is green and new logic has tests at the lowest fitting layer (see the test strategy ticket).

## Acceptance

- Unit tests cover validation limits, state transitions, error mapping, and the fixtures.
