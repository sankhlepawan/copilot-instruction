---
applyTo:
  - "**/auth/**"
  - "**/security/**"
  - "**/middleware/**"
  - "**/*auth*.py"
  - "**/*auth*.ts"
  - "**/*auth*.js"
  - "**/*auth*.go"
---

# Security-critical code review

These paths handle authentication, authorization, sessions, or security middleware.
Apply extra scrutiny — most comments here should be `[must-fix]` or `[should-fix]`.

## Required

- Every new endpoint or handler must have an explicit auth check. Flag any missing.
- Password storage must use `bcrypt`, `argon2`, or `scrypt`. Flag any other algorithm
  as `[must-fix]`.
- Token / hash comparisons must be constant-time:
  - Python: `secrets.compare_digest`
  - Node: `crypto.timingSafeEqual`
  - Go: `crypto/subtle.ConstantTimeCompare`
  Flag `==` comparisons on tokens, HMACs, or password hashes as `[must-fix]`.
- JWT verification must:
  - Verify signature explicitly (no `verify=False`, no `algorithms=["none"]`).
  - Validate `iss`, `aud`, and `exp` claims.
  - Pin allowed algorithms (`algorithms=["RS256"]`, never accept arbitrary).
- Sessions must set `Secure`, `HttpOnly`, and `SameSite=Strict` (or `Lax` with
  inline justification).

## Disallowed

- Logging passwords, tokens, session IDs, full auth headers, or PII — even at DEBUG.
- Authorization based on "trusted" client-supplied headers (`X-User-Id`, `X-Role`,
  `X-Forwarded-User`) without server-side verification.
- Hand-rolled crypto. Always use the standard library or a vetted package.
- Disabling TLS certificate verification (`verify=False`, `rejectUnauthorized: false`,
  `InsecureSkipVerify: true`) — even in tests.
- Storing secrets in source, including in test fixtures with realistic-looking values.
  Use obvious placeholders like `"test-password-not-real"`.

## Patterns to flag as `[must-fix]`

- Any `TODO`, `FIXME`, or `XXX` left in security-critical code without an issue link.
- Any commented-out auth check, rate limit, or input validation.
- Any catch-all error handler in auth code that hides the underlying error from logs.
- Regex used for parsing security-relevant input (URLs, emails for auth) — prefer
  dedicated parsers.

## Common review prompts

When reviewing changes in these paths, explicitly check:

1. Does this change touch the auth path? If so, what is the new attack surface?
2. Are there test cases covering the negative path (invalid token, expired token,
   wrong audience, replay)?
3. If this fails open, what does an attacker gain?
4. Is anything sensitive being logged?
