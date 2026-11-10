# Wishjar v1: Screens and States

Produced by /plan-design-review on 2026-11-10. This file is the screen-by-screen part of the plan in `docs/designs/wishjar-v1.md`. All visual rules come from `DESIGN.md` (tokens cited as `{colors.primary}` and so on). Decisions made in this review are tagged DR-D2 to DR-D11.

Conventions used below:
- **[Name]** is the celebrant's display name, **[date]** is the birthday as "22 November", **[n]** a number.
- Every page: `lang="en"`, a header with the small "Wishjar" wordmark (links home, except on public jar pages where it is plain text), `<main>`, and a footer. One `h1` per page.
- Every primary button shows a loading label ending in "…" and keeps its width. Every form keeps typed text on error.
- "Loading" on any jar page means the jar outline with no notes and no text below it (DESIGN.md: no spinners). If loading takes over 8 seconds, show the slow-network line from S17.

## Navigation map

```
 Home (S1) ──Start my jar──> Create (S2) ──> Your links (S3) ──Go to my jar──> Private jar (S9/S10/S11)
   │  └─Start one for a friend──> Create for a friend (S2b) ──> Gift links (S3b)
   │  └─Open [Name]'s jar (remembered, DR-D7) ─────────────────────────────────> Private jar
   └─Lost your secret link? ──> Recovery (S14) ──email──> Claim recovery (S14c) ──> Private jar

 Public jar link ──> Public page (S4 before / S5 birthday week / S6 Wall after unlock)
        └─Drop it in the jar──> Confirmation (S8) ──Start my jar / Start one for a friend──> S2 / S2b
        └─Lost your secret link? (footer, DR-D11) ──> S14

 Private jar ──Settings──> Settings (S12) ──Delete jar──> Delete sheet (S13)
 Gift claim link ──> Claim (S15) ──> Your links (S3, celebrant version) ──> Private jar
 Gift management link ──> Manage gift (S15m)
 Email confirm link ──> Email confirmed (S16)
 Any bad link ──> Not found / replaced / error (S17)
```

## S1. Home

**Sees (first, second, third):** the jar (lid on, three notes); the heading; one action. (DR-D2: one screen, one action, no scrolling on a 360 x 640 phone.)

**Words:**
- `h1`: "Birthday wishes, kept in one jar"
- Line: "Share one link. Friends drop in wishes. It opens on your birthday, every year."
- Primary button: "Start my jar"
- Link: "Start one for a friend"
- Footer link (muted, small): "Lost your secret link?"

**Returning celebrant (DR-D7):** if this phone remembers a jar, a secondary button "Open [Name]'s jar" appears above "Start my jar" (several remembered jars: one button each, newest first, at most 3). If the remembered jar was deleted or its link replaced, the button is removed silently on tap and S17 shows.

**States:** no loading (static page). No empty or error state.

## S2. Create a jar (for yourself)

**Sees:** jar (small, lid on), heading, five fields, one button. One screen with natural scrolling; no steps or wizard.

**Words:**
- `h1`: "Start your jar"
- Field "What should friends call you?" (placeholder none; helper "This shows on your jar.") max 60 characters.
- Field "Your birthday": two selects side by side, "Day" and "Month". Helper "We never show your birthday year. We don't ask for it."
- Timezone line under birthday: "Your jar opens at midnight [timezone city] time." with a link "Change" that reveals a searchable timezone select. Auto-filled from the phone.
- Segmented control "Who can read the wishes?": "Only me" (Private, default) / "Everyone, on a wall" (Wall). Helper changes with the choice: "Only you will see the wishes." / "Wishes appear on your jar page after your birthday."
- Toggle "Let people wish anonymously" (off). Helper: "If this is off, everyone signs their name."
- Field "Email (optional)" with helper "Only for getting back into your jar and a note on your birthday. We'll ask you to confirm it." (Phase 2; hidden in Phase 1 builds.)
- Primary button: "Make my jar" (loading "Making your jar…")

**Errors (under the field, in `{colors.error}`, announced):**
- Name empty: "Add the name friends know you by."
- Name too long: "That's a bit long. Keep it under 60 characters."
- Birthday missing: "Pick your birthday day and month."
- Impossible date (31 April, 30 Feb): "That date doesn't exist. Check the day."
- Email malformed: "That email doesn't look right."
- Server failure: banner above the button: "Something went wrong making your jar. Your details are still here, try again."

## S2b. Create a jar for a friend

Same layout as S2, with these differences:
- `h1`: "Start a jar for a friend"
- Name field: "Your friend's name" (helper "This shows on their jar.")
- Birthday field: "Their birthday"
- Timezone line: "Their jar opens at midnight [city] time." with "Change".
- Visibility and anonymous controls as S2, worded for the friend ("Only [Name]" / "Everyone, on a wall").
- No email field (the celebrant adds their own after claiming).
- Button: "Make their jar" (loading "Making their jar…")

## S3. Your two links (celebrant)

**Sees:** heading; the public link with a big Share button; the secret link with a clear "Secret" label; the save gate. (DR-D4.)

**Words:**
- `h1`: "Your jar is ready"
- Section 1 heading: "Share this with friends"
  - Link box (public link) with "Copy"
  - Primary button: "Share my jar" (opens the phone's share sheet with the text: "My birthday jar opens at midnight on [date]. Drop a wish in? [link]")
- Section 2 heading: "Your secret link"
  - Label in the link box: "Secret"
  - Line: "This is the only way back into your jar. Don't share it."
  - Link box (secret link) with "Copy" and a secondary button "Send to myself" (share sheet, text: "My Wishjar secret link, keep this safe: [link]")
  - Checkbox (ticked): "Remember my jar on this phone" (DR-D7)
  - Checkbox: "I've saved it somewhere safe"
  - If an email was given: "We've sent a confirmation to [email]. Confirm it so you can get back in if you lose this link."
- Primary button: "Go to my jar". Disabled until the person taps Copy or Send to myself on the secret link, or ticks "I've saved it somewhere safe". While disabled, the helper under it reads "Save your secret link first."

**States:** this page only renders right after creation. Refreshing it shows the same links (they're held for this session only). Opening it later (no session) shows S17 "This page has expired".

## S3b. Gift links (the friend who started it)

**Words:**
- `h1`: "[Name]'s jar is ready"
- Section 1 "Share this with friends": public link box, "Copy", primary "Share the jar" (text: "[Name]'s birthday jar opens at midnight on [date]. Drop a wish in? [link]")
- Section 2 "Give this to [Name]": claim link box with "Copy" and secondary "Send to [Name]" (share sheet, text: "I made you a birthday jar. Open this to claim it: [link]"). Line: "Only [Name] should open this. It gives them their own secret link."
- Section 3 "Keep this for yourself": gift-management link box with "Copy". Line: "If [Name] loses the link above, use this to send a new one. It can't open the jar."
- Button: "Done" (to home). Same save gate as S3 applies to the claim link (Copy, Send, or "I've saved it").

## S4. Public jar page, before the birthday

**Sees:** jar (lid on, notes inside); heading; when it opens; who will read it; the wish form (S7).

**Words:**
- `h1`: "[Name]'s birthday jar"
- Countdown line: "Opens at midnight on [date]. Get yours in before then." In the final 24 hours: "Opens in [h] h [m] min. Get yours in before then." (ticks each minute, not announced)
- Mode line: "Only [Name] will see this." (Private) or "This will appear on [Name]'s wall after their birthday." (Wall)
- Then the wish form (S7).
- Footer link: "Lost your secret link?" (DR-D11)
- No wish count is ever shown (CEO D16).

**Wall mode before the birthday:** identical to Private except the mode line. No wishes are shown (decision: Wall shows wishes only once unlocked).

**States:** loading = jar outline only. Unclaimed gift jar: identical to a normal jar.

## S5. Public jar page, birthday week (Private mode)

Same as S4 with:
- Jar drawn with the lid off.
- Line under `h1`: "It's [Name]'s birthday week. Add yours."
- Mode line unchanged.

## S6. Public jar page, Wall mode after unlock

**Sees (DR-D3):** open jar, heading, the wishes as paper notes, sticky button.
- `h1`: "[Name]'s birthday wall"
- Line: during birthday week "It's [Name]'s birthday week." After day 7: "Wishes from [Name]'s birthday on [date]."
- Wishes: notes from the most recent unlocked year written under Wall mode, newest first, wisher name or "Anonymous" below each.
- Sticky primary button at the bottom: "Add your wish". It scrolls to the form below the wall and hides while the form is on screen.
- Below the wall, the wish form (S7) with the mode line "This will appear on [Name]'s wall." (during birthday week) or "This will appear on [Name]'s wall after their next birthday." (after day 7).
- Empty wall (no Wall-mode wishes this year): "No wishes on the wall yet. Be the first." and the form directly, with no sticky button.

## S7. Wish form (inside S4, S5, S6)

**Order:** prompt chips, message, name, anonymous choice (if allowed), button.

**Words:**
- Chips (buttons): "A memory of us", "What I love about you", "My wish for you". Tapping inserts a starter: "I still think about the time we…", "What I love about you is…", "This year I hope you…", then focuses the message.
- Message field label: "Your wish" (a visible label, never placeholder-only). Counter "0 / 1000".
- Name field label: "Your name" (max 60).
- **Anonymous allowed:** checkbox "Stay anonymous". Ticking it disables and clears the name field and shows "[Name] will see this as Anonymous." Unticking restores the typed name.
- **Anonymous not allowed:** no checkbox. Name is required.
- Primary button: "Drop it in the jar" (loading "Dropping it in…")
- Hidden honeypot field (not focusable, `aria-hidden`).

**Errors:**
- Empty message: "Write a few words first."
- Message over 1,000: counter turns `{colors.error}`: "That's over 1,000 characters. Trim it a little." Typing past the limit is allowed so nothing is lost; sending is blocked.
- Name missing (anonymous not allowed or unticked): "Add your name so [Name] knows it's from you."
- Name over 60: "That's a bit long. Keep it under 60 characters."
- Rate limited: banner "Too many wishes from here right now, try again in a bit. Your wish is saved." (draft kept)
- Network or server error: banner "That didn't go through. Your wish is saved here, try again." (draft kept)
- The jar was deleted while the form was open: S17 deleted-jar copy replaces the page.
- Anonymity switched off by the celebrant while the form was open, and the wisher ticked anonymous: "[Name] now asks everyone to sign their name. Add yours to send." (checkbox removed)

**Draft:** saved on this device as they type; cleared after sending.

## S8. Confirmation, with the invite

**Sees:** the jar with the note dropping in (480ms; reduced motion: it simply appears); the confirmation; space; the invite (only here, never before or during the wish).

**Words:**
- Wish opens later (before the birthday, or after day 7): `h1` "Your wish is in." Line: "Saved for [Name]'s next birthday."
- Birthday week, Private: `h1` "Your wish is in [Name]'s jar." Line: "[Name] can read it now."
- Birthday week, Wall: `h1` "Your wish is on [Name]'s wall." Link: "See the wall"
- Invite: "Want a jar like this for your own birthday?" Primary "Start my jar". Secondary "Start one for a friend". Link "No thanks" (returns to the jar page, scrolled to the top, with the form cleared).
- No edit option (brief).

**States:** browser back from here returns to the jar page with an empty form (no double send). Double tap on "Drop it in the jar" sends once (button disables on first tap).

## S9. Private jar, sealed (before the birthday)

**Sees:** jar with lid on and notes inside; the count; the share action; settings.

**Words:**
- `h1`: "[n] wishes waiting" (1: "1 wish waiting")
- Line: "Your jar opens at midnight on [date]." Final 24 hours: "Your jar opens in [h] h [m] min."
- Primary button: "Share my jar link" (same share text as S3)
- Secondary: "Copy link"
- Count only, no names (DR-D5).
- Past years (if any) below, folded (DR-D10): "2025 · 14 wishes"
- Footer of the page: link "Settings"

**Empty (0 wishes):** `h1` "Your jar is ready". Line: "No wishes yet. Share your link and they'll start dropping in." Same buttons.

**Unconfirmed email (Phase 2):** a quiet line above Settings: "Confirm [email] so you can get back in if you lose your link. Send again"

## S10. The reveal (first visit after unlock)

Plays once per birthday occurrence, on the first private-view visit after midnight.
- Lid lifts and tilts (700ms), notes rise out one by one (90ms apart) and become the list, then the heading fades in.
- `h1`: "Happy birthday, [Name]."
- Line: "[n] wishes were waiting for you." (1: "1 wish was waiting for you.")
- Text link at the top right during the animation: "Skip" (DR-D9). Tapping anywhere also skips.
- Reduced motion: the open jar, heading and list appear with a 150ms fade.
- Zero wishes at unlock: no animation. `h1` "Happy birthday, [Name]." Line: "Your jar is empty this year, but your link still works all week. Share it and wishes will show up here straight away." Primary "Share my jar link".

## S11. Private jar, open

**Sees:** open jar (small); this year's wishes as notes; older years folded.
- `h1`: "Happy birthday, [Name]." during birthday week; after day 7: "Your jar". During birthday week a line: "New wishes show up here straight away until [date + 7]."
- Year heading: "[year]" with "[n] wishes" on the right.
- Each note: the wish, then the name or "Anonymous", then a small "Delete" text button (muted). Delete removes the note with a fade and shows the undo bar "Wish deleted. Undo" for 6 seconds (DR-D6).
- After day 7, the next year's sealed count shows above this year's wishes: "[n] wishes waiting for your next birthday" (or nothing when 0), with "Share my jar link".
- Past years folded (DR-D10): rows "2025 · 14 wishes", unfold on tap, one at a time is fine.
- Empty open year (all deleted): "No wishes here. Wishes sent before your next birthday will wait for you."

**Errors:** delete fails: the note returns and the bar says "Couldn't delete that wish. Try again."

## S12. Jar settings

**Sees:** a single list of plainly labelled controls; changes save immediately with a small "Saved" confirmation next to the control.
- `h1`: "Settings"
- "Your name" field (60 max), "Your birthday" (day and month), "Timezone" (city). Helper under birthday: "Wishes already opened stay where they are. Waiting wishes move to the new date."
- "Who can read the wishes?" segmented: "Only me" / "Everyone, on a wall". Helper for Wall: "Only wishes sent while your jar is on Wall mode appear on the wall." Helper for Only me: "The wall is hidden straight away."
- "Let people wish anonymously" toggle. Helper when turning off: "New wishes will need a name. Anonymous wishes you've had stay in your jar."
- "Email backup": shows the email and status ("Confirmed" / "Not confirmed yet · Send again" / "None · Add email").
- "Add to my calendar" (deferred to TODOS.md, not in v1, CEO D14; do not show).
- "Forget this phone" text button (DR-D7): "This phone won't remember your jar. You'll need your secret link." Confirmed by the change itself, with an undo bar "Forgotten. Undo".
- Destructive section at the bottom, separated by space: "Delete jar" (text button in `{colors.error}`).

**Errors:** save fails: the control reverts and shows "Couldn't save that. Try again." Name empty: "Your jar needs a name."

## S13. Delete confirmations

**Delete jar (bottom sheet):**
- Heading: "Delete [Name]'s jar?"
- Text: "This deletes the jar and every wish in it, for good. Your public link will stop working. Type [Name] to confirm."
- Field label: "Type [Name]"
- Destructive button "Delete jar for good" (`{colors.error}` background, white text 6.1:1), enabled only when the text matches (case-insensitive, trimmed). Secondary "Keep my jar".
- After deleting: a plain page, `h1` "Your jar is deleted", line "Thanks for using Wishjar.", link "Start a new jar".
- Error: "Couldn't delete the jar. Nothing has changed. Try again."

**Delete one wish:** no dialog. Undo bar for 6 seconds (DR-D6), as in S11.

## S14. Lost link recovery

**S14a Request:**
- `h1`: "Get back into your jar"
- Line: "Enter the email you added to your jar. We'll send a new secret link for each jar on that email."
- Field "Email". Button "Send me a new link" (loading "Sending…")
- Below, muted: "No email on your jar? Unfortunately we can't recover it without one."
- Error: malformed email "That email doesn't look right." Rate limited: "We've sent a few already. Check your inbox, or try again in an hour."

**S14b Sent (always the same, whether or not a jar exists):**
- `h1`: "Check your email"
- Line: "If [email] has a jar, we've sent a link. It works once, for one hour."
- Link: "Didn't get it? Send again" (respects limits).

**S14c Claim recovery link (from the email):**
- Valid: `h1` "Here's your new secret link", then the S3 secret-link section (link box, Copy, Send to myself, remember checkbox, save gate). Line: "Your old secret link has stopped working." Button "Go to my jar".
- Expired or used: `h1` "This link has expired". Line: "Recovery links work once, for one hour." Button "Send a new one" (to S14a).

## S15. Gift claim and management

**S15 Claim link (celebrant opens it):**
- Unclaimed: jar with lid on. `h1` "A birthday jar for you, [Name]". Line: "Someone who loves you started this. Claim it to get your own secret link." Primary "Claim my jar". Then S3 (celebrant version, with the optional email field shown above the save gate: "Add your email to back up your link").
- Already claimed: `h1` "This jar has been claimed". Line: "If it's yours and you've lost your link, you can get back in by email." Link "Lost your secret link?"
- Replaced (a newer claim link was issued): `h1` "This link was replaced". Line: "Ask the person who made your jar for the new one."

**S15m Gift management link (friend):**
- Before claim: `h1` "[Name]'s jar". Line: "[Name] hasn't claimed it yet." Primary "Make a new claim link" (shows a new claim link box with Copy and "Send to [Name]"; the old claim link stops working). No read, edit or delete.
- After claim: `h1` "[Name] has their jar". Line: "This link doesn't do anything now. Thanks for starting it."

## S16. Email confirmation landing

- Success: `h1` "Email confirmed". Line: "If you ever lose your secret link, we can send a new one to [email]." Link "Back to Wishjar".
- Expired (over 24 hours): `h1` "This link has expired". Line: "Open your jar's settings to send a new one."

## S17. Not found, deleted, replaced, errors

All use the jar drawn open and empty.
- Unknown or mistyped public link: `h1` "We can't find this jar". Line: "Check the link, or ask whoever sent it." Link "Start your own jar".
- Deleted jar: `h1` "This jar isn't here any more". Line: "Its owner has deleted it." Link "Start your own jar".
- Old private link after recovery: `h1` "This secret link was replaced". Line: "A new one was sent to the email on this jar." Link "Lost your secret link?"
- Unknown private or claim link: same as unknown public link.
- Expired links page (S3 refreshed later): `h1` "This page has expired". Line: "Your jar is safe. Use your secret link to open it." Link "Lost your secret link?"
- General server error: `h1` "Something went wrong". Line: "It's us, not you. Try again in a moment." Button "Try again".
- Offline: banner on any page "You're offline. We'll keep what you've typed."
- Slow network (over 8 seconds loading): under the empty jar, "Still loading. Your connection might be slow."
- All error pages return the right HTTP status (404, 410 for deleted, 500) and `noindex`.

## Interaction state table

```
 FEATURE            | LOADING            | EMPTY                          | ERROR                              | SUCCESS                  | PARTIAL
 -------------------|--------------------|--------------------------------|------------------------------------|--------------------------|------------------------
 Create jar (S2)    | "Making your jar…" | n/a                            | field errors; banner, data kept    | S3 links                 | email failed to send: S3 says "We couldn't send the confirmation. Try again from settings."
 Links (S3/S3b)     | n/a                | n/a                            | copy fails: "Couldn't copy. Press and hold the link to copy it." | "Copied" 2s | gate not met: button disabled + helper
 Public page        | jar outline        | Wall: "No wishes on the wall yet. Be the first." | S17                               | S4/S5/S6                 | n/a
 Wish form (S7)     | "Dropping it in…"  | chips + blank box              | field errors; rate limit; network  | S8                       | anonymity turned off mid-form
 Private sealed     | jar outline        | "No wishes yet…"               | S17                                | "[n] wishes waiting"     | n/a
 Reveal (S10)       | jar outline        | empty-year copy, no animation  | S17                                | "Happy birthday, [Name]."| skipped animation
 Private open (S11) | jar outline        | "No wishes here…"              | delete failed, note restored       | notes by year            | past years folded
 Settings (S12)     | control disabled   | email "None · Add email"       | "Couldn't save that. Try again."   | "Saved"                  | email "Not confirmed yet"
 Delete jar (S13)   | "Deleting…"        | n/a                            | "Couldn't delete the jar…"         | "Your jar is deleted"    | name not matching: button disabled
 Recovery (S14)     | "Sending…"         | n/a                            | malformed; rate limited            | S14b (same every time)   | link expired: S14c expired
 Claim (S15)        | jar outline        | n/a                            | claimed / replaced                 | S3 celebrant             | n/a
```

## Journey storyboard

```
 STEP | WHO      | DOES                          | FEELS                 | SUPPORTED BY
 -----|----------|-------------------------------|-----------------------|-----------------------------------------
 1    | Wisher   | Taps link in group chat       | curious               | timeless preview; jar + name first (S4)
 2    | Wisher   | Sees "opens at midnight"      | let in on a secret    | countdown line, deadline copy (CEO D17)
 3    | Wisher   | Writes                        | a little stuck → warm | prompt chips (S7)
 4    | Wisher   | Sends                         | done, kind            | note drops in, "Your wish is in." (S8)
 5    | Wisher   | Sees invite                   | "I want one"          | invite only after sending (S8)
 6    | Celebrant| Creates jar                   | hopeful, a bit shy    | plain form, friend option (S2/S2b)
 7    | Celebrant| Saves secret link             | safe                  | save gate, remember on phone (S3)
 8    | Celebrant| Checks back                   | anticipation          | count only, no names (S9, DR-D5)
 9    | Celebrant| Opens on the birthday         | delight (the moment)  | reveal once, skippable (S10)
 10   | Celebrant| Rereads next year             | nostalgia             | years folded like drawers (S11, DR-D10)
```
Time horizons: 5 seconds (jar, name, opening date); 5 minutes (wish sent, or jar made and shared); 5 years (yearly drawers of wishes, same link).

## Responsive and accessibility (DR-D9)

- **Phone (320 to 599px):** single column, 20px gutters, primary buttons full width; the sticky button respects the safe area.
- **Tablet and desktop (600px+):** the same column centred at 480px max, 48px top space, the jar 20% larger. Buttons stay full column width. No side-by-side layouts. Hover states mirror the pressed state colours.
- **Screen readers:** one `h1` per page. The jar SVG is hidden; state is given in words (for example, a visually hidden "[Name]'s jar, [n] wishes waiting"). The countdown isn't a live region; only state changes (opened, birthday week) are announced. Field errors use `aria-describedby` and `aria-invalid` and are announced on submit. The undo bar and "Copied" use a polite live region. Notes are list items, each with the wish then the name.
- **Keyboard:** logical tab order that matches the visual order. A visible 2px focus ring offset 2px. Escape closes sheets; focus moves into the sheet when opened and returns to its trigger. Chips are buttons.
- **Large text:** works at 200% text size, with no clipped text and no horizontal scrolling at 320px.
- **Landmarks:** header, main and footer on every page.
- **Motion:** DESIGN.md reduced-motion rules; the reveal can be skipped with a tap or the Skip link.
- **Touch:** 44px minimum targets, 52px buttons, 8px between chips.

## NOT in scope
- Email templates (confirmation, recovery, birthday) visual design: content rules exist in the design doc; layout to be designed with Phase 2.
- "Add to my calendar" and the year-two nudge: deferred to TODOS.md (CEO D14).
- Final jar artwork: direction set in DESIGN.md; drawing it is a design task, not a screen decision.

## What already exists
- `DESIGN.md`: tokens, components, states list, voice table, motion and reduced-motion rules.
- `docs/designs/wishjar-v1-wireframe.png`: wish form, confirmation and celebrant view (consistent with S4, S8, S9).
- `docs/designs/design-system/preview.html`: rendered wish form, confirmation and reveal.

## Design review decisions

| ID | Decision | Answer |
|---|---|---|
| DR-D2 | Home page is one screen with one action (S1) | A |
| DR-D3 | Wall after unlock: wall first, sticky "Add your wish" (S6) | A |
| DR-D4 | Save gate on the links screen (S3) | A |
| DR-D5 | Before the birthday the celebrant sees the count only, no names (S9) | A |
| DR-D6 | Deleting a wish uses a 6-second undo, no dialog (S11, S13) | B |
| DR-D7 | Remember the jar on this phone, ticked by default; "Forget this phone" in settings (S1, S3, S12) | A |
| DR-D8 | Link box, undo bar and sticky button added to DESIGN.md | A |
| DR-D9 | Accessibility spec adopted (this section) | A |
| DR-D10 | Past years folded, newest open (S9, S11) | A |
| DR-D11 | "Lost your secret link?" on home and every public jar page (S1, S4) | A |

## Implementation Tasks
Synthesized from this review's findings. Each task derives from a specific finding above.

- [ ] **T1 (P1, human: ~3d / CC: ~3h)** — screens — Build S1 to S17 to this spec, with exact copy
  - Surfaced by: Pass 1, Pass 2 — 17 screens previously unspecified
  - Files: to be determined (Next.js app routes)
  - Verify: walk every screen at 360px against this file; every listed state reachable
- [ ] **T2 (P1, human: ~4h / CC: ~20min)** — links screen — Implement the save gate and "remember on this phone"
  - Surfaced by: Pass 2 DR-D4, Pass 3 DR-D7
  - Files: to be determined
  - Verify: "Go to my jar" stays disabled until Copy, Send or the tick; home shows "Open [Name]'s jar" after a remembered create; "Forget this phone" removes it
- [ ] **T3 (P1, human: ~1d / CC: ~1h)** — accessibility — Implement and test the DR-D9 spec
  - Surfaced by: Pass 6 DR-D9
  - Files: to be determined
  - Verify: VoiceOver and TalkBack pass on S4, S7, S9, S10; 200% text at 320px; keyboard-only run of create → share → wish
- [ ] **T4 (P2, human: ~3h / CC: ~15min)** — private jar — Undo bar for wish delete and folded past years
  - Surfaced by: Pass 2 DR-D6, Pass 7 DR-D10
  - Files: to be determined
  - Verify: delete then Undo restores the note; delete then wait 6s removes it on reload; older years fold and unfold
- [ ] **T5 (P2, human: ~2h / CC: ~10min)** — Wall — Wall-first layout with sticky add button
  - Surfaced by: Pass 1 DR-D3
  - Files: to be determined
  - Verify: sticky button hides when the form is visible; empty wall shows form directly

## Design Review Completion Summary

```
  +====================================================================+
  |         DESIGN PLAN REVIEW — COMPLETION SUMMARY                    |
  +====================================================================+
  | System Audit         | DESIGN.md present; full-product UI scope    |
  | Step 0               | 4/10 initial; all 7 passes, every screen    |
  | Pass 1  (Info Arch)  | 4/10 → 9/10 after fixes                     |
  | Pass 2  (States)     | 3/10 → 9/10 after fixes                     |
  | Pass 3  (Journey)    | 5/10 → 9/10 after fixes                     |
  | Pass 4  (AI Slop)    | 8/10 → 9/10 after fixes                     |
  | Pass 5  (Design Sys) | 7/10 → 9/10 after fixes                     |
  | Pass 6  (Responsive) | 5/10 → 9/10 after fixes                     |
  | Pass 7  (Decisions)  | 2 resolved, 0 deferred                      |
  +--------------------------------------------------------------------+
  | NOT in scope         | written (3 items)                           |
  | What already exists  | written                                     |
  | TODOS.md updates     | 0 items proposed                            |
  | Approved Mockups     | 0 generated (no image API key), 0 approved  |
  | Decisions made       | 10 added to plan                            |
  | Decisions deferred   | 0                                           |
  | Overall design score | 3/10 → 9/10                                 |
  +====================================================================+
```
Remaining gap keeping passes at 9 rather than 10: no rendered mockups of the new screens (designer unavailable here). Run `/design-review` on the built screens.

### Unresolved Decisions
None.

## GSTACK REVIEW REPORT

| Review | Trigger | Why | Runs | Status | Findings |
|--------|---------|-----|------|--------|----------|
| CEO Review | `/plan-ceo-review` | Scope & strategy | 1 | CLEAR | mode: HOLD_SCOPE, 0 critical gaps |
| Outside Review | Codex (not installed) | Independent 2nd opinion | 0 | skipped | no completed external review |
| Eng Review | `/autoplan` | Architecture & tests (required) | 0 | not run | — |
| Design Review | `/plan-design-review` | UI/UX gaps | 1 | CLEAR | score: 4/10 → 9/10, 10 decisions |
| DX Review | `/plan-devex-review` | Developer experience gaps | 0 | not run | — |

- **OUTSIDE COVERAGE:** design outside voices declined by the founder (skipped); Codex not installed.
- **VERDICT:** CEO + DESIGN CLEARED. Eng review required.

NO UNRESOLVED DECISIONS
