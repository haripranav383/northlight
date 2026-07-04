# PRD — The Backend (Firebase / Firestore, V1)

**Status:** Final for V1. Reconciled against CLAUDE.md, the frozen build authority. Where this document and CLAUDE.md differ, CLAUDE.md wins.
**Scope:** the nervous system. The real-time state that binds June, the wearable, the engine, and the dashboard into one organism.

---

## 1. Objective

The backend is an event-driven conduit. It never scans or polls; data arrives, the brain evaluates it in the same moment, and the result is pushed live to whoever is watching.

- **For the business:** costs near zero when the floor is quiet.
- **For the caregiver:** alerts arrive the instant they exist, no refresh.
- **For the elder:** monitoring that is ambient and invisible.

## 2. The unified real-time state (the moat)

Existing tools are siloed: the wearable talks to one server, the companion to another, and the nurse connects the dots by hand across portals. Northlight forces every stream — biometric, cognitive, and intervention — into a single real-time database keyed by the same residentId. That is what makes cross-stream fusion possible in the same moment data is created, and it is the architectural angle no siloed product can replicate without rebuilding itself.

## 3. The data architecture (eight collections, frozen in CLAUDE.md)

Flat NoSQL. Every document anchored by residentId.

- **residents** — the anchor: identity, conditions (used by the rules only, never told to June), the personal baseline (resting HR, SpO2, sleep hours, interruptions, repetitionNormal), and June's memory of the person.
- **vitalsSamples** — the physical stream: one wearable reading (HR, SpO2, state, battery, in-range) every 30 seconds while streaming.
- **nightSummaries** — one per elder per night: hours, interruptions, quality. The infection rule cannot fire without these.
- **cognitiveFlags** — the cognitive stream: June's structured observations with type, subtype, the elder's exact quote, a severity hint, and confidence.
- **triageEvents** — the output: alerts created only by the brain. Rule id, channel (clinical or ops), frozen severity, hypothesis ending in the frozen disclaimer, evidence list, status.
- **tasks** — the closed loop: nurse instruction, delivered via the companion, status pending → delivered → confirmed or declined.
- **sessions** — one conversation's messages plus its engagement score, computed once when the chat ends.
- **dailyReports** — the end-of-day summary per elder, with the nurse's sign-off.

## 4. The wiring (the golden rule)

- No server functions, no schedulers in V1. Event-driven behavior happens at the application level: **every code path that writes a vital, a flag, or advances a day calls the single shared evaluate helper (lib/evaluate.ts → /api/evaluate) immediately after writing.** New data is always judged the moment it exists. Same semantics as database triggers, zero infrastructure, free tier.
- All live surfaces (queue, elder strip, task chips, companion task delivery) use Firestore real-time listeners. Polling loops are forbidden: they create lag and burn quota.
- Week-1 chore: the dashboard's main query filters and sorts simultaneously and needs a Firestore composite index. Commit firestore.indexes.json up front so the error never appears mid-demo.

## 5. Data sensitivity, by design

- The backend stores calibration data and evidence, not medical records: baseline numbers, observed readings, quoted words, structured counts.
- Alert outputs carry symptoms and evidence with the frozen not-a-diagnosis line, never diagnostic labels. This is what keeps the product decision support.
- The residents.conditions field exists for rule context only and is never surfaced to June or written into alerts.
- The stricter privacy-minimization posture, and consent architecture generally, is Phase 2 governance work.
- The longitudinal record this schema accumulates — every flag, vital, night, and outcome per resident — is deliberately the exact training substrate the Phase 2 learned model requires, with no external medical files needed.
