# Wishjar: greenfield project brief

## What it is
Wishjar is a web app where a person collects birthday wishes from
friends and loved ones through one shareable link, and keeps them
forever. The celebrant's own birthday jar fills up again every
year.

## Why it matters
- Wishes today are scattered across chats, social media and
  email, and get lost or deleted.
- People want heartfelt messages that show effort, like a
  handwritten letter. Paper carries that feeling but is easily
  lost, damaged or forgotten. Wishjar keeps those messages safe,
  so the celebrant can return and relive them at any time.

Similar products are group cards that someone else organises once
for the recipient. Wishjar differs: the celebrant owns the jar,
it is permanent, and it grows yearly.

## Users
1. Celebrant (jar owner): creates a jar, shares the link, reads
   and manages wishes. No account or password.
2. Wisher: opens the link, leaves a wish. No account. Signs with
   a typed name, or stays anonymous if the celebrant allows it.

Most people arrive from a link in a chat app, on a phone. Build
mobile-first.

## Core flows
Create a jar
- Celebrant enters display name, birthday (day and month required,
  year optional, never shown publicly) and an optional email.
- Chooses visibility: Private (only they read wishes) or Wall
  (wishes are public on the jar page). Can switch later.
- Chooses whether wishers may stay anonymous. Off by default.
  Can switch later.
- Receives two links:
  - Public link: for sharing with wishers.
  - Private link: secret, for reading and managing the jar. Tell
    them clearly to save it.
- If an email was given, send them the private link, and allow
  "I lost my link" recovery to that email. Recovery issues a new
  private link and cancels the old one.

Send a wish
- Public page shows the celebrant's name, a countdown to the
  birthday, and a form: message and wisher's name.
- If the celebrant allows anonymous wishes, the wisher can choose
  to stay anonymous instead of typing a name. Otherwise a name is
  required.
- Submit, then a warm confirmation. No editing afterwards.
- On the confirmation screen, invite the wisher to create their
  own Wishjar (one tap into the create flow, easy to skip). Never
  show this before or during the wish.

Read wishes
- Private link opens the jar: wishes grouped by birthday year,
  newest year first.
- Celebrant can delete any wish, change visibility, allow or
  block anonymous wishes, edit their details, or delete the
  whole jar.

## Rules
- Sealing: wishes sent before the birthday are sealed. The
  celebrant sees how many are waiting, not their content. The jar
  unlocks at 00:00 on the birthday in the celebrant's timezone
  (capture it at creation).
- Late wishes: wishes sent on the birthday or up to 7 days after
  belong to that year and are visible straight away. After that,
  new wishes are sealed for the next birthday.
- Yearly archive: the same public link works every year. Each
  wish is stored against its birthday year.
- Wall mode: wishes appear publicly only once unlocked.
- Anonymous wishes: shown as "Anonymous" to the celebrant and on
  the wall, with no name stored. If the celebrant turns the
  option off, new wishes need a name, and anonymous wishes
  already received stay in the jar.
- 29 February birthdays unlock on 28 February in non-leap years.
- If an email exists, send one "Your jar is open" email on the
  birthday.

## Safety and privacy
- Links use long, unguessable tokens. Store the private token
  hashed. The public link must never reveal the private one.
- Rate-limit wish submissions per IP and per jar, add a honeypot
  field, cap message length (1,000 characters) and name length.
- Escape all user content. Never render it as HTML.
- Email is used only for link backup and the birthday email.
- Public pages should not be indexed by search engines.

## Look and feel
Friendly, playful and warm, with a restrained palette: soft cream
or off-white base, one warm accent colour, generous white space.
The jar is the central visual: wishes drop in as notes, the lid
stays on until the birthday, then opens. Small, gentle animations
only. Copy is short and human.

## Tech stack (confirmed)
Next.js (App Router) with TypeScript, Tailwind CSS, Supabase
(Postgres), Resend for email, deployed on Vercel. Use current
stable versions and check official docs rather than memory.

## Out of scope for version one
Accounts and passwords, photos, voice notes, video, payments or
gifts, native apps, reactions or replies, wisher notifications,
verifying a wisher's identity. Shape the data model so photos
and voice notes can be added later without a rewrite.

## Done means
A celebrant can create a jar, share the link, receive sealed
wishes, see them unlock on their birthday, and come back next
year to the same jar. A wisher can send a wish in under a minute
and is invited to start their own jar. All of it works on a
phone, with no sign-up.
