# Blog: public web app

Needs merged first: "Blog: repo foundation"

## Goal

Build the reader pages. Tests use the contract fixtures, so this story does not need a deployed API.

## In scope

- `web/src/api.ts` typed `fetch` wrapper. `@solidjs/router` with `/`, `/posts/:slug`, and `*`.
- `renderMarkdown()` with `marked` and `DOMPurify`, forbidding `script`, `img`, `iframe`. Per-route `document.title`.
- Plain CSS, system fonts, one responsive column, dark mode. Dates with `Intl.DateTimeFormat`.
- `vitest` tests for markdown sanitization, the api wrapper, and the pages.

## Decisions (read these files)

- `docs/wayfinder/blog/tickets/07-frontend-structure-and-markdown-rendering.md`
- `docs/wayfinder/blog/tickets/04-api-contract.md`
- `docs/wayfinder/blog/tickets/08-test-strategy.md`

Vocabulary: `GLOSSARY.md`. Map: `docs/wayfinder/blog/map.md`. Definition of done: `make ci` is green and new logic has tests at the lowest fitting layer (see the test strategy ticket).

## Acceptance

- `make ci` is green. Sanitization tests assert that `script`, `img`, `iframe`, and event-handler attributes are stripped.

## Verify after merge (by the user)

After story 7 is deployed: run `make deploy` and read a published post.
