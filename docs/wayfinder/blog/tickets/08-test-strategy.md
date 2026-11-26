---
type: grilling
status: closed
blocked-by: []
claimed-by: Ben Lenser
---
## Question

What is the test strategy?
Decide the frameworks and test layers for the Rust crates (unit tests in `core`, handler tests, DynamoDB access with a fake or a real table), the SPA (`vitest`, component tests or not), and the CDK app.
Decide what must be tested to count a story done, and what `make ci` runs, within the limits set by the layout ticket (no `cargo-lambda`, `zig`, or AWS).

## Resolution

- Rust: `core` holds a `PostRepository` trait (get, list published, list all, create, update, publish, unpublish, delete) with a DynamoDB implementation and an in-memory fake. `core` tests cover slug and field validation, state transitions, and error mapping against the fake. The `public` and `admin` handlers stay thin. Handler tests call them with a `lambda_http` request against the fake. All run in `make ci` via `cargo test`.
- DynamoDB behavior (conditional create returns `409`, sparse GSI): `#[ignore]` integration tests on the real repository. `make api-test-integration` starts DynamoDB Local with `podman`, runs `cargo test -- --ignored`, and stops it. Not in `make ci`. DynamoDB logic is only checked when the user runs that target.
- Web: `vitest` with `@solidjs/testing-library` in jsdom. Unit tests: `renderMarkdown()` (assert `script`, `img`, `iframe`, and event-handler attributes are stripped), the `api.ts` wrapper (a `401` clears the token), slug derivation. Component tests: the login gate and the editor's button states. No browser end-to-end tool in v1.
- Contract drift: `contract/fixtures/*.json` holds one example per response and error shape. Rust tests assert that serialized output equals the fixtures. Web tests use the same files as mock responses. No code generation.
- Infra: no CDK unit tests in v1. `make ci` runs `tsc --noEmit` and lint only. `make synth` is the infra check.
- Smoke test: `make smoke` runs a `curl` script against the deployed distribution URL (from the CDK stack output). Checks: `GET /api/posts` returns `200` JSON; `GET /` returns the HTML shell; `GET /posts/anything` returns the HTML shell; `GET /api/posts/missing` returns `404` with the JSON error shape; an admin route without a token returns `401`; the response carries the CSP header. Not in `make ci`.
- Definition of done: `make ci` is green and new logic has tests at the lowest fitting layer. No coverage threshold. Infra-touching stories are verified by the user after merge with `make deploy` and `make smoke`, because factory agents cannot run synth or deploy. Each infra story body names that check.
