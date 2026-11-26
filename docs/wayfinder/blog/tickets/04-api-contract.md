---
type: grilling
status: closed
blocked-by: [02-author-auth-for-admin-writes, 03-dynamodb-data-model-and-access-patterns]
claimed-by: Ben Lenser
---
## Question

What are the HTTP endpoints, payloads, error shapes, and auth header?
Split public reads from admin writes. Decide one Lambda per route or one Lambda with a router, and API Gateway HTTP API vs Lambda Function URL.

## Resolution

- Entry: API Gateway HTTP API. Two Rust Lambdas, each with a small internal router. `public` has read-only table access. `admin` has read-write table access and `ssm:GetParameter`.
- Routes (all carry the `/api` prefix):
  - `GET /api/posts` (none): `{posts:[{slug,title,publishedAt}]}`, newest first.
  - `GET /api/posts/{slug}` (none): `{slug,title,body,publishedAt}`. `404` for a draft.
  - `GET /api/admin/posts` (Bearer): `{posts:[{slug,title,status,createdAt,updatedAt,publishedAt}]}`, sorted by `updatedAt` descending in the Lambda.
  - `GET /api/admin/posts/{slug}` (Bearer): full post.
  - `POST /api/admin/posts` (Bearer): body `{slug,title,body}`, creates a draft, `201`.
  - `PUT /api/admin/posts/{slug}` (Bearer): body `{title,body}`, returns the post.
  - `POST /api/admin/posts/{slug}/publish` and `/unpublish` (Bearer): return the post.
  - `DELETE /api/admin/posts/{slug}` (Bearer): `204`.
- Write rules: `POST` creates drafts only. `PUT` works on drafts and published posts and never changes `status`, `slug`, or `publishedAt`. Publish needs a non-empty `title` and `body`, else `400`. Publish and unpublish are idempotent (`200`, no change).
- Errors: `{"error":{"code","message"}}`. Codes: `validation_failed` 400 (adds a `fields` object keyed by field name), `unauthorized` 401 (no message), `not_found` 404, `conflict` 409 (slug taken), `body_too_large` 413, `internal` 500.
- Conventions: JSON only, camelCase keys, ISO-8601 UTC timestamps, no version prefix.
- Limits: `title` 1-200 chars after trim. `body` max 100 KB. An empty body is allowed in a draft. Slug pattern from the data model. Unknown JSON fields are ignored.
- Left to later tickets: CORS and cache headers (edge routing), markdown rendering (frontend).
