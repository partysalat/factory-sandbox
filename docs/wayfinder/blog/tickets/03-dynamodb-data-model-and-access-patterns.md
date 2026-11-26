---
type: grilling
status: closed
blocked-by: []
claimed-by: Ben Lenser
---
## Question

What is the DynamoDB table design?
Cover keys (slug or id), post attributes, draft vs published, and the "list published posts, newest first" access pattern (GSI or other).
Decide the markdown storage form (raw only, or raw plus rendered), the timestamps, and how slug changes work.

## Resolution

- Table `Posts`: partition key `slug`, no sort key. The slug is chosen at creation, defaults from the title, and is immutable. No rename in v1. Pattern `^[a-z0-9]+(-[a-z0-9]+)*$`, 3-80 chars.
- Attributes: `slug`, `title`, `body` (raw markdown only), `status` (`draft` or `published`), `createdAt`, `updatedAt`, `publishedAt` (set on first publish only). ISO-8601 UTC strings. No excerpt, author field, or rendered HTML.
- Public list: one sparse GSI. Published items carry `gsi1pk = "PUBLISHED"` and `gsi1sk = publishedAt`. Query with `ScanIndexForward=false`. Projection `INCLUDE title`. No pagination in v1: return all published posts.
- Admin list: table `Scan`, metadata only, `body` dropped.
- State changes: unpublish removes the two GSI attributes and keeps `publishedAt`. Publish sets them again. Delete is a hard delete.
- Write safety: create is conditional (`attribute_not_exists(slug)`), a clash returns `409`. Update is last write wins. `body` max 100 KB, checked in the Lambda.
- Table settings: on-demand billing, no PITR, `RemovalPolicy.DESTROY` (`cdk destroy` deletes posts).
- Vocabulary: see `GLOSSARY.md` at the repo root.
