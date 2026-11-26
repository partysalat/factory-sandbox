# Blog: edge (S3, CloudFront, headers, smoke test)

Needs merged first: "Blog: admin API Lambda and token auth"

## Goal

Serve the SPA and the API from one CloudFront distribution.

## In scope

- Private S3 bucket with OAC. CloudFront distribution with two origins and the behaviors from the edge ticket.
- CloudFront Function for the SPA fallback. Custom response headers policy with the CSP from the frontend ticket.
- `BucketDeployment` of `web/dist` with the cache headers from the edge ticket. `make deploy` builds the web app first.
- Optional AWS Budget from `BUDGET_EMAIL`.
- `make smoke`: the `curl` script from the test strategy ticket. Output of the CDK stack gives the distribution URL.

## Decisions (read these files)

- `docs/wayfinder/blog/tickets/05-edge-routing-cloudfront-and-api-origin.md`
- `docs/wayfinder/blog/tickets/07-frontend-structure-and-markdown-rendering.md`
- `docs/wayfinder/blog/tickets/08-test-strategy.md`
- `docs/wayfinder/blog/tickets/09-logging-and-alarms.md`

Vocabulary: `GLOSSARY.md`. Map: `docs/wayfinder/blog/map.md`. Definition of done: `make ci` is green and new logic has tests at the lowest fitting layer (see the test strategy ticket).

## Acceptance

- `make ci` is green.

## Verify after merge (by the user)

Run `make deploy` and `make smoke`. All checks pass.
