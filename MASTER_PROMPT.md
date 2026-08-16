# MASTER_PROMPT — Study Buddy: Offline-First Android Study Companion

**Read this entire file before writing any code.** This document is
self-contained by design — assume no prior conversation memory. Companion
files in this folder (`master_specification.md`, `phase0-harness/`,
`design-demo/study-buddy-demo.jsx`) are supporting reference material, not
required reading to understand the mission — everything essential is
restated here.

---

## 0. Mission (one paragraph, no ambiguity)

Build an offline-first Android app that lets underprivileged students —
mostly in rural India, CBSE/NCERT curriculum, classes 6–12 — study with a
locally-run LLM on their own phone, with zero setup, zero required internet,
and zero account creation. An optional, tightly gated online mode adds
occasional help from a larger AI model, rationed to a daily limit and
filtered to schoolwork only. The app must run acceptably on 4GB-RAM
hardware, stay under 2GB total size, and be simple enough for a child to
operate without instruction.

---

## 1. SMART Goals

Each goal below is Specific, Measurable, Attainable given current validated
state, Relevant to the mission in Section 0, and Time-bound relative to the
milestone that closes it (not a calendar date — pace is session-driven, not
deadline-driven).

| # | Goal | Bound to |
|---|---|---|
| G1 | Run `phase0-harness` on ≥1 real Android device with ≤4GB RAM; record load time, RSS, native heap, pp/tg bench tokens/sec, and real end-to-end chat tokens/sec. Decide the model tier from evidence, not assumption. | Closes M1 |
| G2 | Ship a production inference module (wrapping the M1-validated engine) that sustains a 20-turn chat conversation with no crash and no unbounded memory growth on the test device. | Closes M2 |
| G3 | Populate Room schema + versioned JSON content packs covering Mathematics, Science, English, and Social Science for classes 6–12, in English and Hindi, with at least one full chapter's content loading and rendering correctly. | Closes M3 |
| G4 | Ship Compose implementations of Dashboard, Planner, and Wins that match the approved design system in `design-demo/study-buddy-demo.jsx` (visual language, not literal ported code), including the editable weekly goal and syllabus-progress feature. | Closes M4 |
| G5 | Ship an offline test generator producing curriculum-templated MCQs for ≥2 subjects × ≥2 chapters each, with correct end-to-end scoring on-device. | Closes M5 |
| G6 | Achieve full English/Hindi UI parity across every screen (not just navigation labels) — zero untranslated strings in a complete app walkthrough in Hindi mode. | Closes M6 |
| G7 | Implement the online-mode defense-in-depth pipeline (local classifier gate → provider call → quota) such that a non-educational query is blocked with **zero network calls made**, and an educational query correctly round-trips through the chosen provider. | Closes M7 |
| G8 | Produce a signed release build under 2GB that installs and runs on the lowest-spec device tested in M1, with Play Console's Data Safety and content-rating forms completed to truthfully match the app's actual data practices (not a generic template). | Closes M8 |

---

## 2. Goal-Oriented Architecture — traceability matrix

Every module must trace back to a mission objective. If you find yourself
building something that doesn't map to a row below, stop and ask whether it
belongs in this version of the app at all.

**Mission objectives:**
- **MO1** — Works fully offline; zero setup; no connectivity dependency
- **MO2** — Runs acceptably on low-end hardware (4GB RAM, ≤2GB app)
- **MO3** — Simple enough for a child to operate unaided
- **MO4** — Delivers real curriculum-aligned educational value
- **MO5** — Online enhancement is safe, scoped, and never a dependency
- **MO6** — Multilingual, extensible beyond Hindi/English later
- **MO7** — Advanced-user extensibility without complicating the default experience

| Module | Primary objective(s) |
|---|---|
| `core/inference` (llama.cpp/JNI) | MO1, MO2 |
| `core/data` (Room + DataStore) | MO1 (no cloud dependency), MO4 |
| `core/content` (JSON content packs) | MO4, MO6 |
| `feature/chat` | MO1, MO3, MO4 |
| `feature/dashboard` | MO3, MO4 |
| `feature/planner` | MO4, MO3 |
| `feature/testgen` | MO4, MO1 |
| `feature/wins` | MO3, MO4 |
| `feature/online` (classifier, quota, router) | MO5 |
| `feature/settings` | MO7, MO5, MO6 |

---

## 3. Complete feature inventory

Every feature below must exist in the shipped v1. Nothing here is optional
polish — this is the full scope, restated in full so nothing gets quietly
dropped.

### 3.1 Core (always active, fully offline)
- llama.cpp/JNI inference engine wrapping the Phase-0-validated native build
- Default GGUF bundled at install (Q4_K_M, tier decided at M1 from real data)
- RAM-aware startup check with graceful context/thread degradation on
  constrained devices — not an afterthought, built in from M2
- Room database: student profile, class, curriculum progress, planner state,
  test history
- DataStore: language preference, selected model, online-mode toggle state
- Content packs: versioned JSON, not hardcoded — Math, Science, English,
  Social Science, CBSE/NCERT classes 6–12, English + Hindi
- Pre-fed motivational content (quotes) as part of the content pack

### 3.2 Chat (`feature/chat`) — Local-AI-inspired
- Minimal message-bubble UI
- Offline/ready status badge and current-model name pill (tap → Settings)
- Token-by-token streaming response rendering with a typing indicator
- Real tokens/sec readout shown subtly beneath each AI reply
- Inline online-mode toggle with visible daily-quota remaining
- Enter-to-send and tap-to-send

### 3.3 Dashboard (`feature/dashboard`) — Physics-Wallah-inspired
- Greeting header with student name and class/board
- Streak indicator (flame icon + weekly star row)
- "Continue where you left off" hero card, deep-links into the relevant
  chapter/chat context
- Quick-action grid: Ask a Doubt, Practice Test, My Plan, My Wins
- Subject-wise curriculum progress bars
- Quote-of-the-day card
- Header controls: language toggle, notifications, settings

### 3.4 Planner (`feature/planner`)
- Weekly schedule (day, subject, topic, completion checkbox)
- Editable weekly goal (tap to edit inline, not just a static banner)
- Syllabus-progress section showing per-subject completion, sharing its data
  source with the Dashboard (single source of truth, not a duplicate)

### 3.5 Test Generator (`feature/testgen`)
- Subject and chapter selection via chip UI
- Offline curriculum-templated MCQ generation (local model + structural
  templates, so output stays reliable even from a small model)
- Immediate per-question feedback on submit
- Score summary with star rating; "Try Again" and "Ask about this" (deep
  link into Chat) actions
- **Advanced analytics**: online-gated deeper performance advice — this is
  the original "connect to the internet and get advice from the best
  models" requirement, implemented via the Insights section in Settings,
  not as a separate always-on network feature

### 3.6 Wins / Achievements (`feature/wins`)
- Badge grid: earned vs. locked, with the unlock criteria shown for locked
  badges
- Entirely offline — no online gate, deliberately, since these are the
  motivational payoff for using the app at all, not a premium feature
- Bilingual badge content

### 3.7 Settings (`feature/settings`)
- Language selector (English/Hindi now, structured to add more later)
- Model section: switch among bundled models; separate "Add a model file"
  affordance for advanced users importing their own GGUF (not pre-bundling
  every possible model)
- Online connection: toggle, quota display, and plain-language explanation
  of what the content filter actually does (a parent or teacher should be
  able to read it and understand the safety model)
- Insights: always-visible offline stats (questions asked, tests taken,
  average score, streak) plus online-gated personalized AI advice — the
  split matters, don't collapse it into one always-online feature

### 3.8 Online mode (`feature/online`) — defense in depth, not a single filter
1. Local topic-relevance classifier gate runs **before** any network call —
   non-educational queries never leave the device
2. Restrictive, subject/class-scoped system prompt for whatever passes the
   gate
3. Provider-side safety settings as a second, independent layer
4. Daily quota tracked on-device, resets locally
5. In-app flagging/reporting for AI output (required by Google Play policy
   for AI-content apps, and good practice regardless of distribution
   channel)
6. **Provider is not yet chosen** — verify current free-tier terms at
   implementation time; do not hardcode an assumption from this document

### 3.9 Platform, packaging, and compliance
- Native Kotlin, Jetpack Compose
- Total app size ≤2GB (approximate budget: ~80–120MB code/libraries,
  ~1.2–1.5GB default model, ~50–150MB content packs, remainder buffer)
- Acceptable performance on 4GB-RAM devices; graceful degradation below that
- `minSdk = 24` — a reasoned guess based on rural/budget devices skewing
  older, not a verified number; revisit if better data becomes available
- Distribution: direct APK as the primary channel (NGO/school distribution,
  since a multi-GB Play Store download is itself a barrier on mobile data),
  Google Play as a secondary channel for legitimacy and updates
- Play policy compliance built in regardless of channel: AI-content
  disclosure, in-app reporting, honest Data Safety questionnaire, honest
  content rating
- Privacy posture: single-device only, no accounts, no cloud sync by
  default — this is a deliberate choice that keeps the app largely outside
  India's DPDP child-data consent requirements; do not reopen this by adding
  a backend, analytics SDK, or account system without flagging it explicitly

### 3.10 Explicitly out of scope for v1 — do not build these without asking
- Guardian/teacher account layer
- Sarvam-1 instruction fine-tune (tracked separately as a v2 research
  direction — Sarvam-1's base release is completion-only, not chat-ready,
  and fixing that is its own project)
- Curriculum boards other than CBSE/NCERT, or classes outside 6–12
- Languages beyond English and Hindi

---

## 4. Milestones (exact, checkable)

Work through these in order. Do not start a milestone before the previous
one's Definition of Done is fully met.

**M0 — Environment and access ready**
- [ ] Project pushed to a GitHub repo; URL recorded in this repo's README
- [ ] Current, verified Hugging Face repo links recorded for every candidate
      GGUF model (see Section 5 — this is a standing task, not optional)
- [ ] `phase0-harness` confirmed building in Android Studio (already true as
      of this document; reconfirm if time has passed)

**M1 — Phase 0 validated** *(hard gate — nothing below starts until this closes)*
- [ ] Real on-device run completed on ≥1 device at or below 4GB RAM
- [ ] Load time, RSS/native heap, pp/tg bench, and real chat timing recorded
- [ ] Model-tier decision made explicitly, with the evidence, and written
      into `master_specification.md`, replacing the "reasoned guess" framing

**M2 — Offline inference core**
- [ ] Production inference module passes a 20-turn chat smoke test with no
      crash, no unbounded memory growth
- [ ] RAM-aware fallback behavior implemented per the M1 decision

**M3 — Data and content layer**
- [ ] Room schema implemented and migrated cleanly
- [ ] Content packs populated for all four subjects, classes 6–12, EN + HI
- [ ] At least one full chapter loads and renders correctly end-to-end

**M4 — Dashboard, Planner, Wins UI**
- [ ] All three screens implemented in Compose, matching the design-demo
      visual system
- [ ] Editable weekly goal and syllabus-progress section functional
- [ ] Wins badges wired to real local stats, not placeholder data

**M5 — Test generator**
- [ ] Offline MCQ generation working for ≥2 subjects × ≥2 chapters
- [ ] Scoring correct end-to-end on-device

**M6 — Language layer**
- [ ] Full walkthrough in Hindi mode with zero untranslated strings anywhere
      in the app, not just nav/headers

**M7 — Online mode and Settings**
- [ ] Non-educational query provably blocked with zero network calls
- [ ] Educational query round-trips correctly through the chosen provider
- [ ] Quota enforcement and daily reset verified
- [ ] In-app flagging present and functional
- [ ] Settings screen fully wired: model switch, custom GGUF import, online
      toggle, Insights split (offline stats always visible, advice gated)

**M8 — Packaging and compliance**
- [ ] Signed release build, confirmed under 2GB
- [ ] Runs on the lowest-spec device available from M1's test pool
- [ ] Play Console Data Safety and content rating completed truthfully
- [ ] Privacy policy drafted
- [ ] Direct-APK build variant confirmed working standalone

---

## 5. Standing task: surface model and repository access

Before or during M0, and again any time a model decision changes:

1. Search for and verify the **current** Hugging Face repository links for
   each candidate GGUF model — do not rely on training data for exact repo
   paths or quantization filenames, as these change. At minimum cover:
   Gemma-2-2b-it (GGUF), Qwen2.5-1.5B-Instruct (GGUF), and Sarvam-1 (GGUF,
   noting its completion-only status per Section 3.10).
2. Write these into a `REFERENCES.md` in the project root: repo URL, exact
   filename for the Q4_K_M variant, file size, and last-verified date.
3. Confirm the GitHub repository for this project exists and is reachable;
   report the URL back to the user in your first status update.
4. If any of this requires credentials the current environment doesn't
   have (a Hugging Face token, GitHub auth), say so explicitly rather than
   silently skipping the step.

---

## 6. Step-by-step execution pipeline

For every milestone, follow this loop — don't skip steps under time
pressure, especially the checkpoint:

1. **Plan** — restate the milestone's Definition of Done in your own words
   before writing code, so scope drift gets caught early.
2. **Implement** — build in the smallest coherent increment that moves
   toward the DoD.
3. **Self-verify** — actually run/build/test what you wrote. Reporting
   something as working without having run it is not acceptable at any
   milestone.
4. **Checkpoint with the user** — stop, summarize what changed and what you
   verified, and surface any assumption you had to make. This matters most
   at M1 specifically, since real-device behavior is the entire point of
   this project and cannot be simulated from an IDE.
5. **Update `master_specification.md`** — if a decision was made or an
   assumption resolved, record it there so it stays the single source of
   truth for future sessions.
6. **Advance** — only move to the next milestone once its checklist is
   fully checked, not "mostly done."

---

## 7. Ground rules (apply throughout, no exceptions)

- **The design demo is a visual reference, not source code.** Match its
  design language in real Compose UI; do not port the JSX.
- **Every flagged assumption is a guess, not a decision**, until evidence
  says otherwise (`minSdk`, model tier, online provider, etc.).
- **Verify, don't assume** — you have a real toolchain now; use it before
  claiming something works.
- **Work in reviewable chunks.** Long-horizon autonomy is for depth, not for
  disappearing and returning with an unreviewable wall of changes.
- **The privacy/compliance posture in Section 3.9 is locked**, not a
  default to casually change.
- **This app is for children in low-resource settings.** Prefer the robust,
  simple solution over the clever one whenever the two conflict.

## 8. Your first message back to the user

A plain-language status of M0/M1, the Hugging Face and GitHub links from
Section 5, and the one concrete next action needed from the user. Not a
wall of code.
