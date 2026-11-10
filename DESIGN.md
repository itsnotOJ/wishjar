---
# gstack: design-md-format=spec
name: Wishjar
description: A hand-drawn jar on warm cream paper, friendly and calm, saving its one bright moment for the birthday.
colors:
  background: "#FBF6EC"
  surface: "#FFFDF8"
  text: "#2B2420"
  text-muted: "#6B5D52"
  border: "#9A8878"
  hairline: "#E7DCCB"
  primary: "#C23B5E"
  on-primary: "#FFFFFF"
  primary-pressed: "#A12E4C"
  primary-text: "#A12E4C"
  primary-soft: "#F8E1E4"
  note-white: "#FFFDF8"
  note-blush: "#F8E1E4"
  note-butter: "#FBEBD0"
  focus: "#2B2420"
  success: "#2E6B45"
  warning: "#8A5300"
  error: "#B3261E"
  dark-background: "#1F1A17"
  dark-surface: "#2A2420"
  dark-text: "#F5EDE1"
  dark-text-muted: "#BCAE9F"
  dark-border: "#7D6E61"
  dark-hairline: "#3A322C"
  dark-primary-text: "#EE8AA3"
  dark-primary-soft: "#3A2A2E"
  dark-note-white: "#2F2925"
  dark-note-blush: "#3A2A2E"
  dark-note-butter: "#3A3226"
  dark-focus: "#F5EDE1"
  dark-success: "#7CC49A"
  dark-warning: "#E3B66B"
  dark-error: "#F2938B"
typography:
  display:
    fontFamily: Bricolage Grotesque
    fontWeight: 700
    fontSize: clamp(1.875rem, 7.5vw, 2.5rem)
    lineHeight: 1.08
    letterSpacing: -0.015em
  heading:
    fontFamily: Bricolage Grotesque
    fontWeight: 700
    fontSize: 1.375rem
    lineHeight: 1.2
  body:
    fontFamily: Figtree
    fontWeight: 400
    fontSize: 1.0625rem
    lineHeight: 1.55
  label:
    fontFamily: Figtree
    fontWeight: 500
    fontSize: 0.875rem
    lineHeight: 1.4
  button:
    fontFamily: Figtree
    fontWeight: 700
    fontSize: 1.0625rem
  small:
    fontFamily: Figtree
    fontWeight: 400
    fontSize: 0.8125rem
    fontFeature: tnum
rounded:
  note: 3px 12px 12px 12px
  sm: 6px
  md: 12px
  lg: 20px
  full: 9999px
spacing:
  xs: 4px
  sm: 8px
  md: 12px
  base: 16px
  gutter: 20px
  lg: 24px
  xl: 32px
  2xl: 48px
  3xl: 64px
components:
  button-primary:
    backgroundColor: "{colors.primary}"
    textColor: "{colors.on-primary}"
    rounded: "{rounded.md}"
    minHeight: 52px
  button-primary-pressed:
    backgroundColor: "{colors.primary-pressed}"
  button-secondary:
    borderColor: "{colors.text}"
    textColor: "{colors.text}"
    rounded: "{rounded.md}"
    minHeight: 52px
  link:
    textColor: "{colors.primary-text}"
  input:
    backgroundColor: "{colors.surface}"
    borderColor: "{colors.border}"
    rounded: "{rounded.md}"
    minHeight: 48px
  prompt-chip:
    backgroundColor: "{colors.primary-soft}"
    borderColor: "{colors.primary}"
    textColor: "{colors.text}"
    rounded: "{rounded.full}"
    minHeight: 40px
  wish-note:
    textColor: "{colors.text}"
    rounded: "{rounded.note}"
  jar:
    strokeColor: "{colors.text}"
    strokeWidth: 2.6px
  link-box:
    backgroundColor: "{colors.surface}"
    borderColor: "{colors.border}"
    rounded: "{rounded.md}"
    minHeight: 48px
  link-box-secret-label:
    textColor: "{colors.primary-text}"
  undo-bar:
    backgroundColor: "{colors.text}"
    textColor: "{colors.background}"
    rounded: "{rounded.md}"
    minHeight: 48px
  button-sticky:
    backgroundColor: "{colors.primary}"
    textColor: "{colors.on-primary}"
    rounded: "{rounded.md}"
    minHeight: 52px
---

# Wishjar

## Overview

**Creative North Star:** a jar drawn in one pen line on warm paper. Everything stays quiet so that the lid coming off on the birthday is the moment people remember.
**Product context:** Wishjar collects birthday wishes through one shareable link into a permanent jar that opens at midnight on the celebrant's birthday and fills again every year. Users are friends and family, almost all on a phone, arriving from a chat app. No accounts.
**Mode per surface:**
- Wish form: Operate. Finish in under a minute, one screen, no distractions.
- Birthday reveal: Experience. The one authored moment.
- Reading wishes: Read. Comfortable long text, grouped by year.
- Create a jar and settings: Operate. Plain and reassuring.
**Reference sites:** none. Built from design knowledge; group-card products were deliberately not used as references because they're busy workplace tools.
**Key characteristics:**
- Cream paper, dark ink, one raspberry accent. Colour is rare and means "act here" or "this is a wish".
- The jar is the only illustration on any screen.
- Wishes look like slips of paper, slightly tilted, never like app cards.
- Big friendly headings, generous space, nothing competing with the message.
- Gentle motion, with one real moment: the lid.

## Colors

**Strategy:** Restrained. One accent (raspberry) on warm neutrals. Raspberry appears only on the primary button, links, prompt chips and the countdown. The note papers (white, blush, butter) are tinted neutrals, not extra accents, and they only ever hold wishes.
**Light or dark:** light by default, because people mostly open the link during the day in a chat app. Dark follows the phone's setting (`prefers-color-scheme`) and suits the midnight reveal. Dark mode isn't an inversion. Surfaces step up in warm browns (background, then surface, then note papers), the jar line becomes cream, raspberry buttons keep their colour, and raspberry text lightens to `dark-primary-text` to stay readable.

**Contrast (WCAG AA, all measured):**

| Pair | Light | Dark |
|---|---|---|
| Body text on background | 14.2:1 | 14.8:1 |
| Muted text on background | 5.9:1 | 8.0:1 |
| White on primary button | 5.2:1 | 5.2:1 |
| Link / accent text on background | 6.5:1 | 7.2:1 |
| Text on blush note | 12.3:1 | 11.7:1 |
| Muted (wisher name) on blush note | 5.1:1 | 7.1:1 |
| Input border on surface (non-text, needs 3:1) | 3.4:1 | 3.1:1 |
| Focus ring on background | 14.2:1 | 14.8:1 |
| Success / warning / error text on background | 5.9 / 5.9 / 6.1:1 | 8.4 / 9.2 / 7.7:1 |

Rules: never set text in bright `primary` on blush, and never use `hairline` for anything that must be seen (it's for dividers only). Status is never colour alone: errors always carry words, and an icon when one helps.

## Typography

**Faces:** Bricolage Grotesque for display and headings, Figtree for everything else. Both are free on Google Fonts under the SIL Open Font License, checked on 2026-11-10.
**Why:** Bricolage has squashed, friendly curves that read as playful without being childish. Figtree is clean, open and easy to read on a small screen, so long heartfelt wishes stay comfortable.
**Loading:** load both through `next/font/google` (self-hosted at build time, no runtime request to Google): Bricolage Grotesque with the `opsz` axis at weights 500 and 700, Figtree at 400, 500 and 700. Use `display: swap`. Fallbacks: `ui-rounded, system-ui, sans-serif` for display, `system-ui, sans-serif` for body.
**Scale (phone):** display 30 to 40px (fluid), heading 22px, body 17px, label 14px, small 13px. Headings are always at least 1.3 times body size. Body never goes below 16px (it stops iOS zooming into inputs).
**Rules:**
- Wishes are always set in body type, never in display or a script face. They are the content, not decoration.
- Bricolage is for headings and the celebrant's name only. Never use it for paragraphs or buttons.
- Use sentence case everywhere. No all-caps labels and no letter-spaced small caps.
- Numbers in the countdown use tabular figures so the digits don't jump.

## Layout

**Phone first.** One column, 20px side gutters, content max width 30rem (480px). On tablets and desktops the same column centres on the page, with more space above the jar. No sidebars, no multi-column layouts.
**Rhythm:** a 4px base. Inside a group use 8 to 12px; between groups use 24 to 32px; above and below the jar use 32 to 48px. Generous white space is part of the brief: if a screen feels empty, the copy needs work, not decoration.
**Order on every public page:** the jar, then whose jar it is, then when it opens, then the action. The main button sits in the natural thumb zone near the bottom of the content, full width.
**Touch targets:** at least 44 by 44px; buttons are 52px tall; chips are 40px tall with 8px gaps.
**Safe areas:** respect `env(safe-area-inset-*)` so nothing sits under the iPhone home bar.

## Elevation & Depth

Almost flat. Depth comes from paper colour, not shadows. Surfaces step from background to surface to note paper. Wish notes may use one soft offset shadow (`0 1px 2px rgb(43 36 32 / 0.08)`) so they read as slips of paper. No glows, no blurred glass, no shadows on buttons.

## Shapes

- Buttons, inputs and textareas: 12px.
- Prompt chips: fully rounded.
- Wish notes: 3px top-left and 12px elsewhere, like a slip torn from a pad, tilted between -1° and +1° (alternating). In dark mode or for reduced-motion users the tilt stays; it isn't motion.
- Sheets and dialogs: 20px top corners.
- Nothing else is rounded. Never put a card inside a card.

## Components

**The jar** is the signature and the only illustration. It is drawn in a single ink line (`text` colour, 2.6px stroke at full size, round caps), slightly hand-wobbled, with glass implied by one faint highlight stroke. Notes inside are small tilted rectangles filled with the three note papers. The lid sits closed until the birthday, then lifts and tilts. Sizes: 130 to 150px wide on public pages, 110 to 120px in the private view, 64px in the share preview image. The final artwork should be drawn by hand as SVG paths (not built from CSS shapes); the shapes in the preview are direction sketches. The jar is decorative (`aria-hidden`), so the heading always says the same thing in words.

**Primary button:** raspberry with white text, full width on phones, 52px tall. Pressed state uses `primary-pressed`. Focus shows a 2px `focus` ring offset by 2px. Disabled state uses 40% opacity, plus a reason in text nearby; never disable without saying why. Loading keeps the label and adds "…" ("Dropping it in…") so the width doesn't change.

**Secondary button:** 1.5px ink outline, transparent fill, same size. Use one primary per screen at most.

**Text link:** `primary-text`, underlined with a 3px offset. Use links for skip actions ("No thanks") and navigation, never for the main action.

**Inputs and textarea:** `surface` fill, 1.5px `border`, 12px radius, 17px text. Labels always sit above the field (no placeholder-only labels). The textarea has a live character counter ("56 / 1000") in small type, which turns `error` with words at the limit. Errors appear under the field in `error` colour with a plain sentence. Focus uses the 2px ring.

**Prompt chips:** `primary-soft` fill, 1.5px raspberry border, ink text, fully rounded, 40px tall, wrapping onto lines. Tapping one inserts its sentence starter at the cursor and moves focus to the textarea. Pressed state darkens the fill slightly. Chips are buttons (`<button type="button">`), not links.

**Wish note:** a note-paper background (rotating white, blush, butter), the wish in body type, the wisher's name below in small muted type, or "Anonymous". The 3px/12px radius and slight tilt make it read as a slip. Long wishes simply grow; never truncate a wish.

**Countdown:** shows days ("Opens in 12 days") until the last 24 hours, then hours and minutes ("Opens in 5 h 12 min"), ticking once a minute with tabular figures. During birthday week it's replaced by "It's Ama's birthday week".

**Visibility control (settings):** a two-option segmented control, "Private" and "Wall", 48px tall, with the selected side filled `primary-soft` and labelled in words.

**Delete jar dialog:** a bottom sheet with a plain warning, a field where the celebrant types the jar's name, and a destructive button in `error` colour that only enables once the name matches.

**Link box:** shows a link in body type on a `surface` field with a 1.5px `border` outline and 12px radius, wrapping long links (never truncated). A secondary "Copy" button sits beside it (below it at 320px); after copying it reads "Copied" for 2 seconds and the change is announced politely. The secret link's box carries a small "Secret" label in `primary-text`. If copying fails: "Couldn't copy. Press and hold the link to copy it."

**Undo bar:** ink background with cream text, 12px radius, 48px tall, pinned to the bottom above the safe area with 16px side margins. Text plus one action, e.g. "Wish deleted. Undo". It stays 6 seconds (paused while focused), is announced politely, and never stacks: a new one replaces the old.

**Sticky button:** a primary button pinned to the bottom of Wall pages ("Add your wish"), full column width, above the safe area, on a `background` strip so content doesn't show through. It hides while the form it points to is on screen.

**States every screen must design for:** loading (the jar outline with no notes, no spinners), empty ("No wishes yet. Share your link and they'll start dropping in."), error (a plain sentence and a retry, with the draft kept), rate limited ("Too many wishes from here right now, try again in a bit."), not found (the jar with its lid off and empty: "This jar isn't here any more."), and very long names (wrap, never truncate the celebrant's name).

## Motion

- **Approach:** intentional. Small and gentle, except for one authored moment.
- **Easing:** enter `cubic-bezier(.22, 1, .36, 1)` (ease-out); exit ease-out; move ease-in-out. No springs that overshoot, no bounce.
- **Durations:** micro 80 to 100ms (chip and button press); short 150 to 200ms (fades, countdown digits); medium 300 to 480ms (note drop); long 600 to 900ms (lid opening).
- **Note drop (on sending a wish):** the note falls into the jar over 480ms, rotating from -12° to its resting tilt, ease-out, then settles. This plays once.
- **The one authored moment, the lid opening:** at the first visit after unlock, the lid lifts and tilts away (700ms), then the notes rise out of the jar one by one with a 90ms stagger and become the list of wishes. The heading fades in after them: "Happy birthday, Ama." It plays once per occurrence; later visits show the open jar at rest. Never loop it.
- **Nothing else moves:** no parallax, no scroll-triggered effects, no animated backgrounds, no confetti.
- **Reduced motion (`prefers-reduced-motion: reduce`):** every movement is replaced by a 150ms opacity fade. The note appears in the jar without falling. The lid is simply shown open. Notes appear together. The countdown updates without transitions. Implement as a global rule plus a check in the two authored animations, and test with the setting on.

## Tone of voice

Short, warm and plain, like a friend writing a note. Use the celebrant's name wherever it fits. One idea per sentence. At most one exclamation mark per screen. No emoji in interface text (wishers can use any emoji they like). Never say "sealed", "submit", "user", "content" or "successfully". Never guilt or pressure people.

| Moment | Say | Don't say |
|---|---|---|
| Public page, before the birthday | "Opens at midnight on 22 November. Get yours in before then." | "Submit your message before the deadline." |
| Visibility line | "Only Ama will see this." / "This will appear on Ama's wall." | "Privacy: restricted." |
| Send button | "Drop it in the jar" | "Submit" |
| After sending, before the birthday | "Your wish is in. Saved for Ama's next birthday." | "Your message has been successfully submitted!" |
| After sending, birthday week | "Your wish is in Ama's jar." | "Wish delivered 🎉" |
| Invite to start a jar | "Want a jar like this for your own birthday?" | "Sign up now to unlock your free jar!" |
| Birthday week, public page | "It's Ama's birthday week. Add yours." | "You're late!" |
| Celebrant, before the birthday | "9 wishes waiting. Your jar opens at midnight on 22 November." | "9 sealed messages." |
| Celebrant, reveal | "Happy birthday, Ama. 12 wishes were waiting for you." | "Your jar has been unlocked." |
| Save the private link | "This is your secret link. Save it somewhere safe; it's the only way back to your jar." | "Warning: link loss is permanent." |
| Rate limited | "Too many wishes from here right now, try again in a bit. Your wish is saved." | "Error 429." |
| Empty jar | "No wishes yet. Share your link and they'll start dropping in." | "No data." |
| Not found | "This jar isn't here any more." | "404: resource not found." |
| Delete jar | "This deletes the jar and every wish in it, for good. Type Ama to confirm." | "Are you sure?" |

## Do's and Don'ts

- Do keep raspberry for actions, links, chips and the countdown only.
- Do set every wish in Figtree body type on a note paper, with the full text shown.
- Do give every screen its empty, loading, error and long-content states.
- Do test each screen at 360px wide, with large text, in dark mode and with reduced motion on.
- Do check new colour pairs against AA before shipping and add them to the contrast table.
- Don't add illustrations, icons-in-circles, confetti or decorative shapes. The jar is the only drawing.
- Don't use cards for layout. Notes are paper slips; everything else sits on the page.
- Don't use a second accent colour or gradients, including on buttons.
- Don't use script or handwriting fonts for wishes or interface text.
- Don't centre long paragraphs. Centre only short headings and confirmation lines.

## Decisions Log

| Date | Decision | Rationale |
|------|----------|-----------|
| 2026-11-10 | Initial design system created | Created by /design-consultation from docs/BRIEF.md "Look and feel" and the approved design doc; built from design knowledge (research declined) |
| 2026-11-10 | Accent: raspberry #C23B5E | Founder choice. Only option passing AA as both button and text on cream; reads as birthday without being loud |
| 2026-11-10 | Jar: single-line ink sketch | Founder choice over the cloth-top jam jar. Light, scales to any size, and keeps the notes as the only colour |
| 2026-11-10 | Type: Bricolage Grotesque + Figtree | Founder choice. Playful and modern; avoids the common cream-and-serif look |
| 2026-11-10 | Memorable thing: the lid opening | Founder answer; every screen stays quiet so the reveal stands out |
| 2026-11-10 | Link box, undo bar and sticky button added | /plan-design-review DR-D8; screens spec in docs/designs/wishjar-v1-screens.md |
| 2026-11-10 | Dark mode follows the phone setting | The reveal happens at midnight; light stays the default for daytime chat-app visits |
