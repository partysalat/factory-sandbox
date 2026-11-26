---
label: wayfinder:map
---
# Map: Small blog (SolidJS, Rust Lambda, DynamoDB)

## Destination

A decision set that covers every key choice for the blog, plus a list of build stories ready to add to factory project `FS`. No code is written in this map.

## Notes

- Tracker: local markdown. Tickets live in `tickets/`. Front matter holds `type`, `status` (open, closed), `blocked-by`, `claimed-by`.
- Plan, do not build. Each ticket resolves a decision.
- Single author. Anonymous readers. Sandbox to test the factory workflow, with real AWS deploys.
- All build, test, and deploy steps run through `make` targets.
- Deploy by hand with `cdk deploy`. No CD pipeline.
- Facts: AWS CLI is logged in. `pnpm`, `sam`, Node 22 exist. Rust, `cargo-lambda`, and `zig` are not installed.

## Decisions so far

- Scope: markdown posts, draft/publish, admin UI for the author, public post list and post page.
- Stack: monorepo in this repo. Vite + SolidJS + TypeScript frontend. Rust Lambdas. DynamoDB stores posts. CDK defines infra. S3 + CloudFront hosts the SPA.
- Process: `make` is the interface. Manual `cdk deploy`.
- [Rust Lambda build and CDK packaging](tickets/01-rust-lambda-build-and-cdk-packaging.md): TypeScript CDK in `/infra`. `cargo-lambda` + `cargo-lambda-cdk`, arm64, `provided.al2023`. Install and synth are unverified.
- [Author auth for admin writes](tickets/02-author-auth-for-admin-writes.md): one Bearer token in SSM `SecureString`, cached per cold start, pasted into the admin UI (`sessionStorage`). Drafts return `404` on public routes.
- [DynamoDB data model and access patterns](tickets/03-dynamodb-data-model-and-access-patterns.md): table `Posts` keyed by immutable `slug`, sparse GSI for the public list, no pagination, on-demand, `DESTROY`.
- [API contract](tickets/04-api-contract.md): HTTP API, `public` and `admin` Lambdas, nine `/api` routes, one JSON error shape, drafts-only create, publish needs title and body.
- [Edge routing: CloudFront and API origin](tickets/05-edge-routing-cloudfront-and-api-origin.md): one distribution, S3 default and `/api/*` to HTTP API, extensionless-path CloudFront Function, no API caching, Vite proxy for local dev.
- [Monorepo layout, make targets, and CI](tickets/06-monorepo-layout-make-targets-and-ci.md): `web/`, `api/` (Cargo workspace), `infra/`; `make ci` needs no `cargo-lambda`, `zig`, or AWS; Biome; real `ci` replaces the `fixed.txt` stub.
- [Frontend structure and markdown rendering](tickets/07-frontend-structure-and-markdown-rendering.md): `@solidjs/router`, `createResource`, `marked` + `DOMPurify` in the browser, strict CSP (amends the edge ticket), admin editor with live preview, plain CSS.
- [Test strategy](tickets/08-test-strategy.md): `PostRepository` trait with fake, thin handlers, ignored DynamoDB Local tests, `vitest` for web, shared contract fixtures, no infra unit tests, `make smoke` after deploy.
- [Logging and alarms](tickets/09-logging-and-alarms.md): `tracing` JSON logs, never log token or post content, 14-day log groups, no alarms, optional AWS Budget email via `BUDGET_EMAIL`.
- [Story slicing and build order](tickets/10-story-slicing-and-build-order.md): nine stories in a fixed order, drafts in `stories/`, prerequisites (merge docs to `main`, `rustup` in `sf-worker`, `cdk bootstrap` in `eu-west-1`), `eu-west-1` and `BlogStack`.

## Not yet specified

<!-- empty: the way is clear -->

## Out of scope

- Comments, tags, search, RSS.
- Reader accounts.
- Custom domain, monitoring dashboards.
- CD pipeline and OIDC to AWS.
- Images in posts: v1 has no image routes or fields. Tracked as a separate factory backlog story, FS-5 "Images in blog posts".
