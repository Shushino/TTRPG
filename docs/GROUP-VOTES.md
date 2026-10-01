# GROUP-VOTES.md — the 5 decisions for the group (paste into WhatsApp)

**How to use:** copy everything below the line into the group chat. When votes come back, record them in `docs/USER_TODO.md` §B (D2, D5, D6, D7, D8) — that's the doc of record. Vote text by Claude (2026-10-01), five-model reviewed; recommendations included but no pressure.

---

*Hearth Table (working name) — 5 quick votes before we start building*

We're planning a web app where a Game Master (GM) runs a tabletop RPG online: players join with a code, fill in character sheets, roll dice, and chat live. Nothing is built yet. The full plan is in PLAN.md (George can send it). We just need 5 quick answers. Reply like "D2: A, D5: A, D6: A, D7: A, D8: A" or tell us why you disagree.

*D2 — TypeScript or plain JavaScript?*
A) TypeScript = JavaScript plus labels on data. Catches typos before the app runs. Recommended for team projects. Slightly more to read.
B) Plain JavaScript = simpler to read, but typos only show up when someone's game breaks mid-session.
Suggested: A

*D5 — Should the GM approve each person who joins?*
A) No, anyone with the join code gets in (we share the code only in this chat).
B) Yes, the GM taps "approve" for each joiner. Safer if a code leaks, a bit more work.
Suggested: A for now

*D6 — Private GM notes in the first version?*
A) Later. (The database is already ready for it.)
B) Yes, a "GM only" notes page in v1.
Suggested: A

*D7 — Maximum table size?*
A) 8 people total including the GM (7 players + GM).
B) 9 people total (8 players + GM).
Suggested: B

*D8 — Protect the live site from spam accounts?*
Once the site is public, anyone with the link could create fake accounts and use up our free daily limit. Firebase "App Check" (free) blocks that.
A) Yes, switch it on before we go live.
B) Skip it for now and add it only if someone abuses the site.
Suggested: A

Not voting on yet: which game system we use (we're researching and playing several first, aiming to decide around March 2027).
