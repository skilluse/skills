# Better Auth Error Reference

When a user reports a Better Auth error, follow this process:

1. Identify the error code or message from context or user input
2. Explain in plain English what the error means
3. List common causes
4. Provide a specific fix with code if applicable
5. Note related errors to watch for
6. Link to relevant docs: https://better-auth.com/docs

## Common Errors

| Error | Cause | Fix |
|-------|-------|-----|
| `BETTER_AUTH_SECRET not set` | Missing env var | Add `BETTER_AUTH_SECRET` (32+ chars) to `.env` |
| `Invalid origin` | Request from untrusted origin | Add domain to `trustedOrigins` in auth config |
| `Session not found` | Invalid/expired session token | Re-authenticate; check `session.expiresIn` |
| `Email not verified` | `requireEmailVerification: true` but user hasn't verified | Trigger verification email; check email handler |
| `User already exists` | Duplicate email on sign-up | Handle `USER_ALREADY_EXISTS` error code in UI |
| `Invalid credentials` | Wrong email/password | Generic message to user; check `emailAndPassword.enabled` |
| `Too many requests` | Rate limit hit | Back off; check `rateLimit` config |
| `OAuth callback error` | Redirect URI mismatch | Verify redirect URI in provider dashboard matches `BETTER_AUTH_URL` |
| `Type errors after plugin` | Schema not regenerated | Run `npx @better-auth/cli@latest generate` or `migrate` |
| `Cookies not setting` | `baseURL` mismatch or missing secure cookies in prod | Match `baseURL` to domain; set `advanced.useSecureCookies: true` |

## Error Handling Pattern

```ts
const { data, error } = await signIn.email({ email, password })

if (error) {
  switch (error.code) {
    case "INVALID_EMAIL_OR_PASSWORD":
      setError("Invalid email or password")
      break
    case "EMAIL_NOT_VERIFIED":
      setError("Please verify your email first")
      break
    case "TOO_MANY_REQUESTS":
      setError("Too many attempts. Please wait and try again.")
      break
    default:
      setError("An unexpected error occurred")
  }
}
```

[Error codes reference](https://better-auth.com/docs/reference/error-codes)
