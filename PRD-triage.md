# PRD — Northlight Nurse Triage System (V1)

**Status:** Draft v1 — living document. Revise as we learn.
**Owner:** You (product). Built with Claude Code.
**Parent docs:** MASTER-DOCUMENT.md (why), CLAUDE.md (frozen engineering authority — where they conflict, CLAUDE.md wins on data shapes and severity levels).
**Scope of this PRD:** the nurse-facing surface only — `/dashboard`. The fusion brain, June, and the control room are dependencies, not deliverables of this document.

---

## 1. The problem

A floor nurse in an eldercare facility is responsible for many residents and receives a constant wash of undifferentiated alarms and raw data. The result is alarm fatigue: the monitoring meant to help becomes something to tune out, slow declines slip through, and attention burns on whatever is loudest rather than whatever matters most.

The nurse doesn't need more data. She needs three things, per moment: **who** needs her, **why** (with proof she can trust), and a way to **act** that doesn't create paperwork.

## 2. Who it's for

**Primary user:** one floor nurse, mid-shift, tired, interrupted every few minutes, glancing at a screen between rooms. She will give any new tool about ten seconds of patience before deciding whether it's signal or noise. She does not read manuals.

**Secondary beneficiary:** the facility (evidence trail, earlier catches) and the resident (the nurse arrives at the right moment, already informed). They don't touch this screen.

## 3. User stories

1. As a nurse, I want one queue of open alerts sorted by urgency across the whole floor, so I never have to hunt for who needs me most.
2. As a nurse, I want every alert to show me *why it fired* — the evidence, including the resident's own words — so I can trust it instead of tuning it out.
3. As a nurse, I want a true emergency (S4) to be impossible to miss, and everything else to be impossible to be startled by.
4. As a nurse, I want to acknowledge, resolve, or dismiss an alert in one or two clicks, so triage doesn't become admin.
5. As a nurse, I want to send a gentle task to a resident through June (pre-filled from the alert) and see it confirmed live, so I know the loop actually closed without walking back to check.
6. As a nurse, I want equipment noise (battery, out-of-range) kept completely out of my clinical queue, so a dying battery never competes with a dying person.
7. As a nurse, I want a glance-level view of every resident's current status, so "quiet" is a fact I verified, not an assumption.

## 4. What success looks like (the outcome)

**The one-sentence North Star:** a nurse who has never seen the system can look at the dashboard for ten seconds and correctly answer "who do I check on first, and why?"

Measurable proxies for V1 (demo-scale):

- **Walkthrough test:** steps 1, 4, 5, and 6 of the 90-second walkthrough (CLAUDE.md §23) run start-to-finish on the live URL with no manual database touches.
- **No-spam guarantee:** the UTI scenario (Mary, day 1 → day 2) produces exactly ONE alert card that accumulates evidence — never duplicates.
- **Trust-by-silence:** the "quiet day" scenario produces an empty queue. An empty queue is a feature, not a bug.
- **Loop closure:** task created on dashboard → delivered by June → confirmed by resident → status flips on the dashboard live, in under 5 seconds of the confirmation, with no refresh.
- **Ten-second legibility:** severity, name, room, title, and evidence count readable per card without opening it.

## 5. Core features (V1 — the minimum that solves the problem)

These map to CLAUDE.md §13 and are ranked. The last two are the ones I'd cut first under pressure (see §7).

**F1 — The live queue.** Open clinical triage events, severity-desc then newest-first, real-time via Firestore listeners. S4 renders as a full-width red banner with a tone until acknowledged; S3 pins to top with a single chime; S2 appears silently. Severity colors are the only color on the page (frozen palette, CLAUDE.md §12).

**F2 — The evidence panel.** Clicking a card slides out: the soft hypothesis (always ending in "— not a diagnosis. Nurse judgment required."), every EvidenceItem with the resident's exact quoted words where relevant, and the baseline deviation. This panel is the answer to alarm fatigue; it is never cut.

**F3 — Act on an alert.** Acknowledge / Resolve / Dismiss buttons in the panel. Nurse name is typed once per session (no logins in V1). Resolving/dismissing silences that rule for that resident for 6 hours (enforced by the brain; surfaced here).

**F4 — Create task (the closed loop).** From the panel, "Create task" pre-filled with a suggested instruction from the alert. Writes a CareTask; status chips update live: pending → delivered → confirmed/declined. This is the loop closing on screen — never cut.

**F5 — Ops tab.** Battery/out-of-range (S1, channel:'ops') live in a separate tab with a count badge. They never appear in the clinical queue.

**F6 — Elder strip.** All four residents, each with a dot colored by their worst open alert (or neutral if none), linking to `/resident/[id]`. This is how "quiet" becomes verifiable.

**F7 — Tasks panel.** A small standing list of all tasks and their live statuses, so a task's journey is visible even after its alert is resolved.

## 6. Explicitly NOT in V1 (the anti-wishlist)

- **Logins, roles, multi-nurse assignment.** One typed name per session. V2.
- **Filters, search, sorting options.** Four residents don't need search. Revisit at real-floor scale.
- **Notes / free-text charting on alerts.** The resolution field on dismiss is the ceiling. This tool must never become paperwork.
- **Notifications outside the screen** (SMS, push, paging). V1 assumes the dashboard is open. Geofenced paging is Phase 2 (CLAUDE.md §21).
- **Daily report generation & sign-off.** Real feature, but it belongs to the reporting slice (week 8), not the triage core. It gets its own mini-spec.
- **Configurable thresholds/sounds/dark mode.** The rules are the brain's; the meanings of severity levels are frozen.
- **Any diagnosis language anywhere.** Structural rule, not a feature decision.

## 7. Priorities under pressure

Cut order if week 6 runs long: F7 → F6 → S3 chime polish → ops tab becomes a plain list.
Never cut (they ARE the product): F1's no-spam behavior, F2 evidence panel, F4 closed loop, personal-baseline framing in evidence text.

## 8. Dependencies & assumptions

- The brain (`/api/evaluate`, weeks 4–5) produces TriageEvents with dedup already applied. The dashboard renders truth; it never creates or merges alerts itself.
- June (weeks 2–3) delivers tasks and writes confirmations. The dashboard only observes CareTask status.
- Firestore composite index for the queue (filter status + sort severity/createdAt) is committed as `firestore.indexes.json` in week 1 — before this screen exists.
- Data shapes are frozen per CLAUDE.md §6. Any change requires explicit sign-off.
- Design language: cool, dense, clinical (CLAUDE.md §17) — near-white, slate text, monospace numerals, sharp corners, one slide-in animation per new card and nothing else.

## 9. Open questions (living — resolve before or during week 6)

1. **Acknowledged-but-stale escalation.** MASTER-DOCUMENT §7 flags "someone must watch the watchers." Does an unresolved S3/S4 visually age (e.g., border pulses after 15 min) in V1, or is that Phase 2? My lean: a cheap visual age indicator is in-scope; auto-escalation is not.
2. **Dismiss vs. resolve semantics.** Does "dismiss" require a one-line reason (for the evidence trail) or is friction the greater evil? My lean: optional reason, never required.
3. **What does the S4 tone actually sound like, and how does it stop?** Needs a decision before build, not during.
4. **Empty-state copy.** "No open alerts" must read as reassurance, not malfunction. Words TBD by you — this is a taste call.

## 10. Decisions deliberately made in this draft — push back here

- **D1:** Daily report + sign-off moved OUT of this PRD (into a later reporting spec). Argument: it's a different job (handover, not triage) and week 8, not week 6.
- **D2:** Tasks panel (F7) kept IN core, but first on the cut list. Argument for keeping: the closed loop is the demo's emotional peak and should stay visible after the alert closes. Argument for cutting: the status chip inside the alert panel might be enough.
- **D3:** Elder strip (F6) kept IN. Argument: it's what makes silence verifiable (rule 7 of "rules we never break"). Cheap to build, high trust value.
- **D4:** No sound for S2 at all, ever, even as an option. Argument: alarm fatigue is the enemy; amber must be safe to ignore for an hour.

---

*Revision log:*
*v1 — initial draft, four open decisions pending discussion.*
