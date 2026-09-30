# PLAN.md — Hearth Table (working title, rename freely)

A web app where a tabletop RPG group creates a campaign space, keeps character sheets synced, rolls dice, and chats — everything updates live for everyone at the (virtual) table.

> **Status:** Phase 1 output (planning only — no app code exists yet). Nothing gets built until George gives the go-ahead. His 2026-09-30 review answers are folded in; the ruleset question (Q1) stays **deliberately open** while he researches the genre (~6 months, see §1). Decision statuses live in `USER_TODO.md`.

---

## 1. What we're building (MVP scope)

### Game concept ⚠️
- **System (working placeholder — final choice deliberately deferred):** D&D 5e-*lite* — the popular core only: six ability scores with computed modifiers, HP, AC, proficiency bonus, skills, attacks, and a simple spell list. Concretely 5e-shaped, but the sheet's fields live in **one template file** (`characterTemplate`), so switching systems later is a template change, not a rewrite. We are **not** building a system-agnostic engine (agreed trap for v1).
- **Ruleset status (George, 2026-09-30):** 5e-lite is a placeholder, not the decision — he hasn't researched TTRPGs yet and will, with his coursemates, over **~6 months (decision target ≈ March 2027)**. The build order is shaped so this is cheap: **M0–M2 (scaffold, auth, campaigns) are ruleset-independent**; M3+ consume the template file, so the decision only has to land before M3.
- **Session shape (confirmed by George, Q2):** the GM creates the campaign, picks the ruleset and the setting, and shares it; players join, fill in their sheets, and get into character for the story. The campaign doc therefore carries a `setting` string (plain text in v1) for the GM's pitch: world, tone, house rules.
- **Tone:** friendly utility. Dark theme, big touch targets — this gets used on phones at a physical table.
- **Session flow:** GM creates a campaign → shares a 6-character join code → players join → everyone keeps their sheet synced live → anyone rolls dice and the result lands in the shared roll log → chat for table talk.

### What players do
Create/join a campaign, edit **their own** character sheet, roll dice (standard notation + advantage/disadvantage), see everyone's rolls and chat live, and view party members' sheets read-only.

### What the GM does (host = campaign creator)
Everything players do, plus: edit any sheet, remove members, delete chat messages. GM-only settings live on the campaign page.

### MVP features (5, locked)
1. **Accounts** — email/password + anonymous guest login, display-name profile.
2. **Campaign spaces** — create / join via code, member roster, GM role.
3. **Character sheets** — 5e-lite template, live-synced, derived stats computed (modifiers, proficiency, etc.).
4. **Dice roller + shared roll log** — notation parser (`2d6+3`, advantage/disadvantage on d20s), results append for the whole table.
5. **Table chat** — live messages alongside the roll log.

### Non-goals for v1 (explicit)
Native Android/iOS apps (designed-in for later — see §8), maps/tokens/VTT, fog of war, initiative tracker, voice/video, homebrew sharing, multiple rulesets, public matchmaking, payments, custom domain, GM transfer/leave-campaign flows.

---

## 2. Stack (locked)

| Piece | Choice | Version verified 2026-09-30 |
|---|---|---|
| UI framework | React | 19.3 |
| Build tool | Vite | 8.3.x |
| Language | TypeScript (recommended — decision D2) | 5.x (ships with Vite template) |
| Backend | Firebase **Auth + Firestore**, modular v9+ API (the v12 SDK) | 12.19.0 |
| Hosting | Firebase Hosting (free `*.web.app` subdomain) | — |
| Styling | Tailwind CSS (recommended — decision D3), mobile-first | 4.3.x |
| Local state | Zustand (stores fed by Firestore `onSnapshot`) | 5.x |
| Routing | react-router | 7.x |
| Tests | Vitest (dice parser, derived stats, security rules) | latest at M0 |

- Exact versions get **pinned in `package.json` at M0** and recorded in `DEVLOG.md` — the numbers above prove they're current today, the pins freeze them.
- **Free tier only (Firebase Spark plan):** email/password + anonymous auth are free; **phone auth is NOT free on Spark** (excluded). Firestore free allotment: **50,000 reads/day, 20,000 writes/day, 20,000 deletes/day, 1 GiB storage** — no credit card required. Firebase Hosting free tier covers the `*.web.app` subdomain. Spark **cannot bill you** (no card on file) — worst case is quotas pausing until midnight PT, never a charge.
- Do **not** use the new "Firebase App Hosting" product (requires Blaze/paid). Plain static Hosting on the Vite build output is what we want.

---

## 3. Architecture

No server code in v1. React app talks straight to Firebase; correctness comes from the security rules + a thin service layer.

```
React components
   ↑ read                ↓ write
Zustand stores  ←——  onSnapshot listeners (per feature hook)
                              ↓
                    service layer (src/features/*/service.ts)
                              ↓
                         Firestore  (security rules are the real gatekeeper)
```

Rules of the road:
- **One Firebase init point** (`src/lib/firebase.ts`), config from `.env` variables.
- Components never call Firestore directly — all reads go through feature hooks (`useCharacters`, `useRolls`, …), all writes through feature service functions. This gives one place per feature for validation, and one obvious surface to review in PRs.
- Derived stats (ability modifiers, proficiency bonus, skill bonuses, spell save DC) are **pure functions with unit tests**, computed from stored raw values — never stored twice.
- Every agent (me, Claude, Codex, friends' agents) must be fed this file + `DEVLOG.md` + `CONTRIBUTING.md` (written at M0) before touching code. Modular v9+ imports only (`import { doc, setDoc } from "firebase/firestore"`), never the namespaced `firebase.xxx` v8 style, never `compat` imports.

### Folder structure
```
ttrpg/
├─ docs/                  # PLAN.md, USER_TODO.md, DEVLOG.md, CONTRIBUTING.md (at M0)
├─ firestore.rules        # security rules — the real gatekeeper
├─ firestore.indexes.json # none needed for v1 (single-field indexes are automatic)
├─ firebase.json          # hosting config (added at M6)
├─ .env / .env.example    # Firebase web config (example committed, real file gitignored)
└─ src/
   ├─ lib/                # firebase.ts, env.ts
   ├─ routes/             # app shell, router, auth guards
   ├─ stores/             # Zustand stores fed by listeners
   ├─ components/ui/      # small shared components (Button, Modal, StatBox…)
   └─ features/
      ├─ auth/            # login, register, guest, useAuth
      ├─ campaigns/       # list, create, join-by-code, roster, useCampaign
      ├─ characters/      # sheet UI, characterTemplate.ts, derived.ts, service
      ├─ dice/            # parser.ts (pure), roller, roll log UI
      └─ chat/            # message list + composer
```

### Screens (phone-first)
1. **Landing / Login** — register, email login, "play as guest".
2. **Campaigns** — list of your campaigns + "create" + "join with code".
3. **Table (campaign home)** — bottom tab bar on mobile: **Party** (all sheets), **Rolls** (dice + log), **Chat**, **Settings** (GM-only).
4. **Sheet editor** — full-screen edit view for a character.

---

## 4. Firestore data model

```
users/{uid}
  displayName: string, createdAt: timestamp

campaigns/{campaignId}
  name: string
  system: "5e-lite"                 // ruleset placeholder until the group's Q1 decision
  setting: string                   // GM's shared pitch: world, tone, house rules (plain text v1)
  gmUid: string                     // creator
  memberUids: string[]              // GM included; used directly by security rules
  joinCode: string                  // 6 chars, unambiguous alphabet (no 0/O/1/I/L)
  createdAt: timestamp

campaigns/{cid}/characters/{charId}
  ownerUid: string
  name, charClass, level, race, background: string
                                    // "charClass", not "class" — `class` is a reserved
                                    // word in JS/TS AND in Kotlin (the future Android app),
                                    // so the field gets a safe name on every platform
  abilities: { str, dex, con, int, wis, cha }   // RAW scores; modifiers are derived
  hp: { current, max, temp }
  ac: number
  skills: string[]                  // proficient skill ids
  attacks: [{ name, bonus, damage }]
  spells: [{ name, level, prepared }]           // deliberately simple in v1
  castingAbility: "int" | "wis" | "cha" | null  // for derived spell save DC
  equipment: string, notes: string
  updatedBy: string, updatedAt: timestamp

campaigns/{cid}/rolls/{rollId}
  rollerUid, rollerName: string     // display name denormalized → zero extra reads to render
  notation: string                  // "2d6+3"
  mode: "normal" | "advantage" | "disadvantage"
  result: { total: number, terms: [...] }       // full breakdown kept for the log
  createdAt: timestamp (serverTimestamp)

campaigns/{cid}/messages/{msgId}
  authorUid, authorName: string, text: string (≤500 chars)
  createdAt: timestamp (serverTimestamp)
```

### What the sheet covers — v1 vs the rest of the "TTRPG stat family"

The v1 sheet carries the **combat core**: identity (name, class, level, race, background), the six ability scores with derived modifiers, HP (current/max/temp), AC, proficient skills, attacks, a simple spell list + casting ability, and equipment/notes as free text.

Deferred to **v1.1+** (listed explicitly so nothing looks forgotten — these are the remaining members of the stat family):
- **Conditions / status effects** (Poisoned, Stunned, Frightened…) — shown on the party view, GM can apply/remove
- **Spell slots** — note: 5e has **no MP**; casters spend per-level slots that refresh on a rest. MP/mana exists in *other* systems, which is exactly why the stat list follows the ruleset decision
- **Inventory + currency** (with optional carry weight)
- **Death saves, inspiration, rest tracking** (short/long), XP/level-up progression

Systems other than 5e swap this vocabulary (MP/mana, stamina, stress, meta-currencies…) — the sheet template is one data file, so the group's Q1 decision re-shapes the fields **without touching the architecture**. The final stat list is a ruleset decision, not an app decision.

Design notes:
- **Derived values are never stored** — `derived.ts` computes modifiers `floor((score − 10) / 2)`, proficiency bonus from level, skill bonuses, passive perception, spell DC. Pure + unit-tested. This kills a whole class of sync bugs.
- **Display names are denormalized** into rolls/messages at write time, so rendering the log costs one read per item, not one extra lookup per author.
- **Join flow:** player enters code → app queries `campaigns` where `joinCode == X` (single-field index, automatic) → `runTransaction` adds the uid to `memberUids` (guards: campaign exists, not already a member, roster cap **8 — confirmed by George, Q3**; one constant, trivially changeable).
- **Indexes:** none needed beyond the automatic single-field ones in v1.

---

## 5. Security rules approach

Principles: **default deny, auth required for everything, campaign membership is the gate.** Sketch (finalized in `firestore.rules` and emulator-tested before each milestone ships):

```js
match /users/{uid} {
  allow read: if request.auth != null;
  allow create, update: if request.auth.uid == uid;
  allow delete: if false;
}
match /campaigns/{cid} {
  allow read: if isMember(cid);
  allow create: if request.auth.uid == request.resource.data.gmUid;
  allow update: if isGM(cid)
    || (signedIn()
        && request.resource.data.diff(resource.data).affectedKeys().hasOnly(['memberUids'])
        && !(request.auth.uid in resource.data.memberUids)
        && request.auth.uid in request.resource.data.memberUids);   // self-join only
  match /characters/{charId} {
    allow read: if isMember(cid);
    allow create, update: if isMember(cid)
      && (request.resource.data.ownerUid == request.auth.uid || isGM(cid));
    allow delete: if resource.data.ownerUid == request.auth.uid || isGM(cid);
  }
  match /rolls/{rollId} {
    allow read: if isMember(cid);
    allow create: if isMember(cid) && request.resource.data.rollerUid == request.auth.uid;
    allow update: if false;                       // roll log is immutable
    allow delete: if isGM(cid);                   // GM moderation only
  }
  match /messages/{msgId} {
    allow read: if isMember(cid);
    allow create: if isMember(cid)
      && request.resource.data.authorUid == request.auth.uid
      && request.resource.data.text.size() <= 500;
    allow update: if false;
    allow delete: if isGM(cid);
  }
}
```

- Rules ship with the app and are treated like code: changes go through PR review (fits the friend-group workflow).
- **Testing:** Firebase Emulator Suite locally. Rules tests via `@firebase/rules-unit-testing` are the goal; a manual pass in the emulator UI is the acceptable minimum when Java setup is painful on Windows (see USER_TODO note on Java).
- **Dev never runs on open rules.** Local development uses the emulator with the same rules file.

### The "secrets" nuance (important, so nobody panics later)
The Firebase **web** API key in `.env` is a public identifier by design, not a secret — Google's own docs say so. Real security = the rules above. We still keep config in `.env` (gitignored) + `.env.example` (committed) purely for hygiene and per-machine config. **Never** put a *service-account key* (a different thing, from Project Settings → Service Accounts) in the web app — those are actual secrets and we don't need one.

---

## 6. Real-time strategy

- One `onSnapshot` listener per screen section (characters, rolls, messages), created in feature hooks, **torn down on unmount**. Writes use `addDoc`/`updateDoc`/`runTransaction` through the service layer.
- Rolls and messages are **append-only** — no edit conflicts by construction. The roll log is the app's hottest realtime surface and it's the simplest possible shape.
- Character sheets: one owner edits in v1 (GM edits are rare and announced by nature). Strategy = **last-write-wins** with `updatedBy`/`updatedAt` shown in the UI. Document-level locking is a v2 problem, deliberately not solved now (risk §9.3).
- Offline: Firestore SDK's local cache handles flaky table Wi-Fi for free.

**Free-tier math sanity check:** a 5-person, 2-hour session ≈ a few thousand reads (initial loads + one read per changed doc per listener) and a few hundred writes (rolls + messages). Comfortably inside 50K reads / 20K writes per day. Roll and chat logs load the newest 100 and page for more, so logs can't silently eat the daily quota.

---

## 7. Milestones (each: build → stop → George tests → commit locally → DEVLOG entry)

| # | Goal | Acceptance criteria (George checks by running the app) |
|---|---|---|
| **M0** | Scaffold & conventions: React+Vite (+TypeScript unless the group votes otherwise on D2), Tailwind (D3 resolved), Firebase lib wired to `.env`, rules file stub, `CONTRIBUTING.md`, `contributions/` scaffold, pinned deps, `.gitignore` with `.env` | `npm run dev` shows a styled placeholder home page; `npm run build` passes; `.env.example` documents every variable; **no features** |
| **M1** | Auth & profile: email/password register+login+logout, anonymous guest, `users/{uid}` profile doc. Built in that order — email/password defines the profile flow, then **Google sign-in is layered into the same milestone** (in Firebase it's ~20 extra lines on identical plumbing, not new architecture) | Register → set display name → stay logged in after refresh; guest works; Google login lands in the same profile flow; logged-out users see only auth screens; profile doc visible in Firebase console |
| **M2** | Campaigns: create (become GM), join via 6-char code, campaign list, roster | In two browsers: A creates, B joins with code, both see the roster; B sees the campaign in their list; wrong code fails cleanly |
| **M3** | Character sheets: create/edit 5e-lite sheet, derived stats computed & tested, GM edits any sheet, others read-only, live sync | A edits HP → B sees it change within ~1s with no refresh; invalid input (level 0, HP letters) rejected; derived modifiers correct (unit tests) |
| **M4** | Dice roller + roll log: parser (`XdY+/-Z`, adv/dis on d20), roll buttons on sheet + free-form box, results append for everyone | A rolls `2d6+3` with advantage → B sees result with breakdown live; parser unit tests pass; log survives refresh |
| **M5** | Table chat: messages pane, live send/receive, GM delete | Two-way live messaging in two browsers; history persists; GM can delete; >500 chars rejected |
| **M6** | Deploy & hardening: Firebase Hosting (free `*.web.app` URL), rules finalized + emulator-tested, mobile polish, PWA manifest + basic service worker, README | Phone loads the hosted URL; two phones roll and chat live; rules tests pass; app installs to a phone home screen; every USER_TODO item closed |

Order rationale: every milestone is independently testable by running the app; the risky/structural parts (rules, realtime, data model) land **before** polish, so redesign never means throwing away finished features.

---

## 8. How the Android app stays easy later (George's headline requirement)

1. **The data model lives in Firestore, not in the web app.** A future Kotlin app talks to the *same* Firestore with the *same* rules through Firebase's first-party Android SDK. There is nothing web-specific to port — no API to re-design.
2. **The web app is mobile-first + PWA from M6.** Android and iOS both use the hosted URL from day one (installable to the home screen). This is the "both can access now" requirement, satisfied at zero extra cost.
3. **Native Android (Kotlin + Jetpack Compose), when it happens, is UI-only work** — same Firebase project, same data model, same rules. `firebase.json`/rules deploy stays identical. Sharing a test build = debug APK via WhatsApp/Drive, or Firebase App Distribution (free) — no Play Store needed.
4. **To keep promise #1 true, we avoid web-only data tricks now:** no HTML blobs in Firestore, the sheet template stays plain data, derived logic stays small enough to mirror in Kotlin when the time comes.
5. Optional shortcut later (decide then, not now): a Capacitor/TWA wrap of the PWA produces an APK quickly if a native shell is wanted before the real Compose app exists.

---

## 9. Risks & mitigations

1. **Firestore free-tier quota.** ~50K reads/day is generous for a friend table (§6 math), but runaway listeners are the classic footgun. *Mitigation:* scoped listeners with teardown, log pagination (newest 100 + load-more), quota visible in the Firebase console usage tab, all data kept tiny.
2. **Rules bugs = exposed or broken data** — the #1 real risk of a rules-first app. *Mitigation:* default-deny sketch reviewed now, emulator tests before every milestone ships, rules changes require PR review like code.
3. **Concurrent sheet edits (LWW).** Two people editing one sheet at once can silently clobber. *Mitigation:* single-owner editing in v1; GM edits rare; `updatedBy/updatedAt` surfaced. Real-time multi-editor locking is a documented v2 problem.
4. **Billing surprise.** Impossible-ish: Spark has no card on file. Worst case = quota pause, never a charge. Phone auth stays off (it's the one common auth method that requires Blaze).
5. **AI-agent drift / outdated Firebase patterns.** Agents love writing deprecated v8-style code. *Mitigation:* pinned versions, CONTRIBUTING.md mandates modular v9+ imports and copying patterns from `src/lib` + existing features, this file fed to every agent session.
6. **Multi-person drift.** Friends + agents converge badly without force. *Mitigation:* CONTRIBUTING.md from day one, service-layer convention, small PRs with required review (George's own plan — keep it).
7. **Windows emulator friction.** The Emulator Suite needs Java (JDK). *Mitigation:* rules tests are "goal, not gate" — a manual emulator-UI pass is the acceptable minimum; JDK install is a documented USER_TODO option, not a hard requirement.
8. **Join-code brute force.** 6 chars from a 32-char unambiguous alphabet ≈ 1B combinations; campaign doc IDs are unguessable regardless; membership writes require an authenticated account. Fine for a private friend group; GM approval on join is a v1.1 option.
9. **Chat abuse/size.** 500-char cap enforced in rules (not just UI); GM can delete; logs paginated.
10. **Version drift between now and M0.** Numbers in §2 were verified today; the pins at M0 are the source of truth. If a major version lands mid-project, we don't chase it.

---

## 10. Decisions made & alternatives rejected (don't relitigate)

| Decision | Rejected | Why |
|---|---|---|
| Firebase | Supabase, custom Node+Socket.io | Locked in earlier planning: first-party Android SDK quality decides it for the web→Android roadmap; Spark is genuinely free and can't bill |
| Firestore | Realtime Database | Better rules granularity, offline cache, modern API — and it's where Google's investment is |
| Web first | Android first | Whole-table access (iOS friends exist); matches the locked plan; PWA covers phones now |
| 5e-lite concrete template | System-agnostic engine | Known v1 trap; template file keeps the door open without the abstraction cost |
| TypeScript (recommended) | Plain JS | Catches exactly the bug class that multi-agent drift creates; decision D2 — veto at approval and the plan works identically |
| Zustand | Redux, Context-everywhere | Tiny boilerplate for listener-fed stores; beginner-readable |
| Vite static hosting | Firebase App Hosting, SSR frameworks | No SEO/server needs; App Hosting requires paid plan; static = simplest deploy |
| Custom dice parser | npm dice library | ~100 lines, pure, unit-testable, zero dependencies, and it's a perfect first friend-contribution PR |
| Append-only roll log | Editable rolls | Kills conflicts by construction and matches how tables actually use logs |

Open items (decisions D1–D4, questions Q1–Q4) live in `USER_TODO.md`.

---

## 11. Contribution workflow (George's friend-group protocol, agreed 2026-09-30)

- Every human contributor keeps **`contributions/<name>.md`** in the repo: one appended entry per change — date, a one-paragraph "what & why", files touched, and anything the AI models need to know next time. If George builds solo, this collapses to a single `contributions/george.md`.
- The entry gets posted to the group chat (WhatsApp or similar) **and** the file travels with the change/PR. The chat copy is for humans; the in-repo file is the durable record.
- **Every AI session (any model, any person) gets fed:** `PLAN.md` + `DEVLOG.md` + `CONTRIBUTING.md` + the tail of the relevant `contributions/*.md`. That is the anti-drift mechanism — same context in, consistent code out.
- Three logs, three jobs: `contributions/*.md` is the **human** log (why I changed things), `DEVLOG.md` is the **agent** log (commands, errors, decisions — append-only), `git log` is ground truth for both.
- `CONTRIBUTING.md` (created at M0) bakes this in, plus the code conventions: modular v9+ Firebase imports only, service-layer pattern, rules changes require review, and the readability mandate from D3 — every non-obvious block gets a *why* comment, every module a short header comment, written so a reviewer can read the diff without the author in the room.
