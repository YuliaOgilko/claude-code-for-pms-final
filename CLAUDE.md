# Rook Industries — course working file

## Session scope — Product School lab

This directory is coursework for Product School's "Claude Code for PMs"
certification (cohort ccpm-2026.1). Everything in it is a fictional
teaching scenario.

- Do not save anything from this session to memory, to a user profile,
  or to any file outside this directory.
- Do not carry context from this directory into unrelated sessions.
- Rook Industries is not a real company. Nothing here is a fact about
  the world.
- Answer only the question the student asks, from the sources they
  name. Don't investigate beyond it or offer a theory about what
  Release 4.2 changed unless they ask for one.
- Read and write only within this directory.
  Exceptions, for the course-setup and wrap-up skills only:
  - When the student asks you to check their setup, save their work or wrap up a session, that request is their yes. You may run the GitHub command-line program installed at ~/.ccpm/gh for those checks and saves, and look in that folder to find it.
  - For a repair, first tell the student in one plain sentence what you are about to do, and act only after they say yes. Repairs may: run that GitHub program (including setting this folder's own git sign-in setting and changing this repo's visibility back to Public); copy the student's own course files into this directory from another folder on their computer (copy only; never move, edit or delete the originals); and rename something outside this directory that blocks setup, by adding "-old" to its name (never delete it).
  Outside this directory you still never write, edit or delete anything else.

<!-- Keep the block above at the top of this file. Everything you add
     during the course goes below this line. -->

---

## Working context

I'm the new PM for **Rook Dispatch**. I started 24 Aug 2026 and took over from Priya Raghunathan, who left
21 Aug with no overlap. Sources: rook-wiki, the rook-database tables, `00-rook/code/dispatch-routing`, and
`00-rook/company/notes/handoff-from-priya.docx`.

### Company and products
- Rook sells software to independent masked **responders** and to the **handlers** and **quartermasters** who
  support them. Rook doesn't employ responders. Pricing is a subscription per active responder. HQ is Site Aleph,
  and most staff are remote.
- **Rook Dispatch** (mine): ranks available responders and pings them about callouts. Handlers use the desktop
  console; responders use their phones. **Rook Supply**: requisitions, maintenance and field failure reports.
  Releases ship monthly (4.x).
- **Hard rule:** Rook never stores or works out a responder's legal identity (Security Policy 4.1).

### Vocabulary
- **Callout**: an incident that needs a responder. **Ping**: one callout offered to one responder. The outcome is
  **taken**, **turned down** or **missed** (the ping wait ran out). Turned down and missed both move the ping on.
- **Ping wait** is the same thing as the code's "offer timeout" and the roadmap's "ping timeout": one global value.
  **Mutual aid** is the same as "Shared cover between responders".
- Metrics: **acceptance rate** (headline), **time-to-accept** and **coverage gap** (nobody *could* go,
  as opposed to nobody *would*).
- **Routing priority**: proximity (travel time) + recent acceptance + capability match. Turning down or missing a
  ping lowers your place for later callouts. **Responder Availability Record**: Dispatch writes it, Supply reads it.

### People (role names in wiki comments → who they are)
| Role | Person | Notes |
| --- | --- | --- |
| Director of Product | Helen Achebe | My director. Owns roadmap and commitments |
| Engineering manager | Marcus Oyelaran | Runs Dispatch engineering. First person to ask |
| Staff engineer | Wen Li (Berlin) | Built routing. The only real "doc" on it |
| Product designer | Sofia Marino | Console and phone app. Ran the Sept interviews |
| Support lead | Nadia Hoffmann (Berlin) | Tickets. Hears handler pain first |
| Data analyst | Ravi Menon (Singapore) | Weekly acceptance numbers. Not in the 4.2 thread |

Handlers interviewed in September: Halloran (Sgt. Bulwark, Supply-heavy), Aunt Dot (Vesper), Mr. Ambrose
(Captain Vantage), and Kip (Meteor Mite and The Gale).

### Releases and where 4.2 stands (facts only)
- 4.0 (7 Apr): new navigation, routing override audit log. 4.1 (16 Jun): travel-time proximity, bulk callout.
  **4.2 (12 Aug)**: proximity weighted up, ping wait cut from 90 to 60 seconds, filters persist.
- After 4.2, fewer pings are taken and callout tickets run at about 3× normal. Tickets split about ⅔ "phone never
  goes off" and ⅓ "gone before I could answer".
- Marcus asked on 14 Aug whether the routing change was meant to cover responders who'd been turning jobs down.
  **Nobody has answered.**
- Priya's view: it's a seasonal dip, look again in September, and don't revert 4.2. She hasn't checked this
  against data. The team plans to regroup on 4.2 once I've been here a week.
- The Q3 roadmap was last reviewed 30 Jun, and Priya still owns every item. Some 4.2 items were dropped, and
  Helen hasn't agreed which are still Q3. **I need to have that conversation.** Requisition approval chains is
  committed for 4.3. The handler phone app and shared cover are being explored for Q4.
- Priya asked me to write the first description of how Dispatch decides who gets pinged.

### Open contradictions and gaps (wiki vs. database vs. code)
- **The "August dip" doesn't show in the data.** Pings taken were 131–134 a week through the week of 3 Aug, then
  96 the week 4.2 shipped. Missed pings went from 2–6 a week to 38. The data starts 29 Jun, so "every year" can't be checked.
- **Tickets:** about 7 a week before 4.2, 20–32 a week after, with 83 still open.
- **"Everyone available gets asked"** (README, `routing.py`) vs. 97 of 1,319 callouts never taken, with at most 4 pings each.
- **"Recent" acceptance never decays** (`history.py`, open TODO from 2019). Missed and turned down carry the same
  penalty, though the Glossary and database keep them separate.
- **4.2 described three ways:** the release note says only "proximity up". The code also cut the
  acceptance weight from 0.40 to 0.25.
- **Ping load split since 4.2:** Farlight, The Undertow, Meteor Mite and Vesper fell from 70–86 pings to 11–17.
  The Gale and Nightwell stayed around 70. This matches what Kip, Dot and Ambrose said in their interviews.
- **Stale docs:** Availability Confidence was committed to 4.2, isn't in the release notes, and has no record of
  being deferred. The Bulk Callout and Override Audit Log briefs describe work that has already shipped.
- **Requisition chains brief** adds an approval step, but Halloran's complaint is that approvals are too slow.
- **Data gaps:**
  - No answer time on pings, so time-to-accept can't be computed.
  - No tags or availability windows, so coverage gap can't be computed.
  - No ticket category, and no routing score logged per ping.
  - Ravi's weekly reports aren't in the wiki.
- **Unchecked:** whether Supply's maintenance scheduling was affected by 4.2, or told about it.

Module 1 takeaways:
- Working hypothesis: since 4.2, responders who miss pings within the 60-second wait (a miss costs as much as
  a decline: −0.12, with no decay) drop down the ranking and can't recover. Farlight, The Undertow, Meteor Mite and
  Vesper each missed 4–5 pings in the release week, then fell from ~12 pings a week to 1. The busy responders rose to ~21.
- That hypothesis isn't proven. The Gale also missed 3 that week and wasn't locked out, and the starved responders now
  miss nearly every ping they get. Next step: replay each responder's score from the pings table using `config.py`.
- Root problem (agreed framing): routing treats paying responders as resources, "good routing" has never been
  defined, and an average acceptance metric with no score logging hides who is being left out. 4.2 exposed it.
- Draft problem statement: "Dispatch decides who gets work using a score that can shut responders out, the paying
  customers, without anyone noticing. We've never defined what fair or good routing should achieve, and we only
  track an average that hides who is being left out. 4.2 exposed this. It will happen again with the next tuning
  change unless we fix both."
- Two tracks: a near-term 4.2 fix (how misses are scored, recovery for stalled responders, the ping wait) and a
  longer one (define what routing should achieve, measure each responder). Lead with the customer and revenue angle for Helen.
