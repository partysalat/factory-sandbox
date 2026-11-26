---
type: grilling
status: closed
blocked-by: []
claimed-by: Ben Lenser
---
## Question

How does the single author prove identity for admin writes?
Candidates: shared secret in SSM Parameter Store checked by the Lambda (recommended), or Cognito.
Decide where the secret lives, how the admin UI stores and sends it, and what the Lambda returns on failure.

## Resolution

- Credential: one random token (32 random bytes, base64). The admin UI sends `Authorization: Bearer <token>`. The Lambda compares in constant time.
- Storage: SSM `SecureString`, created by hand, not by CDK. A `make secret` target generates and stores the token and prints it once. CDK references the parameter name and grants the write Lambda `ssm:GetParameter` plus KMS decrypt.
- Read pattern: read once per cold start and cache in memory. Rotation = update the parameter, then redeploy.
- Admin UI: paste the token on a login screen. Keep it in `sessionStorage`. Never put it in the bundle. Any `401` clears it and shows the login screen.
- Failure: all admin routes return `401` with a generic body. No custom rate limiting. API Gateway default throttle only.
- Boundary: public routes return published posts only. A draft by slug returns `404`, same as a missing post. The check lives in the Lambda.
- Out: Cognito and reader accounts.
