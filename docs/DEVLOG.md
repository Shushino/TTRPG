# DEVLOG — append-only audit trail

**Rules (from the project brief):** every milestone logs — goal + date, every file created/modified/deleted with a one-line reason, every command run, every error (exact message → root cause → fix), decisions made + alternatives rejected, deviations from `PLAN.md`, how George can verify it himself, known issues and what was NOT tested. Entries are written as things happen, not retroactively. This log is self-reported — the real audit trail is `git log` / `git diff`.

---

## Phase 1 — Research & Planning (2026-09-30)

**Goal:** produce `PLAN.md`, `USER_TODO.md`, `DEVLOG.md`. No app code — per the brief, Phase 2 does not start until George says "approved".

**Files created:**
- `docs/PLAN.md` — architecture, Firestore data model, security-rules sketch, real-time strategy, milestones M0–M6 with acceptance criteria, Android-readiness plan, risks, decision log.
- `docs/USER_TODO.md` — game-brief questions Q1–Q4 (my assumptions), decisions D1–D4, click-by-click Firebase/Node setup, honest notes.
- `docs/DEVLOG.md` — this file.

**Commands run:**
- `ls -la`, `git ls-files`, `git log --oneline` — repo inspection (found: empty repo, one commit, 12-byte `README.md`).
- Web searches ×6 (versions + free-tier facts, sources below).
- `git add docs/ && git commit` — commit these docs locally (no push, per brief).

**Research findings baked into PLAN.md:**
- Current stable versions verified 2026-09-30: React 19.3, Vite 8.3.x, Firebase JS SDK 12.19.0 (modular v9+ API), Tailwind CSS 4.3.x. Exact pins happen at M0.
- Firestore Spark (free) tier confirmed: 50,000 reads/day, 20,000 writes/day, 20,000 deletes/day, 1 GiB storage, no credit card required. Phone auth requires paid Blaze → excluded from the plan.
- Sources: [Firebase JS SDK release notes](https://firebase.google.com/support/release-notes/js), [React 19.3 blog](https://react.dev/blog/2026/09/09/react-19-3), [Vite](https://vite.dev), [Firebase pricing](https://firebase.google.com/pricing).

**Decisions made (full table in PLAN.md §10):**
- D&D 5e-lite concrete sheet template, fields in one template file — system-agnostic engine rejected (agreed v1 trap). ⚠️ assumption, needs Q1.
- TypeScript recommended over JS (decision D2, veto-able).
- Service-layer pattern: components never touch Firestore directly — one reviewable surface per feature.
- Derived stats computed by pure tested functions, never stored; display names denormalized into rolls/messages.
- Last-write-wins for sheet edits (single owner in v1) — locking deferred to v2, documented as risk.
- Append-only roll log and messages (immutable in rules) — kills sync conflicts by construction.
- Mobile-first + PWA at M6 — this is what makes the Android/iOS path free now and the Kotlin app UI-only later.
- Free-tier safety: scoped listeners, paginated logs, Spark can't bill.

**Deviations from PLAN.md:** none (plan is being authored in this entry).

**Errors encountered:** none (no build system yet). One failed web search (rate limit) — retried successfully.

**Not tested / not verified:** everything — nothing has been built or run. The version numbers and free-tier limits are from official docs dated within the last 3 months; they get re-verified at M0 before pinning.

**Blocked on (George):** Q1–Q4 answers, D1–D4 decisions, setup steps 1–7 in `USER_TODO.md`, and the word "approved".

---

## Phase 1 — revision 1: George's review folded in (2026-09-30)

**Goal:** incorporate George's answers to Q1–Q4 and D1–D4, fix two things he flagged, keep it plan-only (no implementation — his explicit instruction).

**Review feedback handled:**
- **`klass` field question** — he asked why. Explanation: `class` is a reserved word in JS/TS and also in Kotlin (the future Android app), so the field needs a safe name. **Renamed `klass` → `charClass`** (clearer than `klass`) with a comment in the data model.
- **Sheet completeness question** — he asked whether the sheet covered the full TTRPG stat family (MP, stamina, conditions, inventory…). Added PLAN.md §4 "What the sheet covers — v1 vs the rest of the stat family": v1 = combat core; v1.1 = conditions/status effects, spell slots (5e has no MP — slot-based), inventory + currency, death saves, inspiration, rest tracking, XP. Exact vocabulary is a ruleset decision, absorbed by the template-file design.

**Answers folded in:**
- Q1: ruleset stays OPEN by choice — ~6 months of research with coursemates (target ≈ March 2027). 5e-lite demoted to explicit placeholder. Documented that M0–M2 are ruleset-independent; decision gates M3.
- Q2: confirmed (GM creates campaign + ruleset + setting, shares with players) → added `setting` field to campaign model.
- Q3: cap confirmed at 8 (his 6–8 instinct matched my default).
- Q4: open pending research — v1.1 roadmap added so the group sees what's deferred.
- D1: "Hearth Table" stands as default. D2: TS-vs-JS delegated to group + other AI models → paste-ready brief added to USER_TODO §B; decision gates M0. D3: resolved under his readability mandate → Tailwind 4 + named component classes + comment convention (goes into CONTRIBUTING.md at M0). D4: resolved → email/password first, Google sign-in layered into the same M1 (his ordering instinct was correct; ~20 extra lines in Firebase).
- His contribution protocol formalized as PLAN.md §11: `contributions/<name>.md` per person (solo → `contributions/george.md`), entries posted to WhatsApp group + travel with changes, fed to every AI session together with PLAN/DEVLOG/CONTRIBUTING. M0 gains the `contributions/` scaffold.

**Environment findings (from his PowerShell test):**
- Node v24.16.0 already installed → USER_TODO setup step 1 is DONE.
- `npm -v` failed in PowerShell (`npm.ps1 ... running scripts is disabled`) — Windows execution policy, not a broken install. Fix documented: `Set-ExecutionPolicy -ExecutionPolicy RemoteSigned -Scope CurrentUser` (per-user, no admin), or use `npm.cmd` / Git Bash.
- Docker commands in his terminal were from an unrelated tutorial; Docker is not needed for this project — documented as "don't install".

**Files modified:** `docs/PLAN.md` (status header, §1 ruleset/session bullets, §4 data model: `setting` field + `charClass` rename + stat-family subsection, §7 M0/M1 rows, §10 open-items status, new §11), `docs/USER_TODO.md` (statuses, environment fixes, pitch pack §D, D2 brief). `docs/DEVLOG.md` (this entry).

**Commands run:** `grep -n` on PLAN.md (one Edit anchor mismatch — the M1 row said "see only auth screens"; corrected and retried successfully).

**Errors encountered:** none in code (nothing built). One failed Edit due to my imprecise quote of the milestone table — fixed by grepping the actual text.

**Not tested / not verified:** still nothing built or run — plan-only by design. Node version claim is from George's own terminal output (v24.16.0).

**Blocked on (George):** coursemate/group review → then D2 verdict + Firebase steps 2–7 → then the go-ahead. Phase 2 still does not start until he says so.
