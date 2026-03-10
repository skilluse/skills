# Better Auth Providers Reference

Quick reference for configuring authentication providers in Better Auth.

## OAuth / Social Providers

Configure under `socialProviders` in `auth.ts`:

```ts
socialProviders: {
  google: { clientId: process.env.GOOGLE_CLIENT_ID!, clientSecret: process.env.GOOGLE_CLIENT_SECRET! },
  github: { clientId: process.env.GITHUB_CLIENT_ID!, clientSecret: process.env.GITHUB_CLIENT_SECRET! },
  discord: { clientId: process.env.DISCORD_CLIENT_ID!, clientSecret: process.env.DISCORD_CLIENT_SECRET! },
  microsoft: { clientId: process.env.MICROSOFT_CLIENT_ID!, clientSecret: process.env.MICROSOFT_CLIENT_SECRET! },
  apple: { clientId: process.env.APPLE_CLIENT_ID!, clientSecret: process.env.APPLE_CLIENT_SECRET! },
  twitter: { clientId: process.env.TWITTER_CLIENT_ID!, clientSecret: process.env.TWITTER_CLIENT_SECRET! },
}
```

Trigger from client: `signIn.social({ provider: "google", callbackURL: "/dashboard" })`

## Email & Password

```ts
emailAndPassword: {
  enabled: true,
  requireEmailVerification: true,    // optional
  sendResetPassword: async ({ user, url }) => { /* send email */ },
}
```

## Magic Link (Passwordless Email)

```ts
import { magicLink } from "better-auth/plugins"

plugins: [
  magicLink({
    sendMagicLink: async ({ email, url }) => { /* send email */ }
  })
]
```

Client: `import { magicLinkClient } from "better-auth/client/plugins"`

## Passkey (WebAuthn)

```ts
import { passkey } from "@better-auth/passkey"
plugins: [passkey()]
```

Client: `import { passkeyClient } from "@better-auth/passkey/client"`

## Phone Number / OTP

```ts
import { phoneNumber } from "better-auth/plugins"
plugins: [phoneNumber({ sendOTP: async ({ phoneNumber, code }) => { /* send SMS */ } })]
```

## Provider Dashboard Redirect URIs

| Provider | Redirect URI |
|----------|-------------|
| Google | `{BETTER_AUTH_URL}/api/auth/callback/google` |
| GitHub | `{BETTER_AUTH_URL}/api/auth/callback/github` |
| Discord | `{BETTER_AUTH_URL}/api/auth/callback/discord` |
| Microsoft | `{BETTER_AUTH_URL}/api/auth/callback/microsoft` |

[Full provider docs](https://better-auth.com/docs/authentication)
