# Wishjar v1: Engineering Plan

Written 2026-11-10. Implements `docs/BRIEF.md`, the approved design `docs/designs/wishjar-v1.md` (with CEO decisions), the screens spec `docs/designs/wishjar-v1-screens.md` and `DESIGN.md`. Where this file says "the brief", it means all four.

## 0. Stack and versions

Checked against official docs and npm on 2026-11-10. Pin exact versions in `package.json` at scaffold time.

| Piece | Choice | Version seen | Why |
|---|---|---|---|
| Framework | Next.js App Router, TypeScript, React Server Components, Server Actions | next 16.4.6, react 19.3.1 | Confirmed stack. Server Components let sealed data never leave the server. |
| Styling | Tailwind CSS v4 with DESIGN.md tokens as CSS variables | tailwindcss 4.3.5 | Confirmed stack. |
| Database | Supabase Postgres, accessed **only from the server** with the new secret API key (`sb_secret_…`) | @supabase/supabase-js 2.119.0 | Supabase is retiring legacy anon/service_role keys by end of 2026; start on the new keys. |
| Email | Resend Node SDK, with `Idempotency-Key` on every send | resend 6.37.0 | Resend dedupes the same key for 24 hours. |
| Scheduler | Supabase `pg_cron` + `pg_net`, calling a protected Next.js route hourly | n/a | Vercel Hobby cron runs only once a day (±59 min). `pg_net` doesn't retry, so the job itself catches up. |
| Validation | zod | 4.6.7 | One schema per action, shared with forms. |
| Dates | Luxon (IANA zones, explicit DST handling) | latest 3.x | Clear `startOf('day')` semantics in a zone, well-tested. |
| Tests | Vitest (unit, integration against a local Supabase), Playwright (E2E, phone viewport) | vitest 5.1.0, @playwright/test 1.65.0 | Fake clock for all time logic. |
| Hosting | Vercel | n/a | Confirmed stack. |

Sources: [Supabase API keys](https://supabase.com/docs/guides/api/api-keys), [migrating to publishable and secret keys](https://supabase.com/docs/guides/getting-started/migrating-to-new-api-keys), [Scheduling with pg_cron + pg_net](https://supabase.com/docs/guides/functions/schedule-functions), [Resend idempotency keys](https://resend.com/docs/dashboard/emails/idempotency-keys), [Vercel cron usage and pricing](https://vercel.com/docs/cron-jobs/usage-and-pricing), [Next.js blog](https://nextjs.org/blog).

### Architecture

```
 phone browser ── HTML + server actions ──> Next.js on Vercel ──(secret key)──> Supabase Postgres
   no Supabase keys, no wish data                │   server-only data layer        RLS on, no policies
   for sealed occurrences                         │                                 (deny all to anon)
                                                  ├──> Resend (emails, idempotency keys)
   Supabase pg_cron ── pg_net POST (secret) ──> /api/jobs/hourly
```

The browser never talks to Supabase. Every read and write goes through server components or server actions using a `server-only` data module. This is the core of how sealed content stays hidden (section 5).

## 1. Data model

All tables have Row Level Security **enabled with no policies**, so the publishable key (unused) and any leaked anon access see nothing. The server uses the secret key, which bypasses RLS. Times are `timestamptz` in UTC.

### `jar`
| Column | Type | Notes |
|---|---|---|
| id | uuid pk default gen_random_uuid() | internal; may appear in private URLs after token exchange (section 3) |
| public_token | text unique not null | 22-char base64url, plaintext (it is meant to be shared) |
| display_name | text not null check (char_length between 1 and 60) | |
| birth_day | smallint not null check 1–31 | |
| birth_month | smallint not null check 1–12 | plus a check rejecting impossible pairs (31 Apr, 30/31 Feb); 29 Feb allowed |
| timezone | text not null | IANA name, validated against `Intl.supportedValuesOf('timeZone')` |
| visibility | enum `private` / `wall` default `private` | |
| allow_anonymous | boolean not null default false | |
| email | citext null | backup email |
| email_confirmed_at | timestamptz null | |
| origin | enum `self` / `gift` | |
| claimed_at | timestamptz null | null = unclaimed gift jar; set at creation for `self` |
| created_at | timestamptz default now() | |

Indexes: unique `public_token`; `email` where `email_confirmed_at is not null` (recovery lookup).

### `jar_occurrence`
One row per birthday occurrence that has wishes or is current. Created lazily when the first wish for that occurrence arrives, or when computing the current state.
| Column | Type | Notes |
|---|---|---|
| id | uuid pk | |
| jar_id | uuid fk → jar on delete cascade | |
| label_year | smallint not null | calendar year of the birthday date (display only) |
| unlock_at | timestamptz not null | local 00:00 of the birthday in the jar's timezone |
| late_until | timestamptz not null | local 00:00 of birthday + 8 days (end of day 7) |
| unlocked_frozen | boolean not null default false | set true the first time anyone observes `now() >= unlock_at`; frozen rows are never re-resolved |

Indexes: unique `(jar_id, label_year)`; `(jar_id, unlock_at)`; `(unlock_at)` for the email job.

Re-resolution (fix 2, CEO D11): when birthday or timezone changes, one SQL function recomputes `unlock_at` and `late_until` for rows where `unlocked_frozen = false and unlock_at > now()`, using the pure functions in section 4 (computed in the app, written in one transaction). If the new date's window already contains `now()`, the row's `unlock_at` becomes the past instant and it unlocks immediately, with no birthday email (D11).

### `wish`
| Column | Type | Notes |
|---|---|---|
| id | uuid pk | |
| jar_id | uuid fk → jar on delete cascade | denormalized for fast counts |
| occurrence_id | uuid fk → jar_occurrence on delete cascade | |
| kind | enum `text` (later `photo`, `voice`) default `text` | |
| body | text not null check (char_length between 1 and 1000) | rendered as text only |
| wisher_name | text null check (char_length <= 60) | null = anonymous; nothing else about the wisher is stored |
| written_mode | enum `private` / `wall` | jar visibility at send time (decision 5) |
| deleted_at | timestamptz null | soft delete for the 6-second undo (DR-D6) |
| created_at | timestamptz default now() | |

Indexes: `(occurrence_id) where deleted_at is null`; `(jar_id, created_at desc)`. A purge job hard-deletes rows with `deleted_at < now() - interval '2 minutes'`.

### `wish_attachment` (empty in v1)
| Column | Type | Notes |
|---|---|---|
| id | uuid pk | |
| wish_id | uuid fk → wish on delete cascade | |
| kind | enum `photo` / `voice` | |
| storage_path | text not null | object in a **private** Supabase Storage bucket, never a public URL |
| mime_type | text not null | |
| byte_size | integer not null | |
| duration_ms | integer null | voice only |
| width, height | integer null | photo only |
| created_at | timestamptz default now() | |

**How photos and voice notes get added later, without a rewrite:**
- A wish keeps its text, and gains 0..n attachments. `wish.kind` lets a wish be photo-only or voice-only later, by relaxing the `body` check for those kinds.
- Files go to a private bucket under `jar/{jar_id}/{wish_id}/{attachment_id}`, uploaded through a server action that returns a short-lived signed upload URL.
- Sealing applies unchanged: signed download URLs are generated only for wishes in unlocked occurrences, by the same visibility function as text (section 5).
- Deleting a jar must also delete its storage folder. The delete action gets one extra step; the data model doesn't change.

### `link_token`
All secret links share one table.
| Column | Type | Notes |
|---|---|---|
| id | uuid pk | |
| jar_id | uuid fk → jar on delete cascade | |
| purpose | enum `private`, `claim`, `gift_manage`, `recovery`, `email_confirm` | |
| token_hash | bytea not null unique | HMAC-SHA-256 (section 3) |
| expires_at | timestamptz null | null for `private`, `claim`, `gift_manage`; 1 h recovery; 24 h email confirm |
| used_at | timestamptz null | single-use purposes |
| revoked_at | timestamptz null | replaced private links, replaced claim links |
| created_at | timestamptz default now() | |

Indexes: unique `token_hash`; partial unique `(jar_id) where purpose = 'private' and revoked_at is null` so a jar has at most one live private link.

### `deleted_jar`
| public_token text pk | deleted_at timestamptz |
Lets a deleted jar's public link show "This jar isn't here any more" (HTTP 410) instead of "not found", and makes sure the token is never reused. Holds no personal data.

### `email_send`
| Column | Type | Notes |
|---|---|---|
| id | uuid pk | |
| jar_id | uuid fk → jar on delete cascade | |
| occurrence_id | uuid null fk → jar_occurrence on delete cascade | for `birthday` |
| kind | enum `birthday`, `recovery`, `email_confirm` | (`nudge` added with Phase 4 later) |
| status | enum `claimed`, `sent`, `failed` | |
| attempts | smallint default 0 | |
| provider_message_id | text null | Resend id |
| claimed_at, sent_at | timestamptz | |

Index: unique `(jar_id, occurrence_id, kind)` where `kind = 'birthday'`. This row is the "exactly once" lock (section 7).

### `rate_limit`
| key_hash bytea | window_start timestamptz | count int | expires_at timestamptz |
Primary key `(key_hash, window_start)`. Key = HMAC of e.g. `wish:ip:{ip}:jar:{jar_id}`. Rows expire after 1 hour (D18) and are purged hourly. IPs are never stored in plain text.

### `event_log`
| id bigint identity | jar_id uuid null | name text | props jsonb | created_at |
Names: `public_view` (props: `source` from `?s=`), `share_tap`, `private_view`, `wish_sent` (props: `sealed` true/false, `form_ms`, `load_to_submit_ms`), `invite_tap` (`start_mine` / `start_friend`), `reveal_seen`, `jar_created` (`origin`). `props` never holds names, emails, IPs, tokens or wish text. Index `(jar_id, name, created_at)`.

### `job_run`
| id | job text | started_at | finished_at | stats jsonb | error text null |
Used for the "job didn't run" alert (section 8).

## 2. Routes and server actions

Secret tokens are carried in the URL **fragment** (`#…`), which browsers never send to servers. A tiny client page posts the token to a server action that verifies it, sets an HttpOnly cookie and redirects. This keeps secret tokens out of Vercel request logs, Referer headers and analytics (see section 3 and decision Q1).

### Pages (GET)

| Route | Who can open | Shows |
|---|---|---|
| `/` | anyone | S1 home; reads `wj_jar_*` cookies to list remembered jars (DR-D7) |
| `/new`, `/new/friend` | anyone | S2 / S2b |
| `/new/done` | the creator, within 15 min (encrypted flash cookie holds the new links) | S3 / S3b; expired → S17 |
| `/j/[publicToken]` | anyone with the public link | S4 / S5 / S6 and the wish form; 404 unknown, 410 deleted |
| `/k` (key exchange) with `#purpose.token` | anyone holding a secret link | client page that posts the fragment to `exchangeToken` |
| `/m/[jarId]` | browser holding a valid `wj_jar_{jarId}` cookie | S9 / S10 / S11; no cookie → S17 "Lost your secret link?" |
| `/m/[jarId]/settings` | same cookie | S12 |
| `/claim` | after exchange of a `claim` token (cookie `wj_claim`) | S15 |
| `/gift` | after exchange of a `gift_manage` token | S15m |
| `/recover` | anyone | S14a / S14b |
| `/recover/new` | after exchange of a `recovery` token | S14c (button "Get my new link"; nothing is consumed on GET) |
| `/confirm` | after exchange of an `email_confirm` token | S16 (button "Confirm my email") |
| `/api/og/[publicToken]` | anyone (chat-app crawlers) | timeless preview image via `next/og` |
| `/api/jobs/hourly` (POST) | pg_cron only: header `x-wishjar-job: <secret>` compared in constant time | email job, purges |

All pages except `/` send `X-Robots-Tag: noindex, nofollow` and `<meta name="robots" content="noindex">`; `robots.txt` disallows `/j/`, `/m/`, `/k`. Every response sets `Referrer-Policy: no-referrer`. Private pages also set `Cache-Control: private, no-store`.

### Server actions

Server Actions are public HTTP endpoints. Next.js checks the `Origin` header against the host for them, and every action below also validates input with zod and re-checks authority itself.

| Action | Who may call | Does |
|---|---|---|
| `createJar(form)` | anyone (rate-limited 10/IP/hour) | validates, creates jar + private token (+ email confirm token, email), sets flash cookie, redirects `/new/done` |
| `createGiftJar(form)` | anyone (same limit) | creates jar with `claimed_at = null`, claim + gift_manage tokens; no private token yet |
| `rememberJar(jarId, remember)` | holder of that jar's cookie | sets the cookie's lifetime to 400 days or session (DR-D7) |
| `exchangeToken(purpose, token)` | anyone | verifies hash, purpose, expiry, revoked/used; sets the matching HttpOnly cookie; redirects. Never consumes single-use tokens |
| `sendWish(publicToken, form)` | anyone with the public link | honeypot, rate limits, validation, assigns occurrence (section 4), inserts wish with `written_mode`, logs `wish_sent` |
| `logEvent(name, props)` | anyone | allow-listed names and props only, rate-limited |
| `deleteWish(wishId)` / `restoreWish(wishId)` | holder of the jar cookie; wish must belong to that jar | soft delete / undo within 2 minutes |
| `updateDetails(form)` | jar cookie | name, birthday, timezone; re-resolves sealed occurrences in one transaction |
| `setVisibility(mode)` / `setAllowAnonymous(bool)` | jar cookie | |
| `setEmail(email)` / `resendConfirm()` | jar cookie | issues `email_confirm` token, sends (limits from D18) |
| `forgetThisPhone()` | jar cookie | clears the cookie |
| `deleteJar(typedName)` | jar cookie; typed name must match | writes `deleted_jar`, deletes jar (cascade); later also storage folder |
| `claimJar()` | `wj_claim` cookie | single-use: marks claim used, sets `claimed_at`, mints private token, shows S3 |
| `reissueClaim()` | `wj_gift` cookie, jar unclaimed | revokes old claim token, mints a new one |
| `requestRecovery(email)` | anyone (3/email/hour, 10/IP/hour) | always returns S14b; if confirmed jars exist, emails one recovery link per jar |
| `useRecovery()` | `wj_recovery` cookie | single-use: revokes old private token, mints new one, revokes other unused recovery tokens for that jar |
| `confirmEmail()` | `wj_confirm` cookie | sets `email_confirmed_at` |

Cookies: `wj_jar_{jarId}` (HttpOnly, Secure, SameSite=Lax, Path=/, value = private token), short-lived `wj_claim` / `wj_gift` / `wj_recovery` / `wj_confirm` (15 min). On every private request the server re-verifies the cookie's token hash against `link_token`, so revoking a private link (recovery) logs out every old browser immediately.

## 3. Token scheme

- **Generation:** `crypto.randomBytes(n)` from Node, encoded base64url without padding.
  - Public token: 16 bytes = 128 bits → **22 characters**. Example shape: `/j/Vq3x9TmA0b-Lr7cKwZ1yQg`.
  - Secret tokens (private, claim, gift_manage, recovery, email_confirm): 32 bytes = 256 bits → **43 characters**, with a one-letter purpose prefix inside the fragment: `/k#p.<43 chars>` (p = private, c = claim, g = gift, r = recovery, e = email).
- **Storage:** public tokens in plain text (they are shared freely, and storing them plainly lets us look up jars). Secret tokens are never stored; only `HMAC-SHA-256(TOKEN_PEPPER, purpose + ":" + token)` as 32 bytes. `TOKEN_PEPPER` is a 32-byte random secret in Vercel env vars only (not in the database), so a database leak alone can't be used to test guesses.
- **Why HMAC, not bcrypt/argon2:** the tokens have 256 bits of randomness, so slow hashing adds nothing; a fast keyed hash allows a direct unique-index lookup.
- **Checking:** compute the HMAC of the presented token, look up `token_hash` with the purpose, then check `revoked_at is null`, `used_at is null` (single-use purposes) and `expires_at > now()`. Lookup by hash means no secret comparison happens in app code; the job secret is compared with `crypto.timingSafeEqual`.
- **Public never reveals private:** they are independent random values; nothing is derived from anything else.
- **Rotation:** recovery mints a new private token and revokes the old one in the same transaction (partial unique index enforces one live private link).
- **Where tokens appear:** only in the user's own URL fragment, the emails we send them and HttpOnly cookies. Never in server logs, analytics, error reports or Referer headers.

## 4. Birthday logic (pure functions)

One module, `lib/birthday.ts`, no I/O. Every function takes `now` as an argument (no `Date.now()` inside). Luxon does the zone math.

```ts
type Jar = { day: number; month: number; timezone: string }

// Calendar date of the birthday in a given year. 29 Feb → 28 Feb in non-leap years.
birthdayDate(jar, year): { year, month, day }

// Local 00:00 of that date in the jar's zone, as a UTC instant.
// If 00:00 doesn't exist (DST gap at midnight), the first instant of that local day.
unlockAt(jar, year): Instant

// End of the late window: local 00:00 of birthday + 8 days (end of day 7).
lateUntil(jar, year): Instant

// The occurrence a wish sent at `now` belongs to: the smallest year Y with now < lateUntil(jar, Y).
occurrenceFor(jar, now): { year, unlockAt, lateUntil }

// Sealed or visible straight away.
isSealed(occurrence, now): boolean            // now < unlockAt

// What the public page shows.
pageState(jar, now):
  | { kind: 'countdown', unlockAt, msLeft, showHoursMinutes: msLeft < 24h }
  | { kind: 'birthday_week', until: lateUntil }

// Email send window: first hourly run at or after local 08:00 on the birthday date, same local day only.
birthdayEmailWindow(jar, year): { from: Instant, to: Instant }   // 08:00 → next local 00:00

// Re-resolution after an edit (fix 2, D11).
resolveAfterEdit(oldJar, newJar, occurrence, now):
  | { action: 'keep' }                             // already unlocked: never changes
  | { action: 'move', unlockAt, lateUntil, label } // still sealed
  | { action: 'unlock_now', lateUntil, label, sendEmail: false }
```

### Test cases

All expectations are UTC instants unless noted. Day 0 = the birthday; the late window is days 0 to 7. Cases marked † must be confirmed against the bundled IANA tz data when the test is written (written by hand from current rules).

| # | Jar (day/month, zone) | `now` (UTC) | Function | Expected |
|---|---|---|---|---|
| 1 | 22 Nov, Europe/London | 2026-11-10T12:00Z | occurrenceFor / isSealed | year 2026, unlock 2026-11-22T00:00Z, sealed |
| 2 | 22 Nov, Europe/London | 2026-11-21T23:59:59Z | isSealed | sealed (1 s before unlock) |
| 3 | 22 Nov, Europe/London | 2026-11-22T00:00:00Z | isSealed / pageState | visible; birthday_week |
| 4 | 22 Nov, Europe/London | 2026-11-29T23:59:59Z | occurrenceFor | year 2026, visible (last second of day 7) |
| 5 | 22 Nov, Europe/London | 2026-11-30T00:00:00Z | occurrenceFor / pageState | year 2027, sealed; countdown to 2027-11-22T00:00Z (day 8 jump) |
| 6 | 22 Nov, Europe/London | 2026-11-21T00:30Z | pageState | countdown with hours and minutes (under 24 h) |
| 7 | 22 Nov, Europe/London | 2026-11-20T23:00Z | pageState | countdown in days (25 h left) |
| 8 | 3 Jan, Europe/London | 2026-12-20T10:00Z | occurrenceFor | year **2027** (wish sent in December belongs to January's birthday) |
| 9 | 29 Feb, Europe/London | 2026-06-01Z | occurrenceFor | 2027: unlock 2027-02-28T00:00Z |
| 10 | 29 Feb, Europe/London | 2027-03-01Z | occurrenceFor | 2028: unlock 2028-02-29T00:00Z (leap year) |
| 11 | 29 Feb, Europe/London | 2027-03-07T23:59Z | occurrenceFor | 2027 still (day 7 after 28 Feb is 7 Mar) |
| 12 | 22 Nov, Asia/Kolkata (+05:30) | any | unlockAt 2026 | 2026-11-21T18:30Z |
| 13 | 22 Nov, Asia/Kathmandu (+05:45) | any | unlockAt 2026 | 2026-11-21T18:15Z |
| 14 | 1 Jan, Pacific/Kiritimati (+14) | any | unlockAt 2027 | 2026-12-31T10:00Z |
| 15 | 1 Jan, Pacific/Pago_Pago (−11) | any | unlockAt 2027 | 2027-01-01T11:00Z |
| 16 | 1 Jan, Kiritimati vs Pago_Pago | 2027-01-01T00:00Z | isSealed | Kiritimati visible, Pago_Pago sealed (25 h apart) |
| 17 | 25 Oct, Europe/London (BST ends 25 Oct 2026) | any | unlockAt / lateUntil 2026 | unlock 2026-10-24T23:00Z (BST); lateUntil 2026-11-02T00:00Z (GMT) — window is 193 h |
| 18 | 29 Mar, Europe/London (BST starts 29 Mar 2026) | any | unlockAt / lateUntil 2026 | unlock 2026-03-29T00:00Z; lateUntil 2026-04-05T23:00Z — window is 191 h |
| 19 | 14 Mar, America/New_York (DST starts 14 Mar 2027 at 02:00) | any | unlockAt 2027 | 2027-03-14T05:00Z (midnight is still EST) |
| 20 † | DST gap at midnight, e.g. America/Santiago on its spring-forward date | any | unlockAt | first instant of that local day (01:00 local), not an error |
| 21 | 22 Nov, Asia/Kolkata | 2026-11-22T02:30Z | birthdayEmailWindow 2026 | from 2026-11-22T02:30Z (08:00 IST) to 2026-11-22T18:30Z |
| 22 | 22 Nov, Asia/Kathmandu | hourly runs at :00 | first run in window | 2026-11-22T03:00Z (08:00 NPT = 02:15Z, so the 03:00Z run sends) |
| 23 | edit: 3 Jan → 20 Dec, London | 2026-11-10Z | resolveAfterEdit (sealed) | move: unlock 2026-12-20T00:00Z, label 2026 |
| 24 | edit: 3 Jan → 5 Nov, London | 2026-11-10Z | resolveAfterEdit (sealed) | unlock_now (10 Nov is day 5), lateUntil 2026-11-13T00:00Z, sendEmail false |
| 25 | edit: 3 Jan → 1 Nov, London | 2026-11-10Z | resolveAfterEdit (sealed) | move to 2027-11-01T00:00Z (day 9 is outside the window) |
| 26 | edit on an unlocked occurrence | any | resolveAfterEdit | keep |
| 27 | timezone edit: London → Asia/Tokyo, 22 Nov | 2026-11-21T16:00Z | resolveAfterEdit | unlock_now (Tokyo midnight was 15:00Z), sendEmail false |
| 28 | invalid zone string | any | all | throws a typed `InvalidTimezone` error (validated at the edge) |

Plus property tests: for random jars, zones and instants, `unlockAt < lateUntil`, `occurrenceFor(now).lateUntil > now`, and the previous occurrence's `lateUntil <= now`.

## 5. Keeping sealed content away from the browser

1. **No browser database access.** No `NEXT_PUBLIC_SUPABASE_*` variables. The Supabase client lives in `lib/db/server.ts`, which imports `server-only`, so importing it from a client component fails the build.
2. **RLS on, no policies.** Even if a key leaked, anon/authenticated roles read nothing.
3. **One visibility gate.** All wish reads go through a database view `visible_wish` that joins `jar_occurrence` and filters `deleted_at is null and o.unlock_at <= now()` (database time, never the client's). For sealed occurrences the data layer exposes only `countSealed(jarId)`, which returns a number. There is no function that returns sealed rows to app code.
4. **Server Components render wishes.** Wish text only appears in server-rendered HTML for unlocked occurrences. Client components receive props already filtered; the reveal animation gets its list from the server after unlock.
5. **Unlock timing.** The page computes state from database `now()`. A client countdown reaching zero calls `router.refresh()`; the server decides. Changing the phone's clock does nothing.
6. **Caching.** Private pages are dynamic with `Cache-Control: private, no-store`; nothing private is statically cached or put in a shared cache.
7. **Wall.** Public reads use a second view `wall_wish` that adds `written_mode = 'wall'`, the jar's current `visibility = 'wall'` and the most recent unlocked occurrence only.
8. **Tests:** an integration test requests the private page and the public page before unlock and asserts the HTML and the RSC payload contain none of the seeded wish texts or names; the same test at unlock asserts they appear.

## 6. Rate limiting and spam

- **Store:** Postgres `rate_limit` table with one atomic function `hit_rate_limit(key_hash, limit, window_seconds) returns boolean` (insert … on conflict do update set count = count + 1 … returning count). No new vendor.
- **IP:** from Vercel's `x-forwarded-for` (first value); hashed with HMAC (`RATE_PEPPER`) before use; rows expire after 1 hour (D18).
- **Limits (starting values):**

| Action | Limit |
|---|---|
| sendWish | 5 per IP per jar per hour; 200 per jar per day |
| createJar / createGiftJar | 10 per IP per hour |
| requestRecovery, resendConfirm, setEmail | 3 per email per hour; 10 per IP per hour |
| exchangeToken | 30 per IP per 10 minutes (slows token guessing, already infeasible) |
| logEvent | 60 per IP per minute |

- **Honeypot:** a hidden `website` field. If filled, return the normal success screen and store nothing (bots learn nothing).
- **Timing check:** the form includes a signed render timestamp; submissions under 2 seconds after render are treated like the honeypot.
- **Validation:** zod length checks matching the database checks (1–1000 body, 1–60 name), trimming, rejecting control characters except newlines.
- **Escaping:** React escapes all text. `dangerouslySetInnerHTML` is banned by a lint rule. Links in wishes aren't auto-linked. Line breaks render with CSS `white-space: pre-wrap`.
- **Later, only if spam appears:** Cloudflare Turnstile on the wish form (decision Q4).

## 7. Email

### Sending
- `lib/email.ts` wraps Resend with: `from` = `Wishjar <hello@{SENDING_DOMAIN}>` (test sender `onboarding@resend.dev` in development, to your own address only), plain-text and simple HTML versions, and an `idempotencyKey` on every call.
- Templates (React Email or plain TSX strings): confirm email, recovery (one per jar), birthday. No private links except recovery's one-time link (decision 8). Birthday email links to the public page and to "Lost your secret link?".

### Recovery
1. `requestRecovery(email)`: rate-limit, look up jars with that **confirmed** email.
2. For each jar: create a `recovery` token (1 h, single-use), send an email with `/k#r.<token>`. Idempotency key `recovery:{tokenId}`.
3. Always show S14b, whatever happened, after a constant minimum response time (~400 ms) so timing doesn't reveal whether the email exists.
4. Using the link (`useRecovery`, POST from a button, never on GET, so email link scanners can't consume it): one transaction revokes the old private token, revokes the jar's other unused recovery tokens, mints a new private token, and shows it with the S3 save gate.

### Scheduled birthday email (exactly once)
- pg_cron runs every hour at minute 0: `net.http_post` to `/api/jobs/hourly` with the secret header read from Supabase Vault.
- The job selects occurrences where `birthdayEmailWindow.from <= now() < .to`, the jar has a confirmed email, the occurrence wasn't unlocked by an edit (D11), and no `email_send` row exists in `sent` state.
- For each: `insert into email_send (jar_id, occurrence_id, kind, status) values (…, 'birthday', 'claimed') on conflict do nothing returning id`. Only the run that inserts gets a row back and sends; overlapping runs skip.
- Send with Resend idempotency key `birthday:{occurrenceId}`. Resend dedupes the same key for 24 hours, so even a crash between "sent by Resend" and "marked sent" can't produce a second email.
- Success: `status = sent`, `sent_at`, `provider_message_id`. Failure: `status = failed`, `attempts + 1`; a later run the same local day retries rows with `failed` and `attempts < 3` using the same idempotency key. After the local day ends, it isn't sent late (fix 4).
- The job also purges expired `rate_limit` rows and soft-deleted wishes, writes a `job_run` row, and returns counts.
- pg_net doesn't retry a failed call; the next hourly run picks up anything missed.

## 8. Error handling and logging

- **Typed errors** in the data layer: `NotFound`, `Gone` (deleted jar), `Revoked`, `Expired`, `RateLimited`, `ValidationError`, `Conflict`. Actions map each to the exact copy in the screens spec; nothing generic leaks to the user. Unknown errors show S17 "Something went wrong" and are logged with a request id.
- **Draft safety:** every failing wish submit keeps the text (client draft in localStorage plus form state).
- **Logger:** one `lib/log.ts` writing JSON lines to stdout (Vercel runtime logs), fields `ts, level, event, request_id, jar_id, duration_ms, error_code`. A redaction step drops any key named `token|email|ip|name|body|wisher_name|cookie|authorization` and replaces any 22+ character base64url run with `[redacted]` before writing.
- **Never logged:** tokens (they're in fragments and cookies, so not in request paths either), emails, IP addresses, names, wish text. Resend message ids and jar UUIDs are fine.
- **Error reporting:** none in v1 beyond Vercel logs (decision Q5). If added later (Sentry), turn off request bodies, cookies and URL fragments.
- **Alerts:** a daily check (from the hourly job) that the previous `job_run` is under 3 hours old; if not, the next run that does fire emails the founder (`ALERT_EMAIL`). Also `email_send` rows stuck in `failed` after 3 attempts are counted in the job output and included in the alert.

## 9. Phased build plan

Your seven phases, in your order. One change of content, not order: gift jars and claim links are part of phase 3 (create), and recovery is part of phase 6 (email). The CEO decision D13 means **no real test jar is created until phase 6 is done** (email backup must exist first), so phases 1 to 6 are the path to the first test birthday.

### Phase 1: Foundation
Scaffold Next.js 16 + TypeScript + Tailwind v4 with DESIGN.md tokens; ESLint (ban `dangerouslySetInnerHTML`, ban client imports of `lib/db`); Vitest and Playwright; Supabase project (new secret key), migrations for every table above with RLS on; `lib/log.ts` with redaction; security headers (Referrer-Policy, X-Robots-Tag, CSP with no inline scripts except Next's nonce); Vercel project connected to `itsnotOJ/wishjar` with env vars; CI running lint, typecheck, unit tests on every push.
**Accept when:**
- `npm run lint && npm run typecheck && npm test` pass in CI on a push.
- The Vercel preview URL loads a placeholder home page with the cream background and Bricolage heading.
- `supabase db reset` applies all migrations; a query with the publishable key returns zero rows from every table.
- A unit test proves the logger drops `email`, `token` and a 43-character base64url string.

### Phase 2: Core rules, test-first
`lib/birthday.ts` and `lib/tokens.ts`, written against the tests in section 4 before the code.
**Accept when:**
- All 28 table cases plus the property tests pass with a fixed fake clock; case 20 is confirmed against tz data.
- Token tests: lengths 22 and 43, base64url only, HMAC lookup round-trips, a revoked/used/expired token is rejected, two tokens never collide in 1 million generations (statistical sanity).
- No function in these modules reads the system clock (lint rule or grep check in CI).

### Phase 3: Create a jar
S1 home, S2, S2b, S3, S3b, token exchange (`/k`), remember-on-this-phone cookies, claim flow S15 and gift management S15m.
**Accept when:**
- On a 360 px phone viewport (Playwright), creating a jar ends on S3 with two working links; "Go to my jar" stays disabled until Copy, Send or the tick.
- The secret link opened in a fresh browser lands on the private jar; the server request log for that flow contains no 43-character token (checked in the E2E test by capturing logs).
- With "Remember" ticked, `/` shows "Open [Name]'s jar"; after "Forget this phone" it doesn't.
- A gift jar can't be read with its management link; claiming it gives the celebrant a private link; a second claim attempt shows "This jar has been claimed"; reissuing a claim link kills the old one.
- All S2 validation messages from the screens spec appear for their inputs.

### Phase 4: Send a wish
S4, S5, S6, S7, S8; `sendWish`; occurrence assignment; honeypot, timing check and rate limits; OG preview image; event log calls (`public_view` with source, `wish_sent`, `invite_tap`, `share_tap`).
**Accept when:**
- A wisher can go from opening the link to the confirmation in under 60 seconds on a phone viewport (timed E2E).
- Named, anonymous-allowed and anonymous-not-allowed forms behave as the screens spec says, including the "anonymity switched off mid-form" message.
- A 6th wish from the same IP to the same jar within an hour shows the rate-limit banner and keeps the draft; a 6th wish to a *different* jar succeeds.
- A filled honeypot returns the success screen and stores nothing.
- Wishes sent before, during and after the late window land in the right occurrence (integration test with a fake clock).
- A message containing `<script>alert(1)</script>` renders as literal text.
- The share preview (fetched as a crawler) shows the timeless title and image, with no countdown.

### Phase 5: Read and manage
S9, S10 (reveal, skippable, reduced motion), S11 (years folded, undo delete), S12 settings including re-resolution, S13 delete jar, Wall views, `forgetThisPhone`.
**Accept when:**
- Before unlock, the private page HTML and RSC payload contain none of the seeded wish texts or names (section 5 test); at unlock they appear.
- Changing the birthday in settings moves sealed wishes per cases 23 to 27; unlocked ones never move.
- Deleting a wish then tapping Undo within 6 s restores it; waiting removes it after reload.
- Deleting the jar requires the typed name; afterwards its public link returns 410 with "This jar isn't here any more" and the token can't be reused.
- Switching to Private hides the Wall immediately; switching back re-shows only Wall-mode wishes from the latest unlocked year (D7).

### Phase 6: Email
Resend wrapper, sending domain verification, confirm email (S16), recovery (S14), birthday email job, pg_cron setup, alerts.
**Accept when:**
- Confirmation and recovery emails arrive (Resend test sender in preview, verified domain in production).
- Recovery always shows the same screen, for a real and a fake email, with similar response times.
- Using a recovery link revokes the old private link (old browser is logged out on next request); a second use of the same link shows "This link has expired".
- Triggering the hourly job twice at the same time for a due occurrence sends exactly one email (integration test with a mocked Resend counting calls, plus a check that both calls carry the same idempotency key).
- A Kathmandu jar's email goes out on the first run at or after 08:00 NPT (case 22).
- Stopping the job for 3 hours produces an alert email on the next run.
- Email links work after an email scanner has fetched them (GET never consumes a token).
**Gate:** after this phase, real test jars can be created (CEO D13).

### Phase 7: Design polish
Final hand-drawn jar SVGs (closed, open, empty), note-drop and lid animations, every empty/loading/error state from the screens spec, accessibility spec DR-D9, dark mode, copy pass against DESIGN.md's voice table.
**Accept when:**
- Every screen S1 to S17 matches the screens spec copy exactly (snapshot of visible text per screen).
- Axe reports no serious or critical issues on S1, S2, S3, S4, S7, S9, S11, S12.
- VoiceOver run: countdown isn't announced each minute; form errors are announced; the undo bar is announced.
- At 200% text size and 320 px width, no screen scrolls sideways.
- With reduced motion on, no element moves (only fades); the reveal is skippable by tap.
- Lighthouse mobile on the public jar page: performance ≥ 90, accessibility 100.
- `/design-review` on the deployed preview finds no high-severity issues.

## Trade-offs considered

| Topic | Chose | Over | Why |
|---|---|---|---|
| Where secret tokens live in URLs | Fragment + exchange to HttpOnly cookie | Token in the path | Paths end up in Vercel logs, history sync and Referer headers; the brief says no tokens in logs. Cost: the first open needs JavaScript (one tiny page). |
| Token hashing | HMAC-SHA-256 with a pepper | bcrypt/argon2 | 256-bit random tokens can't be brute-forced; a keyed fast hash allows indexed lookup. |
| DB access | Server-only with the secret key, RLS deny-all | Browser Supabase client + RLS policies | Sealing is a business rule about time; keeping all reads on the server makes "no sealed data in the browser" provable, and there are no user accounts for RLS to use. |
| Scheduler | Supabase pg_cron + pg_net hourly | Vercel Cron | Hobby cron is daily only. If you move to Vercel Pro, switch to Vercel Cron with the same route and secret. |
| Rate limiting | Postgres table | Upstash Redis, Vercel Firewall rules | No extra vendor at this scale; easy to swap later. |
| Occurrence storage | `jar_occurrence` rows with stored `unlock_at` | Computing everything from a year number | Required for birthday edits (fix 2) and for indexing the email job. |
| Single-use links | Consumed by POST from a button | Consumed on GET | Email security scanners open links automatically and would burn them. |
| Wish delete undo | Soft delete + purge after 2 minutes | Client-side delayed delete | Survives page reloads and keeps the server authoritative. |
| Date library | Luxon | date-fns-tz, native Temporal | Mature IANA and DST handling today; revisit Temporal when it's in all target runtimes. |

## Decisions for you

- **Q1. Secret links in the URL fragment** (`wishjar.app/k#p.…`). Recommended, for the no-tokens-in-logs rule. The alternative (`/m/<token>`) is simpler but puts the secret link in Vercel's logs.
- **Q2. Vercel plan.** The plan assumes Hobby and uses Supabase's scheduler. If you're on Pro (or will be), say so and the job moves to Vercel Cron.
- **Q3. Sending domain.** Still undecided. Needed before phase 6 ends (DNS records for Resend).
- **Q4. Bot protection.** Honeypot plus timing check now, Turnstile only if spam appears. Or add Turnstile from the start (a little friction for every wisher).
- **Q5. Error reporting.** Vercel logs only in v1, or add Sentry (with bodies, cookies and fragments turned off).
- **Q6. Data region.** Pick the Supabase region nearest your friends and family (for example London `eu-west-2` or US East). It can't easily be changed later.
- **Q7. Product name and address.** "Wishjar" plus the domain you'll buy; the code uses a `SITE_URL` setting, so this can be decided before deploy without code changes.
