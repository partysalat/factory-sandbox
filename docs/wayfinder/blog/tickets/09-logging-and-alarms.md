---
type: grilling
status: closed
blocked-by: []
claimed-by: Ben Lenser
---
## Question

What logging and alarms does the sandbox have?
Decide the log format and level for the Lambdas (structured JSON, `tracing`), log retention, which fields to never log (the token, post bodies), and whether any alarm or budget alert exists. Dashboards are out of scope.

## Resolution

- Format: the `tracing` crate with JSON output, set up in each `main` with the helper from the `lambda_runtime` crate (check the exact helper name at build time). Default level `info`. The `RUST_LOG` environment variable overrides it. CDK sets the function log format to JSON.
- What to log: one `info` line per request with method, route pattern, status, duration in ms, and the Lambda request ID. Errors log at `error` with the API error code and the cause. Failed auth checks log at `warn` with no detail about the token. Slugs may appear in logs.
- Never log: the `Authorization` header, the admin token, the SSM value, post titles, post bodies.
- Error responses: the API error shape does not change. A `500` returns the generic message. The cause stays in the logs.
- Retention: CDK creates an explicit log group per Lambda with 14-day retention and `RemovalPolicy.DESTROY`.
- Alarms and extras: no CloudWatch alarms, no X-Ray, no API Gateway access logs, no CloudFront logs in v1. One AWS Budget (about 5 USD a month) emails an alert. CDK reads the address from the env var `BUDGET_EMAIL`, so it is not in the repo. If the variable is unset, the stack skips the budget. Dashboards stay out of scope.
