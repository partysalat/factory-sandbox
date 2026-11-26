# Blog: admin web app

Needs merged first: "Blog: public web app", "Blog: edge (S3, CloudFront, headers, smoke test)"

## Goal

Build the author UI.

## In scope

- Login gate: paste the token, keep it in `sessionStorage`, validate with `GET /api/admin/posts`. Any `401` clears the token and shows the login screen.
- Routes `/admin`, `/admin/new`, `/admin/edit/:slug`. Admin list with status.
- Editor: title, slug (create only), markdown `textarea` with live preview, Save, Publish or Unpublish, Delete with confirm, unsaved-changes warning.
- Component tests for the login gate and the editor button states.

## Decisions (read these files)

- `docs/wayfinder/blog/tickets/07-frontend-structure-and-markdown-rendering.md`
- `docs/wayfinder/blog/tickets/02-author-auth-for-admin-writes.md`
- `docs/wayfinder/blog/tickets/04-api-contract.md`
- `docs/wayfinder/blog/tickets/08-test-strategy.md`

Vocabulary: `GLOSSARY.md`. Map: `docs/wayfinder/blog/map.md`. Definition of done: `make ci` is green and new logic has tests at the lowest fitting layer (see the test strategy ticket).

## Acceptance

- `make ci` is green.

## Verify after merge (by the user)

Run `make deploy`. Log in, write a post, publish it, see it on the public page, unpublish it, delete it.
