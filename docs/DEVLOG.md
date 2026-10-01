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

---

## Phase 1 — revision 2: external AI security review applied (2026-09-30)

**Goal:** apply the docs-only revision George relayed from his external review (Claude, with ChatGPT/Grok Bot/Gemini/Grok findings). His instruction stands: **plan-only, no app code.**

**Verification first (nothing was applied on trust):** all 6 claimed security-rules bugs were re-derived against PLAN §5 independently before touching the docs — every one confirmed.
1. **Join flow was denied by our own rules** — §4 said "query campaigns by joinCode" while §5 only lets members read campaigns. A genuine self-contradiction in the previous plan; none of the first-round AIs caught it. Fix: `joinCodes/{code}` lookup collection (get-only, list denied), join = `getDoc` → server-side `arrayUnion` update; no client read of the campaign first. Cloud Functions (the textbook fix) need Blaze = paid → excluded; GM creates the code doc in the same batch as the campaign, `getAfter()` proves ownership.
2. **Sheet hijack** — `characters` update checked the *incoming* `ownerUid`; any member could rewrite it and take someone's sheet. Fix: update checks the *existing* doc's owner + ownership is immutable (even for the GM).
3. **Loose self-join** — `hasOnly(['memberUids'])` limited *fields*, not *list contents*; a joiner could add/remove others in the same write. Fix: new list must equal `old.concat([me])`.
4. **Roster cap wasn't rules-enforced.** Fix: `memberUids.size() <= 8` in the rule itself.
5. **Campaign create allowed pre-added members.** Fix: `memberUids == [auth.uid]` + name validation.
6. **`isMember()`/`isGM()` undefined.** Fix: helpers defined; cost documented — verified via Firebase pricing docs that rules `get()`/`exists()`/`getAfter()` are **billed as reads even on rejection**, max 10 per single-doc/query request (20 batched). Our cost: ≤1 per operation.

**Also adopted (and from whom):** ChatGPT's DECIDED/PROPOSED/OPEN/RULESET-DEPENDENT status tags + its honesty point that "template change, not rewrite" is optimistic — PLAN §1 now says M3+ get re-shaped after Q1; ChatGPT's contribution-entry template (PLAN §11); Grok Bot's "MVP draft" / "Stack (proposed)" relabels; its GM-approval promotion → **decision D5**; its gmNotes point → `gmNotes` subcollection added to model + GM-only rules, UI is **decision D6**; Claude's guest-orphan catch → M1 now ships an in-app guest upgrade (`linkWithCredential`, same uid keeps characters) — a stronger fix than the review's warning-only suggestion; Claude's RESEARCH.md idea → `docs/RESEARCH.md` created (per-system log template + free/legal candidate list, feeds the ≈March 2027 ruleset vote). Security-review checklist added to §5.

**Not adopted (with reasons):** ChatGPT's "a ruleset defines what a character is" schema-*now* proposal — contradicts the agreed no-system-agnostic-engine decision; its status tags were adopted instead. Gemini's "open PowerShell as Administrator" — wrong, `-Scope CurrentUser` needs no admin (already documented correctly). Grok's summary-only reply — nothing to apply.

**Git:** review said "git push" but the operative paste-ready prompt says "commit locally, do not push", and the standing rule is never push unasked → committed locally. Origin exists (`github.com/Shushino/TTRPG`), branch ahead by 3 after this commit; push on George's word. README fixed (it literally contained `"# TTRPG"` *with quotes*; the line-ending noise in the review was from the zip copy — the working tree was clean).

**Files modified:** `PLAN.md` (§1 relabel + status tags + honesty caveat, §2 relabel, §4 joinCodes/gmNotes + join-flow redesign, §5 full rules rewrite + read-cost note + security-review checklist, §7 M1/M2, §9 risk 2 expanded + risk 8 rewritten + new risk 11, §10 +2 rows, §11 entry template), `USER_TODO.md` (revision header, D5/D6, pitch pack), `RESEARCH.md` (new), `README.md` (heading fixed). This entry.

**Commands run:** `git status`/`git remote -v`/`git log` (verified 2-commits-ahead + origin exists + clean tree), grep for stale terms, web search ×1 (rules `get()` billing).

**Errors:** one Edit anchor mismatch ("nothing but auth screens" vs actual "only auth screens") — fixed by grepping the real text. No others.

**Not tested / not verified:** rules remain a **sketch** — they get proven in the emulator at M1/M2. One thing flagged to watch: the `diff()` + `arrayUnion` interaction in the self-join rule; if the emulator disagrees, the fallback is explicit field-equality checks. No app code exists, so nothing else is testable yet.

**Next (George):** relay updated PLAN §4–§5 to Claude for the re-check → group votes D2/D5/D6 → Firebase steps 2–7 → go-ahead.

---

## Phase 1 — revision 3: second external review round (2026-10-01)

**Goal:** apply round 2 of George's multi-model review (Claude relaying; Grok's find center-stage). Still docs-only, no app code.

**Verification first:**
- **Gemini's two "critical bugs" — rejected as phantom.** Its "corrected production-ready" rules were compared against our §5: line-for-line identical (modulo comments). Bug A misread `getAfter()` (the campaign create rule never calls `isMember()`, and getAfter exists precisely for batch writes); Bug B misread the concat check (duplicates/odd ordering are denied *on purpose*; `arrayUnion` appends, matching `old.concat([me])`). Grok Bot's assessment was correct — recorded as the project's first phantom-bug incident.
- **Grok's Bug A — confirmed real, docs-checked.** Rules are not filters: a `list` query is all-or-nothing and must be provable from its own `where` clauses; the get()-based `isMember(cid)` read rule can't be proven for a collection query, so B's campaign list (`where('memberUids','array-contains',uid)`) would have failed with permission denied even with correct membership. Verified against firebase.google.com rules-query docs before patching. Key distinction preserved: subcollection rules keep get()-based `isMember()` because the parent path is fixed per query (get evaluated once); the campaigns *collection* rule uses `resource.data` (provable, and saves a billable get()).

**Fixes applied to §5:** query-safe campaign read (`signedIn() && request.auth.uid in resource.data.memberUids`); GM update branch locked (gmUid/joinCode immutable, name validated, `memberUids is list` ≤ 8, GM stays in roster; uses `resource.data.gmUid` instead of `isGM()`'s get()); `users` split to get-only/list-denied; `joinCodes` create requires `campaignId is string` (non-empty). New subsections: **Known limits & trust model** (client-supplied display names, client-computed dice, unpinned timestamps + optional `request.time` hardening, joinCode sync app-enforced, GM-set ownerUid on create is a feature, GM branch leaves setting/system/createdAt unlocked, guest-upgrade `credential-already-in-use` error path for M1) and the **8-case M2 emulator test set**; checklist grew item 6 (list-rule provability). §7 M2 acceptance now names the exact list query and requires the test set; §9 risk 2 records both rounds + the phantom-bug incident; §10 gained two decision rows.

**Git:** review again says push, paste-ready prompt again says "commit locally, do not push" — holding the standing rule: committed locally, push on George's explicit word. (README "modified" reports refer to the zip copy; the actual tree is clean and the heading was already fixed in `22911e3`.)

**Files modified:** `PLAN.md` (§5 intro + helpers comment + users + joinCodes + campaigns read/update + known-limits + checklist + test set; §7 M2; §9 risk 2; §10 +2 rows), `USER_TODO.md` (revision header, next-steps), this entry.

**Commands run:** web search ×1 (rules-are-not-filters doc behavior); git status checks.

**Errors:** none.

**Not tested / not verified:** all rules remain a sketch — the 8-case emulator set exists precisely to convert claims into proofs at M2. The subcollection-query claim (fixed-path get() works for list) is test case 8.

**Next (George):** Claude's re-check of the patched blocks → push decision → group votes D2/D5/D6 → Firebase steps 2–7 → go-ahead.
