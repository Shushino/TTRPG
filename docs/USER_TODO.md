# USER_TODO.md — what George must do, decide, or answer

Every item says **what** it is, **why** it's needed, and **which milestone it blocks**. Nothing here costs money.

> **Status after George's 2026-09-30 review:** Q2/Q3 confirmed, D1/D3/D4 resolved, Q1/Q4/D2 deliberately open pending his research and the group review. Firebase setup (steps 2–7) waits for friend feedback — that's fine, nothing else is time-sensitive.
>
> **Revision 2 (same day, external AI review applied):** 6 security-rules bugs fixed in PLAN §5 (join flow redesigned via `joinCodes`, sheet-hijack hole closed, cap rules-enforced), decisions **D5/D6 added**, `docs/RESEARCH.md` created, "locked" labels softened to draft/proposed. Send the updated PLAN.md back to Claude for the rules re-check.
>
> **Revision 3 (2026-10-01, 2nd external review round):** Gemini's two "critical bugs" verified **phantom** (its "corrected" rules were line-for-line identical to ours — diffed, rejected). Grok's real bug fixed: the campaign-*list* query would have been denied by the get()-based read rule ("rules are not filters" — Firebase docs confirmed). Also: GM update locked, `users` get-only, `joinCodes` type check, a **known-limits & trust-model** section and an **8-case emulator test set** added to PLAN §5. Push still pending George's explicit word.
>
> **Revision 4 (2026-10-01, 3rd review round = sign-off):** Claude read the actual files and found **no new security bugs** — all four reviewers concur. New decisions **D7** (does the cap count the GM?) and **D8** (App Check vs anonymous spam) added; emulator test set now 9 cases; JDK moved into setup (§C step 9); `.gitattributes` added to M0. autocrlf is already `true` on this machine and the working tree is clean — the CRLF-noise README exists only in the zip copies reviewers examined.
>
> **Revision 5 (2026-10-01, 4th review round — wording corrected, one more hole closed):** Codex joined the reviewer pool and was right on both counts. (1) The round-3 agreement is a **review sign-off, not a security sign-off** — no `firestore.rules` file exists yet and nothing has executed; the rules stay **UNVERIFIED** until the emulator set passes at M2. (2) Real edge found: the character **delete** rule didn't require membership — a removed player could still delete their old sheet. Fixed + test case 10; the set is now **10 cases** and a hard **GATE** for M2 (the old "goal, not gate" line in PLAN §9 risk 7 is gone — that was a contradiction with M2 acceptance). D7's constant stays **8 until the group votes** (Gemini's summary said "set to 9" — it isn't; 9 is only the recommendation).

---

## A. Questions (game brief) — status after George's review

- **Q1 — Ruleset: OPEN, on purpose.** George hasn't researched TTRPGs yet and will, with coursemates, over ~6 months (**decision target ≈ March 2027**). **5e-lite stays as the working placeholder** — it shapes the example sheet but nothing architectural. Build impact: M0–M2 (scaffold, auth, campaigns) don't care about the ruleset; M3+ read the sheet template, so the decision must land before M3. *Blocks: M3, M4.*
- **Q2 — Session shape: CONFIRMED.** GM creates the campaign, picks ruleset + setting, shares it; players join, fill their sheets, get into character. The campaign doc now carries a `setting` field for the GM's pitch. *Blocks: nothing — reflected in M2.*
- **Q3 — Party size cap: CONFIRMED at 8.** (George imagines 6–8; my default was 8 — they agree.) One constant in the join logic, trivially changeable later. *Blocks: nothing.*
- **Q4 — Must-haves: OPEN** pending research + coursemate input; the non-goals list in PLAN.md §1 stands until then. The v1.1 sheet roadmap (conditions, spell slots, inventory/currency, death saves, rests) is now in PLAN.md §4 so the group can see what's deferred, not forgotten. *Blocks: milestone ordering.*

## B. Decisions (D) — status

- **D1 — Name: RESOLVED by default.** Working title **"Hearth Table"** stands until the group picks something. *Blocks: M6 only.*
- **D2 — TypeScript vs JavaScript: OPEN — group + other AI models decide.** The plan works identically either way; the decision is expensive to reverse after ~M2, so it must land before M0. Paste-ready brief for other models/friends:

  > **Brief (D2 — TypeScript vs JavaScript):** Project: React 19 + Vite web app, Firebase Auth + Firestore backend, multiple contributors including AI agents; contributors are beginners currently learning JS. **Option A — TypeScript:** catches cross-contributor/agent drift bugs at compile time (a typo'd field name fails the build instead of silently breaking at a friend's table); industry standard for team React apps; cost: type syntax in every file, slightly steeper start. **Option B — plain JavaScript:** zero extra syntax and matches what George is learning right now; cost: no compile-time safety net — the exact bug class a multi-contributor, multi-agent project drifts into. Recommendation from ZCode's planning pass: **TypeScript**, with strict-but-minimal config and a documented comment convention. Reversibility: trivial before M0, painful after M2.

- **D3 — Styling: RESOLVED — Tailwind CSS 4, under George's readability mandate.** He set the rule: "use the better one, as long as the code is human-readable with comments." Tailwind wins for this project (agent-generated UI stays consistent, no naming drift between contributors), and the readability concern is handled by convention, codified in `CONTRIBUTING.md` at M0: reused styles become **named component classes with comments** (Tailwind's `@apply`), every component gets a header comment, and reviewers should be able to read a diff cold. If the group later votes plain CSS, the swap is cheap before M0 only. *Blocks: M0.*
- **D4 — Login methods: RESOLVED — both, in M1.** Email/password is built first (it defines the profile flow — George's instinct was right), then **Google sign-in is layered into the same milestone**: in Firebase it's ~20 extra lines on identical plumbing, not a new architecture. Guests also get an **in-app upgrade path** (`linkWithCredential`) — same uid, keeps their characters — because anonymous accounts die with browser storage (external review catch). *Blocks: M1 scope, now set.*
- **D5 — GM approval on join: OPEN — group vote.** Promoted from "v1.1 maybe" by external review: today anyone signed in — *including anonymous guests* — who has the join code can join. For a trusted friend group my recommendation is **code-only for v1** (codes are unguessable and can't be enumerated, the cap is enforced by security rules, and the GM can remove members), adding an approval flow only when campaigns get shared beyond the circle. The vote changes M2's shape. *Blocks: M2.*
- **D6 — GM-private notes: OPEN — group wish-list.** The `gmNotes` subcollection and its GM-only rules are already in the plan (data + security are nearly free now); the question is whether the group wants a GM-notes UI in v1 or pen-and-paper until v1.1. *Blocks: nothing.*
- **D7 — Does the party cap include the GM? OPEN — group vote (3rd review catch).** Today the rules cap `memberUids` at **8 total including the GM** → 7 players + GM. George's "6–8 for one leader to manage" probably meant 6–8 *players* (→ cap 9). My recommendation: **9** (GM + 8 players), since the mental-load argument was about players. One constant + one test-case number change. **Until the vote, the shipped constant stays 8** — nothing changes in the rules beforehand. *Blocks: M2 (the cap constant and its test).*
- **D8 — App Check before deploy: OPEN — recommend YES (3rd review catch).** Once the app is hosted publicly, anyone with the URL can mint anonymous guest accounts and burn the 20k/day write quota — an availability problem, not a data leak. Firebase **App Check** (free tier) blocks non-app traffic — and must be in **enforce** mode to do it (4th review: registering alone doesn't block anything). Cheap at M6, annoying to retrofit after abuse. *Blocks: M6.*

## C. One-time setup (click-by-click, all free)

### ✅ Step 1 — Node.js: ALREADY DONE (verified 2026-09-30)
`node -v` printed **v24.16.0** — well above the ≥20 requirement. Nothing to install.

Two findings from George's terminal test:
- **`npm -v` failed in PowerShell** with `npm.ps1 cannot be loaded because running scripts is disabled`. That's Windows' default PowerShell execution policy, not a broken install. Fix (once, per-user, no admin): open PowerShell and run
  `Set-ExecutionPolicy -ExecutionPolicy RemoteSigned -Scope CurrentUser`
  then `npm -v` works. Zero-change alternative: use `npm.cmd -v`, or run npm from Git Bash (which already works).
- **Docker is NOT needed for this project.** The `docker pull node…` commands George pasted are from a different tutorial (running Node inside containers). This project runs Node directly — skip Docker entirely, don't install it.

### Steps 2–7 — Firebase (do after the group review, ~30 min total)
1. ~~Install Node.js~~ ✅ done (above).
2. **Sign in to Firebase** — https://console.firebase.google.com, Google account, no card anywhere. *Blocks: M1.*
3. **Create the project** — "Create a project" → name, e.g. `hearth-table` (the project ID becomes the free URL, `hearth-table.web.app` — same trick as the Netlify subdomain) → **disable Google Analytics** → Create. *Blocks: M1.*
4. **Register the web app** — Project settings (gear) → Your apps → Web `</>` → nickname `web` → register → **copy the 6 `firebaseConfig` values**. Do NOT generate any "service account" key (server-only thing we never need). *Blocks: M1.*
5. **Enable sign-in methods** — Build → Authentication → Get started → Sign-in method → enable **Email/Password**, **Google**, and **Anonymous**. Everything else stays off (phone sign-in costs money). *Blocks: M1.*
6. **Create Firestore** — Build → Firestore Database → Create → location near you → **Start in production mode** (locked rules; we ship real rules from M1). *Blocks: M1.*
7. **Fill in `.env`** — at M0 I'll create `.env.example` with exactly: `VITE_FIREBASE_API_KEY`, `VITE_FIREBASE_AUTH_DOMAIN`, `VITE_FIREBASE_PROJECT_ID`, `VITE_FIREBASE_STORAGE_BUCKET`, `VITE_FIREBASE_MESSAGING_SENDER_ID`, `VITE_FIREBASE_APP_ID`; you paste the step-4 values. (They're public identifiers, not secrets — see PLAN.md §5 — kept out of git anyway.) *Blocks: M1.*

8. **(Only at M6)** Firebase CLI for deploy: `npm install -g firebase-tools` → `firebase login`. *Blocks: M6.*
9. **(Optional, recommended before M2)** Install a JDK (17+, e.g. Eclipse Temurin from https://adoptium.net) — the Firebase Emulator Suite needs Java, and the M2 milestone is gated on running the 10-case rules test set locally. Installing it now, with the rest of setup, avoids discovering the dependency mid-milestone (3rd review's point). *Blocks: M2 test set.*

## D. Pitch pack — what to send friends / other AI models

The review set is: **`PLAN.md`** (the whole architecture), **`DEVLOG.md`** (what happened so far), **`RESEARCH.md`** (grows as systems get played during the 6-month research phase), and for other AI models additionally the **D2 brief** in §B above. `USER_TODO.md` (this file) shows them the open decisions. When collaborators join: each keeps `contributions/<name>.md` (protocol + entry template in PLAN.md §11); solo builds collapse it to `contributions/george.md`.

## E. Heads-up for later (nothing to do now)

- **Android Studio / Kotlin: not needed for this web app.** The native Android phase is months away; the web learning path (HTML/CSS/JS → React) comes first. PLAN.md §8 keeps the Android path open without extra work today.
- **Phone testing at M6:** two phones on the hosted URL. That's the whole requirement.
- **Java (JDK):** needed for the M2 emulator test set — now a proper setup step (§C step 9) so it's installed before M2, not discovered during it.

## F. Honest notes (so nothing surprises you)

- **Spark plan can't charge you.** No card on file = worst case is quotas pausing until midnight Pacific. PLAN.md §6 shows a table session uses a small fraction of the daily quota.
- **Self-reported logs vs reality:** claims in `DEVLOG.md` are checkable via `git log` / `git diff` — the devlog explains, git verifies.
- **What happens next:** the rules have a five-model **review** sign-off (4th round) — *review*, not verification: they stay UNVERIFIED until the emulator set runs at M2. The plan is ready for your coursemates. Push decision remains yours (`git push origin main` — docs-only commits). Then: group votes **D2, D5, D6, D7, D8** → Firebase steps 2–7 (+ JDK step 9) → the go-ahead, and I start **M0 only**, stop, and you test.
