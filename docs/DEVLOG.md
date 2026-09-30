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
