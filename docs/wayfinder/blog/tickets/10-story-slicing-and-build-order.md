---
type: grilling
status: closed
blocked-by: [08-test-strategy, 09-logging-and-alarms]
claimed-by: Ben Lenser
---
## Question

How do we cut the decided design into factory stories for project `FS`, and in what order?
Stories cannot block each other, so the order is a list that you refine one at a time. Decide the slices, the order, each story's body (decisions copied in, acceptance checks), and the prerequisite outside the repo (`rustup` in the `sf-worker` image).

## Resolution

- Nine stories, added to `FS` in this order so the IDs ascend: repo foundation; infra skeleton (table, hello Lambda, HTTP API); core domain crate; DynamoDB repository; public API Lambda; admin API Lambda and token auth; edge (S3, CloudFront, headers, smoke test); public web app; admin web app. Edge comes before the web stories so the SPA has a place to deploy. The infra skeleton comes early to prove the Rust packaging.
- Drafts: `stories/01-...md` to `stories/09-...md`. Each body has a goal, an in-scope list, links to decision tickets by path (decisions are not copied), an acceptance list, and "Verify after merge" for infra stories. Each names the earlier stories that must be merged first, because the factory has no dependency links.
- Prerequisites, in order: (1) merge `GLOSSARY.md` and `docs/wayfinder/` to `main`; (2) add `rustup` to the `sf-worker` image; (3) before the first deploy (story 2): run `make setup` and `cdk bootstrap` once for the account in `eu-west-1`.
- Region and stack: `eu-west-1`, stack name `BlogStack`, account from credentials, no account ID in the repo.
- Process: the user reviews the drafts. Then `factory add` runs for each. `factory refine` stays with the user, one story at a time. The existing story FS-5 (images) stays at the end of the backlog.

## Added to the factory

FS-6 repo foundation, FS-7 infra skeleton, FS-8 core domain, FS-9 DynamoDB repository, FS-10 public API, FS-11 admin API, FS-12 edge, FS-13 public web, FS-14 admin web. All at backlog. FS-5 (images) is out of scope for v1.
