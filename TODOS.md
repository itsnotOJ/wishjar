# TODOS

## Year-two re-share nudge and calendar file (deferred from v1)
- **What:** An email about 14 days before each birthday (first hourly run at or after 08:00 local, once per jar per occurrence) asking the celebrant to re-share their jar. Plus an "Add to my calendar" file from the private view: a yearly all-day event using a "last day of February" rule for 29 Feb birthdays, public link only.
- **Why:** Year two only works if the celebrant re-shares before the birthday. Without it, year two's jar may open empty.
- **Pros:** Makes "fills up again every year" real without relying on memory.
- **Cons:** Adds a second email type, a calendar file and nudge scheduling logic.
- **Context:** Decided in the office-hours session (D3), deferred by /plan-ceo-review D14 so v1 ships with Phases 1 to 3. The full rules are in `docs/designs/wishjar-v1.md` (D3, fixes 3, 4 and 7, Assumptions 5 and 6). No first-year nudges exist in v1, which also resolved spec-review gap R2-5. Known limitation: Google Calendar ignores alarms in imported .ics files, so consider a second yearly event 14 days before instead of an alarm.
- **Effort:** human M / CC S
- **Priority:** P1 once the first test jars exist
- **Deadline / trigger:** must ship 14 days before the earliest jar's second birthday.
- **Depends on:** Phase 3's hourly job and email send log.
