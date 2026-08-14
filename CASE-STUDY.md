# 2027 ROBOFEST Korea National Qualifier — competition website

**Silas Lim (임수현)** · [@limsooh](https://github.com/limsooh)
Built July–August 2026 at LUXROBO, for 부산광역시교육청 (Busan Metropolitan City Office of Education)

**Live:** https://robofestbusan2026.com
**Code:** https://github.com/luxroboeducation/2027-ROBOFEST-ROK-Qualifying-Website (public)

---

## Summary

A public information and registration site for a government-sponsored national
robotics qualifier — 8 competition categories, ~100 teams, ~400 students.

I built it in two weeks knowing I would leave the company **18 days before
registration opened**, and that nobody remaining would be a developer. That
constraint, not the feature list, drove every technical decision on the project.

| | |
| --- | --- |
| Role | Sole developer — architecture, build, content system, handover |
| Timeline | 2026-07-29 → 2026-08-14 (2 weeks) |
| Stack | Next.js (App Router), TypeScript, Tailwind, Vercel |
| Output | 21 statically generated routes |
| Runtime dependencies | 5 — Next, React, React-DOM, and two Vercel analytics packages |
| Contribution | 147 of 178 commits, pinned at `0ce1a06` |

---

## The constraint

The site had to keep working, unattended, through a registration window I would
not be present for:

```
2026-08-14   I leave the company
2026-09-01   registration opens          ← 18 days later
2026-10-16   registration closes
2026-11-27   competition
```

The people remaining had other full-time jobs, none had built the site, and the
designated main contact was not a developer.

So "good" here did not mean feature-rich. It meant: **nothing may require a
human to notice something.** No component that could expire, throttle, exhaust a
free tier, or need a password rotated. Anything I built that needed supervision
was a liability, because supervision was the one resource guaranteed to be absent.

---

## Decisions

### Rejected a database, twice

The original plan was a native registration form backed by Supabase — accounts,
team registration, student records, consent tracking. I had already scoped the
schema.

I killed it. A database meant credentials to rotate, a free tier to exhaust, a
service to monitor, and personal data on minors to secure — with nobody there to
do any of it. Registration moved to an embedded Google Form: the school already
had the account, the data lands in a spreadsheet the staff already know how to
read, and **no participant personal data ever touches infrastructure we own.**

The trade-off is real and I would defend it: we lost design control over the form
and gained a manual step (the iframe cannot self-size, so its height is a
configured value that needs re-measuring when the form's description changes).
I documented that as the single recurring manual task in the project rather than
pretending it away.

### Made bad edits fail the build, not the site

All content lives in one typed TypeScript file rather than JSON or a CMS. That
looks like a strange choice until you consider who edits it: a non-developer,
through the GitHub web interface, with no one to ask for help.

Because it is typed and compiled, a malformed edit **fails the build** — so
Vercel refuses to deploy and keeps serving the last good version. The failure
mode is "your change didn't appear," which is recoverable and obvious, instead of
"the site is broken during registration week," which is not.

### Wrote the handover as a deliverable, not an afterthought

~4,600 lines of documentation, structured by who is reading and why:

| document | lines | audience |
| --- | --- | --- |
| `docs/RUNBOOK.md` | 1,134 | non-technical operator, browser only, no terminal |
| `docs/RUNBOOK-CLAUDE-CODE.md` | 881 | someone doing structural work with an AI assistant |
| `docs/SESSION-LOG.md` | 2,339 | dated decision record, including reverted attempts (continued by a colleague from 08-04) |
| `CLAUDE.md` | 231 | the single authoritative rules file |

Two things I would call out as deliberate:

**One authority.** Exactly one file holds rules. A successor with nobody to ask
cannot adjudicate between two documents that disagree, so there is only ever one
— and it says so about itself.

**Rejected approaches are recorded, not deleted.** The log includes things that
were built and then thrown away, and why. Without that, the next person
rediscovers the same dead end and spends the same two hours.

The test I actually cared about: before I left, the main contact edited a file
and watched it go live, unaided. Documentation you have not watched someone use
is a guess.

### Measured accessibility instead of eyeballing it

Every header photograph sits under a navy scrim with white text on it. Rather
than adjusting opacity until it looked fine, I computed the worst-case contrast
ratio for each image against WCAG thresholds — 3:1 for headings, 4.5:1 for body
text — and set each overlay from the measurement.

That caught a case where the heading passed comfortably at 5.4:1 while the
description line underneath sat at 4.30:1, just under the requirement. Eyeballing
would have shipped it.

---

## What I would do differently

**No tests.** For a static content site with no branching logic the compiler
catches the realistic failures, and I would make that call again under the same
deadline — but I would add smoke tests over the built HTML if the project ran
longer, checking that key routes render and critical strings are present.

**One 2,800-line config file.** Keeping content in a single file made it
navigable for a non-developer, which was the point. It is still a large file, and
splitting it per-page with a typed index would have kept that benefit while
reducing the chance of an edit landing in the wrong section.

**Attribution.** Commits were authored from a shared departmental account, so the
work is attributed to that account rather than to the individuals who did it. I
did not notice until the end. On a shared machine, check `git config user.email`
on day one — it is invisible until it is permanent.

---

## Notes

Development was assisted by Claude Code. The architecture decisions, the
constraints they follow from, and the trade-offs above are mine, and I can talk
through any of them.

`CONTRIBUTORS.md` in the repository records who did what, since the commit
history alone attributes everything to a shared account.
