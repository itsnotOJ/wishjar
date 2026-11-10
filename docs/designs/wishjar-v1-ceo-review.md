# CEO Review: Wishjar v1

Reviewed by /plan-ceo-review on 2026-11-10 · Branch: claude/clever-gates-o17qb3
Plan under review: `docs/designs/wishjar-v1.md` (against `docs/BRIEF.md`)
Mode: HOLD SCOPE (explicit founder instruction: "Hold scope to version one as written in the brief")
Depth: strategy and scope (no code exists yet)

## Answers to the founder's three questions

**Single riskiest assumption:** wishers will write *before* the birthday. Every reminder people get today fires on the day, while the reveal needs wishes early. If most friends wish on the day, the jar opens near-empty at midnight and the main hook goes flat. Mitigation approved: D17 (deadline in the copy, plus the event log separating before-unlock wishes from birthday-week ones). Test: on each test birthday, compare wishes before midnight with wishes after.

**Missing pieces that would stop it working (all resolved by founder decisions):**
- Lost gift claim link leaves an unopenable jar → D10, gift-management link that can only reissue a claim link.
- Birthday moved to a date already passed → D11, unlock now if inside the 7-day window, else next occurrence.
- "Did the celebrant share?" can't be measured → D12, Share-button tap plus a source tag, with every success criterion mapped to an event.
- Test jars created before email backup exists → D13, Phase 2 live before the first test jar.
- Inbox flooding, false blocking on shared Wi-Fi, an unprotected job route → D18, three safety fixes.

**Cuts:**
- Phase 4 (nudge and calendar) deferred to TODOS.md (D14). It wasn't in the brief, isn't needed for about 11 months, and caused gap R2-5.
- Public wish count dropped (D16). It was unrequested scope.
- Wall kept (D15), at the founder's choice.

## Decision ledger

| ID and owner | Contract and evidence | Current | Proposed | Status | Exact approval and scope |
|---|---|---|---|---|---|
| GAP-1 (Section 4) | Spec review R2-2; design doc D2 | No recovery for an unclaimed gift jar | Gift-management link that only reissues claim links | approved | D10 = A |
| GAP-2 (Section 4) | Spec review R2-4; fix 2 | Re-resolution undefined for passed dates | Unlock now if in window, else next occurrence; no email | approved | D11 = A |
| GAP-3 (Section 8) | Spec review R2-6; premise 1 | Shares unmeasurable | Share tap + `?s=` source tag + timing; criterion map | approved | D12 = A |
| GAP-4 (Section 9) | Spec review R2-20; premise 7 | Phase 2 required by birthday only | Phase 2 required before first test jar | approved | D13 = A |
| CUT-1 (Step 0G) | D3 not in brief; R2-5 | Phase 4 in v1 | Defer to TODOS.md | deferred | D14 = A |
| CUT-2 (Step 0G) | Brief: Wall mode | Wall in v1 | Defer Wall | declined (kept) | D15 = B |
| CUT-3 (Step 0G) | Assumption 1, not in brief | Public count at 3+ wishes | Drop | approved | D16 = A |
| RISK-1 (Section 11) | Premise 2; status quo is on-the-day reminders | No early-wish push | Deadline copy + before/after split in log | approved | D17 = A |
| SEC-1 (Section 3) | Spec review R2-13, R2-15, R2-16, R2-17 | No email limits; global IP limit; open job route | Three safety fixes | approved | D18 = A |

Approval readiness: PASS (GAP-1 D10, GAP-2 D11, GAP-3 D12, GAP-4 D13, CUT-1 D14, CUT-2 D15, CUT-3 D16, RISK-1 D17, SEC-1 D18).

## Section findings

**1. Architecture.** One Next.js app on Vercel, one Supabase Postgres, Resend for email, and `pg_cron` for the hourly job.
```
 phone browser ──> Next.js (Vercel) ──> Supabase Postgres
   public page        server actions       jar, jar_occurrence, wish,
   private page       /api/jobs/hourly <── pg_cron + pg_net (shared secret, D18)
   claim/recovery          │                wish_attachment (empty), event_log,
                           └──> Resend      email_send_log, rate_limit (hashed, 1h)
```
Single points of failure: Supabase (all data) and Resend (recovery). Both are acceptable at this scale. Rollback is a Vercel instant rollback to the previous deploy, and migrations must be additive. Issues: GAP-1 and GAP-2 (resolved).

**2. Error and rescue map (capability level).**

| Capability | Failure | User sees | Owner must prove |
|---|---|---|---|
| Submit wish | DB write fails / rate limited | Draft kept, friendly retry message | Draft survives the error; no duplicate on retry |
| Recovery email | Resend down | Same "if a jar exists, we sent it" message; failure logged | Send failures are visible in logs |
| Hourly job | Run missed / overlaps | Email still sent once same day | Claim-row idempotency (fix 4) |
| Claim link | Already used / reissued | "This jar has been claimed" or "This link was replaced" | No path mints two private links |
| Unlock | Clock/timezone edge | Correct state at 00:00 local | Tests at DST edges, +5:30, 29 Feb |

No critical gaps remain after D10 to D18.

**3. Security.** Bearer-token links (public, private, claim, gift-management, recovery) are long, random and stored hashed. The private token is never derivable from the public one. All user text is escaped. Fixed by D18: recovery and confirmation flooding (High likelihood, Medium impact), shared-IP false blocking (Medium, Medium) and the open job route (Medium, High). Still to watch: tokens leaking through `Referer` headers. The implementation must set `Referrer-Policy: no-referrer` on private and claim pages.

**4. Data flow and edge cases.** Covered: double-submit (disable the button plus server dedupe on draft hash), navigating away (local draft), a deleted jar (friendly not-found page), a birthday edit across an occurrence boundary (fix 2, D11), and a claim link opened twice (D10). Edge cases the build must test: two tabs claiming the same link at once (only one may succeed), and a wish submitted exactly at 00:00 local (goes to the occurrence that is open at server time).

**5. Code quality.** No code yet. Watch out for spreading unlock logic around. One pure function, "which occurrence is open or sealed at instant T for jar J", must own every unlock and late-window decision.

**6. Tests.** Required: unit tests for the occurrence and unlock function (DST, +5:30, +5:45, 29 Feb, birthday edits, 7-day window); integration tests for claim, recovery and rate limits; an E2E test on a phone viewport for create → share → wish → confirmation; a job test for exactly-once email. Flakiness risk: anything time-based must use an injected clock.

**7. Performance.** Tiny scale. Indexes needed: `wish(occurrence_id)`, `jar_occurrence(jar_id, unlock_at)`, `email_send_log(jar_id, occurrence_id, kind)` unique, and a token-hash lookup on each link table. No issues.

**8. Observability.** The event log (D8, D12) covers the success criteria. Add structured logs for email sends and failures, and for job runs (jars due, sent, skipped). Alert: hourly job didn't run for 3 hours (a Supabase cron failure is otherwise silent).

**9. Deployment.** Migrations first, then deploy. Phases ship behind normal deploys with no feature flags. Phase 2 is gated before the first test jar (D13). Post-deploy smoke test: create a jar, send a wish, open the private link, then request recovery against the Resend test sender.

**10. Long-term trajectory.** Reversibility 4/5. The `jar_occurrence` model and the empty `wish_attachment` table keep photos, voice notes and the deferred Phase 4 additive. Debt: one TODO (Phase 4) with a hard trigger date.

**11. Design and UX.** First thing seen: whose jar and when it opens. Then the mode line, the prompts and Send. The emotional arc runs: curiosity (jar with notes) → effort (prompts) → warmth (confirmation) → invitation. States to design: jar not found or deleted, rate limited, network error with draft kept, birthday week, already claimed. Riskiest UX point: RISK-1, the early-wish copy. Recommend running `/plan-design-review` before building the UI.

## NOT in scope
- Year-two nudge and calendar file: deferred (D14), see TODOS.md.
- Public wish count: rejected (D16), unrequested scope.

## What already exists
Nothing in code. The repo holds only `docs/BRIEF.md` and the design doc.

## Dream state delta
v1 answers "do people share, write and open?" on 3 to 5 birthdays. It doesn't yet prove year-two return (Phase 4 plus a full year).

## Completion Summary
```
  Mode selected        | HOLD SCOPE
  System Audit         | Greenfield: brief + design doc only, no code
  Step 0               | Hold scope; 3 cuts proposed, 2 accepted (Phase 4 deferred, count dropped), Wall kept
  Sections 1-11        | 5 gaps found, all resolved by D10-D13, D18; RISK-1 resolved by D17
  Error/rescue registry| 5 capability rows, 0 CRITICAL GAPS
  Failure modes        | 5 total, 0 CRITICAL GAPS
  TODOS.md updates     | 1 item (Phase 4)
  Outside voice        | skipped: Codex not installed; native fallback not run (two independent spec-review rounds already ran on this doc)
  Lake Score           | 6/6 coverage-scored recommendations chose the complete option
  Unresolved decisions | 0
```

## GSTACK REVIEW REPORT
| Review | Trigger | Why | Runs | Status | Findings |
|--------|---------|-----|------|--------|----------|
| CEO Review | `/plan-ceo-review` | Scope & strategy | 1 | CLEAR | mode: HOLD_SCOPE, 0 critical gaps |
| Outside Review | Codex (not installed) | Independent 2nd opinion | 0 | skipped | no completed external review |
| Eng Review | `/autoplan` | Architecture & tests (required) | 0 | not run | — |
| Design Review | `/plan-design-review` | UI/UX gaps | 0 | not run | — |
| DX Review | `/plan-devex-review` | Developer experience gaps | 0 | not run | — |

- **OUTSIDE COVERAGE:** Codex plan review skipped (CLI not installed). Native fallback not run. Independent coverage came from two office-hours spec-review rounds on the same doc.
- **VERDICT:** CEO CLEARED. Eng review required.

NO UNRESOLVED DECISIONS
