# Northlight — Build Spec (CLAUDE.md)

Read this whole file at the start of every coding session before writing any code.

Plain-English rule for reading this doc: the normal sentences are for the human who's
directing the build (they design, they don't code). The code blocks and tables are the
exact details for Claude Code. If you're the human, you can skim past the code blocks —
the sentences around them tell you everything you need to follow along.

---

## 1. What we're building, in one breath

An old person wears a wristband. It quietly watches their body — heart rate, sleep,
oxygen. At the same time, a warm voice companion ("June") chats with them and quietly
notices things in *how they talk* — confusion, low mood, "I didn't sleep," a pain
mentioned in passing.

Neither signal means much alone. The magic is putting them together. A small brain
cross-checks the body signals against the talk signals — always measured against *this
person's own normal*, not some average — and decides: is something actually starting to
go wrong here, and how urgent is it?

When the answer is yes, a nurse sees one clean card. Not an alarm. A card that says
*what might be wrong, how sure we are, and the evidence* — including the person's own
words. The nurse goes and checks on them in real life and does whatever's needed.

That's the loop. Body + voice → one brain → one clear nudge to the right nurse → the
nurse helps the person → it all gets remembered. Then it does it again tomorrow.

---

## 2. This is the foundation, not the finished thing

We are building the **solid foundation**. Its job is to prove the loop works end to
end, and to be built clean enough that the big grown-up version bolts on later without
tearing anything down.

**What we build now:**
- Real conversations (a real language model talks to the elder).
- A real brain that decides (a clear set of rules — see section 11).
- A real closed loop (nurse's task actually travels back to the elder and gets
  confirmed).
- It runs live on the internet.

**What we do NOT build now** (this is the grown-up version, section 21):
- Real wearable hardware (ours is faked, on purpose).
- A self-learning AI brain (ours is hand-written rules — same inputs, same outputs,
  but you can *see* why every decision was made; the learning brain swaps in later).
- Geofencing that pings the closest nurse.
- The full consent system.
- Logins, multiple care homes, phone apps.

Saying this out loud clearly is a strength, not a weakness. Anyone serious will trust
"here's the working foundation and here's exactly how it grows" far more than someone
pretending the foundation is the whole house.

---

## 3. The rules we never break

1. **We never diagnose.** Every guess is soft: "pattern consistent with…", "may
   indicate…". Every guess ends with this exact line:
   `— not a diagnosis. Nurse judgment required.`
2. **No alert without evidence.** Every alert carries at least one piece of proof. The
   nurse can always click and see *why*.
3. **June is honest.** If the elder asks what she is, she says it warmly and plainly:
   she's a companion who keeps them company and can let the nurses know if they're
   unwell. She never pretends to be human, never gives medical advice, always points
   medical questions back to the nurses.
4. **June never says scary number stuff to the elder** ("your heart rate is high").
   That just creates fear. She only passes along nurse tasks, gently.
5. **The severity levels are frozen** (section 12). You can tune the rules. You can't
   change what the levels mean.
6. **Always compare a person to their own normal**, never to an average. Joseph's
   healthy oxygen is low because of his COPD. The system must know that.
7. **Silence is a signal.** If an elder goes quiet, or the band goes dead, that is
   *information*, not "all fine." The system must eventually notice (rule R8).

---

## 4. How to work with Claude Code (this part keeps you safe)

The human directing this can't read code to catch mistakes. So:

- **One feature per session.** Start each session by re-reading this file. Build one
  numbered thing from the build order. Don't wander.
- **Save your work after every small win.** `git commit -m "week2: dizzy creates a
  flag"`. Tiny saves are the only safety net — they let us undo a single broken thing
  instead of losing a day.
- **Never "clean up" or rewrite code that already works.** Copy-paste is fine. A clever
  rewrite that breaks something the human can't debug is not fine.
- **Never change the data shapes (`lib/types.ts`) without asking first.**
- **If you're stuck for ~30 minutes, stop.** Put a button that says "not wired up yet"
  and move on. Tell the human what you skipped. A small visible gap beats a hidden
  broken thing.
- **Wrap every outside call** (the language model, the database) so the app never shows
  a white error screen. If something fails, show a calm message and keep going.
- **Deploy it live in week 1 and keep it live.** Broken-and-live beats perfect-on-my-
  laptop.
- **End every session in plain English:** what works now, what's faked, and what to
  click to test it. The human checks by clicking, not by reading code.

---

## 5. The tech we're using (decided — don't re-litigate)

- **One Next.js app** (App Router, TypeScript). Several pages inside it.
- **Firebase / Firestore** for the database. It updates screens live by itself, which
  is how a new alert pops onto the nurse's screen the instant it's created. No fancy
  server functions.
- **No logins.** The home page just has three buttons: Elder / Nurse / Control Room.
- **The language model** goes through one small file, `lib/llm.ts`, so we can swap
  providers by changing one setting. Default for the hackathon build is the
  **Anthropic API (Claude)** — Haiku for fast extraction, Sonnet for June's
  conversation — funded by the event's API credits. Budget fallback if building
  outside the event: **Gemini's free tier** (note: the Google AI Pro student plan
  does NOT cover API calls), with rate limits handled gracefully (section 22).
- **Voice** uses the browser's built-in speech (Chrome only). A push-to-talk button:
  tap to talk, tap to stop. Typing always works too, as a backup.
- **Tailwind** for styling. **Motion** (the library, formerly Framer Motion) for
  animation. **Recharts** for the little trend graphs.
- **three.js** only if we want one fancy moment on the landing page. Otherwise skip it.
- **Lives on Vercel** (the app) + Firebase (the data).

The pages:

```
/                home — three buttons: Elder / Nurse / Control Room (+ link to landing)
/companion       the elder's screen (which elder is set by ?resident=ID)
/dashboard       the nurse's screen
/control         the control room — fake the wearable, run the demo scenarios
/resident/[id]   one elder's trends over time + their history
/landing         the public one-page pitch site
/api/companion   handles one chat turn (reply + spots flags + updates memory)
/api/evaluate    runs the brain for one elder (checks the rules, makes alerts)
/api/report      writes the end-of-day report for one elder
```

---

## 6. How the data is stored

Everything lives in flat Firestore collections. Every record carries which elder it
belongs to. Claude Code: match these shapes exactly. Don't change them without asking.

```ts
// residents/{id} — one elder
interface Resident {
  id: string;
  name: string;
  room: string;
  age: number;
  conditions: string[];            // used by the RULES only — never told to June
  baseline: {                      // this person's own "normal"
    hrRest: number;                // resting heart rate (bpm)
    spo2: number;                  // blood oxygen (%)
    sleepHours: number;
    sleepInterruptions: number;    // times woken per night
    repetitionNormal: boolean;     // true for dementia — repeating is normal for them
  };
  companionMemory: {
    summary: string;               // <=300 words, rewritten when a chat ends
    keyFacts: string[];            // <=20 facts, e.g. "daughter Priya visits Sundays"
    lastConversationAt: Timestamp | null;
  };
}

// vitalsSamples/{auto} — one reading from the (fake) wearable, every 30s while running
interface VitalsSample {
  residentId: string;
  ts: Timestamp;
  hr: number;
  spo2: number;
  state: 'awake' | 'asleep' | 'restless';
  battery: number;                 // %
  inRange: boolean;                // is the band near its base station
  source: 'sim';                   // later this becomes 'device' for real hardware
}

// nightSummaries/{auto} — one per elder per night
interface NightSummary {
  residentId: string;
  date: string;                    // YYYY-MM-DD
  totalHours: number;
  interruptions: number;
  quality: number;                 // 0–100
}

// cognitiveFlags/{auto} — a thing June noticed in how the elder talked
interface CognitiveFlag {
  residentId: string;
  ts: Timestamp;
  type: 'physical_complaint' | 'cognitive' | 'mood' | 'sleep_complaint'
      | 'appetite' | 'distress';
  subtype: string;                 // from the list in section 8
  quote: string;                   // the elder's exact words — this is the proof
  severityHint: 0 | 1 | 2 | 3;     // June's quick guess; the brain makes the real call
  confidence: number;              // 0–1
  source: 'companion';
}

// triageEvents/{auto} — an alert. Created ONLY by the brain (/api/evaluate)
interface TriageEvent {
  residentId: string;
  createdAt: Timestamp;
  updatedAt: Timestamp;
  ruleId: string;                  // which rule fired, e.g. 'R4' — used to avoid dupes
  channel: 'clinical' | 'ops';     // ops = battery/range stuff, kept in a separate tab
  severity: 0 | 1 | 2 | 3 | 4;     // section 12 — frozen
  title: string;                   // "Possible early infection pattern"
  hypothesis: string;              // soft language, ends with the section-3 line
  evidence: EvidenceItem[];        // always at least one
  status: 'open' | 'acknowledged' | 'resolved' | 'dismissed';
  ackBy?: string; ackAt?: Timestamp;
  resolution?: string;
}
interface EvidenceItem {
  kind: 'flag' | 'vital' | 'sleep';
  refId: string;
  summary: string;                 // "Asked the same question 3x in one chat (rare for her)"
}

// tasks/{auto} — the closed loop: a job the nurse sends back through June
interface CareTask {
  residentId: string;
  createdAt: Timestamp;
  createdBy: string;               // nurse's name, typed in
  instruction: string;            // "Encourage a full glass of water before bed"
  deliverVia: 'companion';
  status: 'pending' | 'delivered' | 'confirmed' | 'declined';
  deliveredAt?: Timestamp; confirmedAt?: Timestamp;
  linkedEventId?: string;
}

// sessions/{auto} — the back-and-forth of one conversation
interface CompanionSession {
  residentId: string;
  startedAt: Timestamp;
  messages: { role: 'user' | 'assistant'; text: string; ts: Timestamp }[]; // last 30
  engagement: number;              // 0–100, worked out ONCE when the chat ends
}

// dailyReports/{auto} — the end-of-day summary (section 14)
interface DailyReport {
  residentId: string;
  date: string;
  generatedAt: Timestamp;
  vitalsSummary: { hrAvg: number; spo2Min: number; sleep: NightSummary | null };
  flagCount: Record<string, number>;
  eventIds: string[];
  narrative: string;               // 3–4 soft sentences, written by the model
  signedOffBy?: string;            // nurse name — they just verify and sign (section 14)
  signedOffAt?: Timestamp;
}
```

**The four elders to start with** (`scripts/seed.ts`). Also build a **reset** function
that wipes everything back to this clean state, and put a reset button on the control
page — one tap to recover from a fumble during a live demo.

| Name | Age | Room | Conditions | The point of them |
|---|---|---|---|---|
| Mary Whitfield | 82 | 114 | mild arthritis | the "quiet infection sneaking up" story |
| Joseph Arsenault | 78 | 117 | COPD | his healthy oxygen is *93* — proves we use personal normals |
| Anna Li | 85 | 120 | early dementia | repeating is normal for her, so her bar is higher |
| Robert Gallant | 80 | 122 | none | the healthy one — proves the system stays quiet when all is well |

---

## 7. The golden wiring rule (the thing most likely to silently break)

The brain (`/api/evaluate`) only does something if something *calls* it.

**Rule: anything that writes new data must then call the brain for that elder.** Put
this in one helper, `lib/evaluate.ts`, so there's only one path.

- June writes a flag → call the brain.
- The control room writes a wearable reading or a night summary → call the brain.
- The "advance to next day" button → call the brain for every elder (this is how the
  "silence" rule can fire).
- A `distress` flag (someone says "I fell") → call the brain *right now*, immediately.

If you build a rule that watches the wearable but nothing calls the brain after wearable
readings, that rule will quietly never fire. Don't let that happen.

---

## 8. What June listens for

While chatting, June sorts anything notable into these buckets. She picks from this
fixed list — she never invents new categories.

| Bucket | The specific things |
|---|---|
| `physical_complaint` | dizziness, pain, nausea, breathing_difficulty, weakness, headache |
| `cognitive` | disorientation_time, disorientation_place, repetition, word_finding, unusual_confusion |
| `mood` | loneliness, hopelessness, anxiety, agitation, flat_affect |
| `sleep_complaint` | poor_sleep, nightmares, daytime_drowsiness |
| `appetite` | not_eating, not_drinking, nausea_food |
| `distress` | explicit_help, fall_mention, cant_breathe, chest_pain |

A `distress` bucket is special: June calls the brain immediately and replies with calm
reassurance ("I've let the nurses know — someone is coming"). She never tries to handle
it like a doctor.

---

## 9. What June does, step by step (/companion + /api/companion)

**Her personality:** warm, slow, simple. Short sentences. One question at a time. She
remembers things about the person. She talks about *their life*, not about the system.

**Each time the elder says something, the server does two model calls:**
1. **Reply** (Claude Sonnet): her personality + the last 30 messages → what she says back.
2. **Notice** (Claude Haiku): looks at what was just said and replies with a tidy

Then: save any flags → call the brain (section 7).

**Passing a nurse's task along:** if there's a pending task, June works it into the chat
naturally ("the nurses left a little note for us…"). When the elder agrees, the "notice"
step marks the task confirmed, and it flips to confirmed on the nurse's screen live.

**If the model is rate-limited:** June says "Give me a moment, dear — my thoughts are
slow today," and the elder can still type. Never a white screen.

**When a chat ends** (90 seconds quiet, or they close it): one model call rewrites her
memory summary and works out an engagement score for the session.

**The screen:** big and simple. Large friendly text (20px+). A push-to-talk mic button
that softly *breathes* (glows in and out) while listening. Typed input always visible.
She reads her replies out loud. (Heads-up for Claude Code: browser voices load late —
wait for the `voiceschanged` event before the first time she speaks, or it'll silently
fail.) No menus. Nothing clinical on screen. Warm lamp, not a hospital.

---

## 10. The fake wearable

The control room (section 16) pretends to be the wristband: sliders for heart rate and
oxygen, an awake/asleep/restless toggle, battery, in-range toggle, a "write a reading
now" button, and an auto-stream toggle (a reading every 30 seconds). Every write calls
the brain. Later, real hardware replaces this and nothing else changes.

---

## 11. The brain — how it decides (lib/rules.ts)

The brain looks at: the last 3 days of flags, the last day of wearable readings, the
last 3 nights of sleep, and the elder's personal normal. Then it checks these rules.

- **R1 — emergency (someone said it):** any `distress` flag → top alert (S4) if it's
  cant_breathe, fall_mention, or chest_pain; otherwise S3.
- **R2 — oxygen drop:** 3 readings in a row below (their normal − 3) → S3. Any reading
  below 90 *and* below (their normal − 4) → S4. (This is why Joseph's personal normal
  matters.)
- **R3 — heart racing:** 3 readings in a row above (their normal × 1.3) → S3.
- **R4 — the infection pattern (the headline story):** within 3 days, ALL of: at least
  one `cognitive` flag (respecting that repetition is normal for some), AND 2+ bad
  nights (woken at least 2 more times than normal), AND average heart rate up 10% →
  S2, titled "Possible early infection pattern." No single feed catches this. Together
  they do.
- **R5 — sinking mood:** 3+ loneliness/hopelessness flags in a week → S2.
- **R6 — dizzy + racing:** a dizziness complaint plus a heart-rate drift within 6 hours
  → S2.
- **R7 — equipment:** battery under 20%, or band out of range over 2 hours → S1, in the
  separate "ops" tab (never mixed with health alerts).
- **R8 — silence:** checked when a day advances. If an elder had no chat AND no change
  in their body readings for 12 hours → S2 wellness check. This one matters: it's how
  "no data" refuses to mean "fine."
- **R9 — the real emergency (nobody said anything):** the wearable shows something
  alarming (e.g. a sharp spike then a crash, or "restless" jammed on with a racing
  heart) AND June has gotten no response from the elder for a set window → top alert
  (S4). This is the fall/faint case: the person who's collapsed *can't* talk, so the
  proof is the silence plus the bad vitals together. (In the grown-up version this also
  pings the nearest nurse by location — not now, see section 21.)

**Don't spam (this is the whole answer to alarm fatigue):** before making a new alert,
check for an open alert for the same elder + same rule. If one exists, just add the new
proof to it and bump its time — never make a second copy. Once a nurse resolves or
dismisses an alert, that rule stays quiet for that elder for 6 hours.

**One setup chore, do it in week 1:** the nurse's main list needs a Firestore composite
index (it filters and sorts at the same time). Firestore will error the first time with
a link to create it. Save a `firestore.indexes.json` up front so this never bites mid-
demo.

---

## 12. The severity levels (frozen — never change the meanings)

| Level | Name | What it means | What the nurse sees | Color |
|---|---|---|---|---|
| S0 | Ambient | normal, just learning | nothing; stored only | — |
| S1 | Note | one mild thing | shows in the daily report only | slate `#64748B` |
| S2 | Review | a cross-signal pattern, not urgent | a card in the queue, no sound | amber `#D97706` |
| S3 | Check soon | check within 15 min | card + pinned to top + a chime | orange-red `#EA580C` |
| S4 | Now | emergency | full-width red banner + a tone until acknowledged | red `#DC2626` |

---

## 13. The nurse's screen (/dashboard)

This is a tool, not a pretty page. Tight rows, real info, no decoration. The only colors
on the page are the severity colors — color always *means* something here.

- **The live queue:** open health alerts, most urgent on top, newest next. It updates by
  itself. An S4 takes over as a full-width banner.
- **Each card:** severity badge, name + room, title, time, and how many pieces of proof.
  Click it → a panel slides out with the soft hypothesis, every piece of proof (with the
  elder's exact words where relevant), buttons to Acknowledge / Resolve / Dismiss, and a
  "Create task" button that's pre-filled from the alert.
- **Ops tab:** the equipment stuff (battery, out of range), kept totally separate.
- **Elder strip:** all four elders with a colored dot showing their worst open alert →
  click through to their trends.
- **Tasks panel:** make a task → watch it go pending → delivered → confirmed, live, as
  June does her thing.
- **Daily report button:** per elder, generates the report (section 14).

---

## 14. The end-of-day report (/api/report)

At day's end the system writes a short report per elder by itself: average heart rate,
lowest oxygen, sleep, the flags that came up, any alerts — turned into 3–4 soft
sentences for handover. The key principle: **the nurse adds nothing by hand.** They read
it, and they sign it off (one button → their name + time stamped on it). No paperwork,
no extra typing — that's the last thing an overloaded caregiver needs.

These reports pile up. Over weeks and months they become the trend a doctor can actually
read — a real picture over time, instead of guessing from one 15-minute visit.

---

## 15. The trends page (/resident/[id])

Little line graphs (Recharts): heart rate per day, lowest oxygen per day, night wake-ups,
flags per day, engagement per chat. Below them, a time-ordered list of alerts and daily
reports. This is the seed of the long-term doctor view. Don't build more than this now.

---

## 16. The control room (/control)

Where you puppeteer the demo. Per elder: the wearable sliders/toggles from section 10,
the reset button, and one-click scenario buttons that run the story:

- **UTI Day 1 (Mary):** last night → woken 4 times, heart rate drifting up 8%.
- **UTI Day 2 (Mary):** woken 5 times, up 12% → plus one confusion flag from a chat,
  rule R4 fires.
- **Fall (anyone):** writes alarming vitals while June gets no answer → tests R9.
- **COPD dip (Joseph):** oxygen to 91 (still fine *for him*), then 89 (now it's serious)
  — the personal-normal story.
- **Quiet day (all):** everything normal — proves the system stays silent.
- **Advance a day:** writes last night's sleep from the current settings, then calls the
  brain for everyone (so silence can fire).

---

## 17. How it should look and feel

This is the part you're best at, so this is where the product earns trust. Three screens,
three completely different moods. Claude Code: use the `frontend-design` skill for the
styling rules before building any screen.

- **June's screen — lamplight, not hospital.** Deep warm charcoal background. A soft amber
  glow around the mic button, like a reading lamp. Big, friendly, humanist text (try
  Atkinson Hyperlegible — it's designed for low vision). The breathing mic glow is the
  one signature animation. Everything else stays still. Calm beats clever.
- **The nurse's screen — cool, dense, clinical.** Near-white, slate text. Severity colors
  are the *only* color. Sharp corners, no gradients. Numbers in a monospace font so they
  line up. One bit of motion: a new card slides in once. That's it.
- **The landing page — confident and quiet.** The hero is the idea, not a stock photo:
  "Care systems are reactive. We make them proactive." One animated diagram of the loop
  (this is the one page where motion is allowed to show off — scroll-triggered reveals
  are good here). The infection story as a 3-step strip. The five-people grid from the
  master doc. One button: capture an email. No fake logos, no fake testimonials, no
  pricing. One optional three.js moment — cut it the second it costs more than an hour.

Whole-app motion rule: motion either makes something clearer or it's gone. June and the
nurse screen stay nearly still. The landing page is the only place animation performs.

---

## 18. Honesty and consent (the small version)

The big open question for this whole field is consent: does the elder know the friendly
companion is also a health sensor, and did they agree? We can't fully solve that now, but
we refuse to hide it: June is honest when asked (section 3), the landing page says plainly
what the system is and isn't, and `LIMITATIONS.md` writes down that full consent is a
real Phase 2 system we've thought about — not an afterthought. Showing you see this clearly
is worth more in the pitch than pretending it's solved.

---

## 19. The build order (~8 weeks, part-time around factory shifts)

*Note: this table is the full V1 journey. The July 7–13 hackathon sprint targets a narrower slice — the closed loop end to end, once (weeks 1–2 plus the cores of 4–7) — and the ordering and cut rules below still govern it.*

Hard deadline: moving to Waterloo, mid-to-late August. "DONE" means something the human
can check by clicking — not by reading code. Weeks are loose on purpose so one rough week
doesn't sink it.

| Week | Build | DONE when |
|---|---|---|
| 1 | Firebase + Next.js set up, data shapes, seed + reset, home buttons, a raw debug page, the index. **Go live on Vercel.** | the four elders show up at a live web link; a reading you type in shows up on screen by itself |
| 2 | June as a TEXT chat: the chat API, her personality, memory, the "notice" step → flags. Fold in the June voice work you and Steve already have | typing "I feel dizzy" creates the right flag with the exact words saved |
| 3 | Add voice (push-to-talk in/out) + make June's screen beautiful (the lamplight look, breathing mic) | a full spoken conversation works in Chrome on the live link |
| 4 | The control room: sliders, write-now, auto-stream, advance-a-day, reset — all calling the brain | sliders make readings; advancing a day writes a night; reset cleans up |
| 5 | The brain: all the rules + dedup + the wiring from section 7 | the UTI story makes exactly ONE growing alert; the COPD dip fires correctly |
| 6 | The nurse screen: live queue, the proof panel, ack/resolve/dismiss, ops tab, elder strip | walkthrough steps 1–4 work start to finish |
| 7 | The closed loop: make a task → June delivers it → elder confirms → it flips live | walkthrough step 5 works without ever touching the database by hand |
| 8 | Scenario buttons, trends page, daily report + sign-off, landing page, README + LIMITATIONS, record the 90-second walkthrough, tag it v1.0 | every demo click is one button; the live product tells its own story in 90 seconds |

**If you fall behind** (check honestly at the end of week 5, not week 8): cut in this
order — daily-report wording → trend graphs → voice polish → landing animations → the
three.js moment. **NEVER cut:** the proof panel, the no-spam logic, the closed loop, the
personal-normal comparison. Those four ARE the product.

**Running alongside the build (this is the human's job, not Claude Code's):** keep talking
to real caregivers — about how they decide who needs attention, and whether early signs
get missed. A handful of real conversations is worth more to Velocity than another feature,
and the quotes belong in the README and the pitch.

---

## 20. Quick rules for every Claude Code session

Start with: "Read CLAUDE.md. This is week N. Build section X." One feature. Save after
every win. Don't rewrite working code. Don't change data shapes without asking. Stub
after 30 minutes stuck. Wrap every outside call. Make sure reset + seed make every demo
click work fresh. End in plain English: what works, what's faked, what to click.

---

## 21. The grown-up version (NOT now — this is what grows at Waterloo)

Real wearable hardware · a self-learning brain that replaces the rules (same inputs and
outputs, so it swaps in cleanly) · showing severity as a percentage instead of/alongside
levels (a real design choice for later) · geofencing to ping the nearest nurse in an
emergency · the full consent and disclosure system · keeping data private on-device so
only flags ever leave the room · logins, many care homes, a family app, medication, phone
apps, other languages. The rules brain in section 11 is the deliberate, swappable stand-in
for the learning brain — build the foundation so the brain transplant later is clean.

---

## 22. Money — aim for $0

The app's model calls run on the **Anthropic API** during the hackathon (covered by
event credits); the $0 fallback is **Gemini's free tier** (again: the student AI Pro
plan does NOT cover API calls). On the fallback:
- All model calls go through `lib/llm.ts`, which retries politely when rate-limited
  (wait 1s, 2s, 4s) and degrades gracefully. Free tier has per-minute limits — fine for
  one user, but warm it up before recording the walkthrough so a limit doesn't hit mid-take.
- Free-tier text may be used by Google to train. Fine here because every elder is made up.
  Write that in LIMITATIONS.
- Firebase free plan only. Don't leave the auto-stream running overnight.
- Vercel free plan. A `something.vercel.app` link is fine.

Two ways to remove the rate-limit annoyance at little/no cost, worth checking before you
rely on them: the ~$10/month Google Cloud credits that come with AI Pro might cover the
API through Vertex; or a few dollars of paid Gemini API. Neither is required. If any step
suddenly demands a credit card, stop — something drifted off plan.

---

## 23. The 90-second walkthrough (build everything to make this true)

Not a fake demo — a quick tour of the real thing running live:

1. **Calm:** the nurse screen is quiet, Robert's having a normal day. "No alarms. It stays
   silent until it has proof."
2. **June, live:** chat with June as Mary. Mention sleeping badly; ask the same question
   twice. She's warm; flags appear in the database in real time.
3. **Control room:** UTI Day 1, advance, UTI Day 2, advance.
4. **The moment:** rule R4 fires. An amber card slides onto the nurse screen — "Possible
   early infection pattern, 4 pieces of proof." Open it: the bad nights, the heart rate,
   and Mary's exact words. "No single feed would have caught this."
5. **The loop closes:** the nurse makes a task — "encourage fluids before bed." Flip to
   June; she works it in. Mary says "alright, I'll drink it now." The task flips to
   confirmed on the nurse screen, live.
6. **The contrast:** Joseph at 93 oxygen — no alarm (it's his normal). At 89 — serious.
   "Personal normals, not averages."

End on: "Caught a week before the 3 a.m. ambulance. The nurse decides. The system points."
