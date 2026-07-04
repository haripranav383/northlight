Northlight

V1 is a small organism, not a feature list: two senses, one brain, one closed reflex arc, built to grow.

Northlight is an eldercare triage system that catches quiet decline about a week before it becomes a 3 a.m. ambulance.

The problem: conditions like UTIs, dehydration, and cognitive drift show faint warning signs days early, but those signs are split across two streams that nothing connects, while nurses drown in single stream alarms tuned to population averages. Wearables see the body. Companion apps see the mind. Nothing fuses the two against a person's own baseline. Northlight sits in that empty intersection.

How it works:
June, a warm voice companion, chats with a resident and quietly logs cognitive flags like confusion, repeated questions, or low mood. A wearable streams vitals (simulated in V1). The fusion engine reads both against that resident's own baseline. A COPD resident with 93% oxygen is healthy for him, so no alarm. Only when a pattern crosses a threshold does the nurse get one evidence backed card: a soft hypothesis, the exact vitals, and the elder's own words. Never a diagnosis. The nurse's follow up task travels back through June and gets confirmed by the resident, closing the loop.

The engine is a transparent, hand written ruleset in V1, architected so a learned model can slot into the same inputs and outputs later without dismantling anything.

What's in this repo:
CLAUDE.md — the frozen V1 engineering specification: data model, the nine rule fusion engine, severity system, build order, and the 90 second demo script. This document is the single source of truth; every PRD below answers to it.
PRD-triage.md — the nurse triage dashboard, the surface where the loop meets the caregiver.
PRD-companion.md — June, the voice companion the elder actually lives with.
PRD-fusion-engine.md — the brain: baseline deviation detection, the nine rules, the no-spam law.
PRD-backend.md — the nervous system: the unified real-time state that binds it all.
What this is not
Northlight is decision support, not a medical device. It never diagnoses; it detects deviation from a personal baseline and hands the evidence to a human. It is not a fall alarm or nurse call replacement, and it does not attempt to catch acute events in real time. It watches for the slow drifts that current tools miss.

Status:
Spec complete. Build starts July 7, 2026. The live URL will appear here.

Who:
Built by Hari Pranav (incoming MEng, Systems Design Engineering — Health Technologies, University of Waterloo), co-owned with a co-founder of 35+ years of industry experience whose voice companion concept seeded the ecosystem.







