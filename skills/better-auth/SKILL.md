---
name: better-auth
description: Scaffold, configure, and troubleshoot authentication in TypeScript/JavaScript apps using Better Auth. Use when users want to add login, sign-up, OAuth, 2FA, or any auth feature. Also use when users mention Better Auth, auth.ts, or need to set up authentication with email/password, OAuth, or plugins.
---

# Better Auth Skill

Guide for adding or configuring authentication using [Better Auth](https://better-auth.com/docs).

---

## Phase 1: Planning (REQUIRED before implementation)

### Step 1: Scan the project

Auto-detect from codebase:
- **Framework** — `next.config`, `svelte.config`, `nuxt.config`, `astro.config`, `vite.config`, Express/Hono entry files
- **Database/ORM** — `prisma/schema.prisma`, `drizzle.config`, `package.json` deps (`pg`, `mysql2`, `better-sqlite3`, `mongoose`)
- **Existing auth** — `next-auth`, `lucia`, `clerk`, `supabase/auth`, `firebase/auth` in `package.json`
- **Package manager** — `pnpm-lock.yaml`, `yarn.lock`, `bun.lockb`, `package-lock.json`

Use findings to pre-fill defaults and skip questions already answered.

### Step 2: Ask planning questions (single call, skip if already known)

1. **Project type** — New project | Adding to existing | Migrating from another auth lib
2. **Framework** — Next.js App Router | Next.js Pages | SvelteKit | Nuxt | Astro | Express | Hono | SolidStart
3. **Database** — PostgreSQL (Prisma/Drizzle/pg) | MySQL (Prisma/Drizzle/mysql2) | SQLite (Prisma/Drizzle/better-sqlite3) | MongoDB
4. **Auth methods** *(multi-select)* — Email & password | Social OAuth | Magic link | Passkey | Phone number
5. **Social providers** *(if OAuth selected)* — Google | GitHub | Apple | Microsoft | Discord | Twitter/X
6. **Email verification** *(if email/password selected)* — Yes | No
7. **Email provider** *(if verification Yes)* — Resend | Mock (console.log)
8. **Plugins** *(multi-select)* — 2FA | Organizations/teams | Admin dashboard | API bearer tokens | Password reset | None
9. **Auth pages** *(multi-select)* — Sign in | Sign up | Forgot password | Reset password | Email verification
10. **UI style** — Minimal & clean | Centered card | Split layout | Glassmorphism

### Step 3: Confirm plan

Present a markdown checklist summarizing framework, database, auth methods, plugins, and implementation steps. Wait for user confirmation before Phase 2.

---

## Phase 2: Implementation

```
Is this a new/empty project?
├─ YES → New project setup
│   1. Install better-auth (+ scoped packages per plan)
│   2. Create auth.ts with all planned config
│   3. Create auth-client.ts with framework client
│   4. Set up route handler
│   5. Set up environment variables
│   6. Run CLI migrate/generate
│   7. Add plugins from plan
│   8. Create auth UI pages
│
├─ MIGRATING → Migration from existing auth
│   1. Audit current auth for gaps
│   2. Install better-auth alongside existing auth
│   3. Migrate routes → session logic → UI
│   4. Remove old auth library
│
└─ ADDING → Add auth to existing project
    1. Analyze project structure
    2. Install better-auth
    3. Create auth config matching plan
    4. Add route handler
    5. Run schema migrations
    6. Integrate into existing pages
    7. Add planned plugins
```

At end of implementation, guide user on next steps: OAuth credentials, env vars, testing, deployment.

---

## Installation

```bash
npm install better-auth
```

Scoped packages (as needed):
| Package | Use case |
|---------|----------|
| `@better-auth/passkey` | WebAuthn/Passkey |
| `@better-auth/sso` | SAML/OIDC enterprise SSO |
| `@better-auth/stripe` | Stripe payments |
| `@better-auth/expo` | React Native/Expo |

---

## Environment Variables

```env
BETTER_AUTH_SECRET=<32+ chars — generate: openssl rand -base64 32>
BETTER_AUTH_URL=http://localhost:3000
DATABASE_URL=<connection string>
# OAuth: GITHUB_CLIENT_ID, GITHUB_CLIENT_SECRET, GOOGLE_CLIENT_ID, etc.
```

Only define `baseURL`/`secret` in config if env vars are NOT set.

---

## Server Config (`lib/auth.ts`)

| Option | Notes |
|--------|-------|
| `appName` | Optional display name |
| `database` | Required — connection or adapter |
| `emailAndPassword` | `{ enabled: true }` to activate |
| `socialProviders` | `{ google: { clientId, clientSecret }, ... }` |
| `plugins` | Array of feature plugins |
| `trustedOrigins` | CSRF whitelist |
| `session.expiresIn` | Default 7 days |
| `rateLimit` | `{ enabled, window, max, storage }` |

Export types: `export type Session = typeof auth.$Infer.Session`

---

## Client Config (`lib/auth-client.ts`)

| Framework | Import |
|-----------|--------|
| React/Next.js | `better-auth/react` |
| Vue | `better-auth/vue` |
| Svelte | `better-auth/svelte` |
| Solid | `better-auth/solid` |
| Vanilla JS | `better-auth/client` |

Client plugins go in `createAuthClient({ plugins: [...] })`.

Common exports: `signIn`, `signUp`, `signOut`, `useSession`, `getSession`

---

## Route Handler

| Framework | File | Handler |
|-----------|------|---------|
| Next.js App Router | `app/api/auth/[...all]/route.ts` | `toNextJsHandler(auth)` → `export { GET, POST }` |
| Next.js Pages | `pages/api/auth/[...all].ts` | `toNextJsHandler(auth)` → default export |
| Express | Any | `app.all("/api/auth/*", toNodeHandler(auth))` |
| SvelteKit | `src/hooks.server.ts` | `svelteKitHandler(auth)` |
| Hono | Route file | `auth.handler(c.req.raw)` |

**Next.js Server Components:** Add `nextCookies()` plugin to auth config.

---

## Database Adapters

| Database | Setup |
|----------|-------|
| SQLite | Pass `better-sqlite3` or `bun:sqlite` instance directly |
| PostgreSQL | Pass `pg.Pool` instance directly |
| MySQL | Pass `mysql2` pool directly |
| Prisma | `prismaAdapter(prisma, { provider: "postgresql" })` |
| Drizzle | `drizzleAdapter(db, { provider: "pg" })` |
| MongoDB | `mongodbAdapter(db)` from `better-auth/adapters/mongodb` |

**Critical:** Config uses ORM model name (e.g., `"user"`), NOT the DB table name (e.g., `"users"`).

---

## Migrations

| Adapter | Command |
|---------|---------|
| Built-in Kysely | `npx @better-auth/cli@latest migrate` |
| Prisma | `npx @better-auth/cli@latest generate --output prisma/schema.prisma` then `npx prisma migrate dev` |
| Drizzle | `npx @better-auth/cli@latest generate --output src/db/auth-schema.ts` then `npx drizzle-kit push` |

Re-run after adding/changing plugins.

---

## Common Plugins

Import from dedicated paths for tree-shaking: `import { twoFactor } from "better-auth/plugins/two-factor"`

| Plugin | Server Import | Client Import | Purpose |
|--------|---------------|---------------|---------|
| `twoFactor` | `better-auth/plugins` | `twoFactorClient` | 2FA with TOTP/OTP |
| `organization` | `better-auth/plugins` | `organizationClient` | Teams/orgs |
| `admin` | `better-auth/plugins` | `adminClient` | User management |
| `bearer` | `better-auth/plugins` | — | API token auth |
| `passkey` | `@better-auth/passkey` | `passkeyClient` | WebAuthn |
| `magicLink` | `better-auth/plugins` | `magicLinkClient` | Passwordless email |
| `apiKey` | `better-auth/plugins` | `apiKeyClient` | API key auth |

---

## Auth UI Patterns

**Sign in:** `signIn.email({ email, password })` or `signIn.social({ provider, callbackURL })`

**Session (client):** `useSession()` → `{ data: session, isPending }`

**Session (server):** `auth.api.getSession({ headers: await headers() })`

**Protected routes:** Check session → redirect to `/sign-in` if null.

---

## Security Checklist

- [ ] `BETTER_AUTH_SECRET` set (32+ chars)
- [ ] `advanced.useSecureCookies: true` in production
- [ ] `trustedOrigins` configured
- [ ] Rate limiting enabled
- [ ] Email verification enabled
- [ ] Password reset implemented
- [ ] CSRF protection NOT disabled
- [ ] `account.accountLinking` reviewed

---

## Common Gotchas

1. **Model vs table name** — Config uses ORM model name, not DB table name
2. **Plugin schema** — Re-run CLI after adding plugins
3. **Secondary storage** — Sessions go there by default, not DB
4. **Cookie cache** — Custom session fields NOT cached, always re-fetched
5. **Change email flow** — Sends to current email first, then new email

---

## Troubleshooting

| Issue | Fix |
|-------|-----|
| "Secret not set" | Add `BETTER_AUTH_SECRET` env var |
| "Invalid Origin" | Add domain to `trustedOrigins` |
| Cookies not setting | Check `baseURL` matches domain; enable secure cookies in prod |
| OAuth callback errors | Verify redirect URIs in provider dashboard |
| Type errors after adding plugin | Re-run CLI generate/migrate |

---

## Resources

- [Docs](https://better-auth.com/docs)
- [Options Reference](https://better-auth.com/docs/reference/options)
- [Plugins](https://better-auth.com/docs/concepts/plugins)
- [CLI](https://better-auth.com/docs/concepts/cli)
- [Examples](https://github.com/better-auth/examples)
- [LLMs.txt](https://better-auth.com/llms.txt)

See also: [providers.md](references/providers.md) | [explain-error.md](references/explain-error.md)
