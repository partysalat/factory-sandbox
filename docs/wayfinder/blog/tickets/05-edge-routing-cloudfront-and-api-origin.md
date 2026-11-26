---
type: grilling
status: closed
blocked-by: [02-author-auth-for-admin-writes]
claimed-by: Ben Lenser
---
## Question

How does CloudFront route traffic?
Decide whether `/api/*` goes to the API on the same domain (no CORS) or the SPA calls the API URL directly. Cover SPA fallback to `index.html`, cache rules for the API, and how the SPA learns the API base URL.

## Resolution

- One CloudFront distribution, two origins. Default behavior: S3 bucket. `/api/*` behavior: the HTTP API. The SPA calls relative `/api/...` URLs. No CORS. No API URL in the bundle.
- SPA fallback: a CloudFront Function on the default behavior's viewer-request rewrites any path with no file extension to `/index.html`. No custom error responses, because they apply to the whole distribution and would turn an API `404` into the HTML page.
- Bucket: private, Origin Access Control, no website hosting. `RemovalPolicy.DESTROY` with `autoDeleteObjects`.
- Caching: `/assets/*` gets `Cache-Control: public, max-age=31536000, immutable`. `index.html` gets `no-cache`. `/api/*` uses managed `CachingDisabled` with managed `AllViewerExceptHostHeader` origin request policy, so `Authorization` reaches the Lambda. No API caching in v1.
- API origin: the HTTP API default `execute-api` URL. Routes already carry `/api`, so no origin path rewrite. Direct access to that URL is accepted. No secret origin header.
- Asset deploy: CDK `BucketDeployment` uploads `web/dist` and invalidates `/*` on each `cdk deploy`. `make deploy` builds the web app first.
- Distribution: default `*.cloudfront.net` domain, HTTPS only (HTTP redirects), `PriceClass_100`, HTTP/2 and 3, managed `SecurityHeadersPolicy`. A custom CSP is left to the frontend ticket.
- Local dev: the Vite dev server proxies `/api` to the deployed distribution. The env var `VITE_API_PROXY_TARGET` holds the target. No local Lambda emulation in v1.

## Amendment (from the frontend ticket)

The managed `SecurityHeadersPolicy` is replaced by one custom response headers policy. It sets the same security headers plus the CSP defined in the frontend ticket.
