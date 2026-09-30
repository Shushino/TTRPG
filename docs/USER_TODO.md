# USER_TODO.md — what George must do, decide, or answer

Every item says **what** it is, **why** it's needed, and **which milestone it blocks**. Nothing here costs money.

---

## A. Questions I need answered (the game brief) — answer these at approval time

I had to assume the game concept because the brief was blank when I started. Correcting any of these now is cheap; correcting them at M3 costs rework.

- **Q1 — Ruleset.** I assumed **D&D 5e-lite** (six abilities, HP/AC, proficiency, skills, attacks, simple spell list, advantage/disadvantage on d20 rolls). Does your table actually play 5e (or something close)? If you play Pathfinder, a homebrew system, or something else, say so **now** — it changes the character sheet template (M3) and dice rules (M4). *Blocks: M3, M4.*
- **Q2 — Session shape.** I assumed: GM creates a campaign → shares a 6-character join code → players join and keep their own sheets. Does that match how your table would actually use it? *Blocks: M2.*
- **Q3 — Party size cap.** I assumed **max 8 members** per campaign. Fine? *Blocks: M2 (a constant in the join rules).*
- **Q4 — Must-haves.** Is there anything your table would need in v1 that isn't in the 5 MVP features (accounts, campaigns, sheets, dice+log, chat)? If no, the non-goals list in PLAN.md §1 stands. *Blocks: milestone ordering.*

## B. Decisions (D) — my recommendation is marked; veto at approval if you disagree

- **D1 — App name.** Working title **"Hearth Table"** (placeholder). Decide any time before M6 — it's branding, not code. *Blocks: M6 only.*
- **D2 — TypeScript vs JavaScript.** I recommend **TypeScript**. Why: it catches exactly the class of bugs that multiple AI agents + casual contributors drifting in different directions will create, Vite's template makes it near-invisible to use, and it's the industry standard you'll meet in any job. Counterpoint (honest one): you're learning plain JS right now, and if you want the codebase to read like what you're learning, plain JS works — every plan doc here works identically. Say which at approval. *Blocks: M0.*
- **D3 — Styling.** I recommend **Tailwind CSS 4** — fast, consistent, hard to make ugly, and the best AI-agent-to-decent-UI ratio. Alternative: plain CSS files (more learning value, slower, more visual drift between contributors). *Blocks: M0.*
- **D4 — Google sign-in** in addition to email/password + guest? I recommend **deferring to v1.1** — keeps M1 small. *Blocks: M1 scope only.*

## C. One-time setup (click-by-click, ~30 min total, all free)

Do steps 1–7 before milestone M1 (step 1 before M0). I can't create these accounts for you — Google requires it to be you.

1. **Install Node.js (LTS)** — before M0, ~10 min.
   Go to https://nodejs.org → download the **LTS** Windows installer → run it with defaults → open a new terminal and confirm `node -v` prints something ≥ 20. Why: Vite's dev server and build tool run on Node. *Blocks: M0.*
2. **Sign in to Firebase** — ~5 min.
   Go to https://console.firebase.google.com → sign in with your Google account. No credit card anywhere in this flow. Why: everything backend lives here. *Blocks: M1.*
3. **Create the Firebase project** — ~5 min.
   Console → **"Create a project"** (or "Add project") → name it, e.g. `hearth-table` (the project ID becomes part of your free URL, e.g. `hearth-table.web.app`, same trick as your Netlify subdomain — pick a name you'd be happy to share) → **disable Google Analytics** when asked (one less thing) → Create. *Blocks: M1.*
4. **Register the web app** — ~5 min.
   Project → gear icon (**Project settings**) → **Your apps** → the **Web** `</>` icon → nickname `web` → Register → you'll see a `firebaseConfig` block with 6 values. **Copy those values somewhere safe** (I'll need them in step 7). Do **not** download or generate any "service account" key — that's a different, server-only thing we never need. *Blocks: M1.*
5. **Enable sign-in methods** — ~5 min.
   Project → **Build → Authentication → Get started** → **Sign-in method** tab → enable **Email/Password** and **Anonymous**. Leave everything else off (phone sign-in costs money — it stays off). *Blocks: M1.*
6. **Create the Firestore database** — ~5 min.
   Project → **Build → Firestore Database → Create database** → pick a location near you (any works) → **Start in production mode** (rules start locked — good; we ship real rules from M1). *Blocks: M1.*
7. **Fill in `.env`** — ~5 min, at M0.
   I'll create `.env.example` with exactly these names at M0; you copy the matching values from step 4 into a `.env` file:
   `VITE_FIREBASE_API_KEY`, `VITE_FIREBASE_AUTH_DOMAIN`, `VITE_FIREBASE_PROJECT_ID`, `VITE_FIREBASE_STORAGE_BUCKET`, `VITE_FIREBASE_MESSAGING_SENDER_ID`, `VITE_FIREBASE_APP_ID`.
   Why `.env`: keeps per-machine config out of git. (The values are public identifiers, not secrets — see PLAN.md §5 — but we keep them out of the repo anyway.) *Blocks: M1.*

8. **(Only at M6)** Install the Firebase CLI for deploy: `npm install -g firebase-tools` → `firebase login`. Nothing to do now. *Blocks: M6.*

## D. Heads-up for later (nothing to do now)

- **Android Studio / Kotlin: not needed for this web app.** Don't start the 60-hour Kotlin course for this project yet — the native Android phase is months away and everything you're learning now (HTML/CSS/JS → React) is the dependency chain that comes first. The plan keeps the Android path open (PLAN.md §8) without any extra work today.
- **Phone testing at M6:** two phones on the hosted URL. That's the whole requirement.
- **Java (JDK):** only needed if we run the Firestore emulator's rules tests locally (PLAN.md §9.7). Optional; I'll tell you exactly what to install if we go that route.

## E. Honest notes (so nothing surprises you)

- **Spark plan can't charge you.** No card on file = worst case is hitting free limits and things pausing until midnight Pacific. The usage math in PLAN.md §6 says a whole table session uses a small fraction of the daily quota.
- **Your self-reported logs vs reality:** everything I claim in `DEVLOG.md` is checkable via `git log` / `git diff` — the devlog explains, git verifies.
- **What happens next:** you answer Q1–Q4 + D1–D4 (even just "all defaults fine") and say **"approved"** — then I start **M0 only**, stop, and you test.
