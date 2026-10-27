# Worklog

Decisions, history and what is still outstanding. Newest context at the top of each section.

**Last updated:** 2026-10-27

---

## Open tasks

### Needs the owner to decide or supply

1. **Eight winter items still need action.** Listed in `trips.md`, in urgency order on the page. Holyroodhouse may already be in Abi's email. It is not in Ramesh's. Wonderlights on Tue Dec 8 may not be an open night. Enterprise pick-up is 10:00 AM on Dec 2 and AY555 lands at 11:15, so the pick-up time needs a call. The confirmation email flags an account issue. Do not describe that booking as broken, and do not put the confirmation number on the page. The Enterprise desk number is on the page because the owner asked for the call to be written down. It is their published reservations line. Leave it.
2. **Print-friendly ledger.** The PDF is a single tall dark page. A light A4 version
   paginated over three or four pages was offered and not yet requested.

### Agent can act once unblocked

3. **Winter page logistics additions.** Five concrete details are ready to ship, listed at the
   end of the winter section in `trips.md`. The owner has been told they are ready. This is
   the most obviously useful next change.

### Loose ends on the private record, not blocking any page work

4. Whether the cancelled Helsinki to Toronto return was actually refunded in points and cash.
   No confirmation exists in either inbox. Needs the Atmos activity log and an Amex statement.
5. The Avios cost of the two Finnair Lapland legs. Zero cash is certain, the Avios figure was
   never recorded anywhere. Finnair Plus shows it under the booking.

---

## Shipped so far

Fourteen changes, all via PR and squash merge into `main`. Most recent first.

| PR | Change |
|---|---|
| next | Winter bookings: Castle of Light out, Botanics in, prices and urgency on the to-book list, Holyroodhouse marked confirm, BA92 seats, Enterprise pick-up call |
| #49 | London days: Sunday evening and dinner in Harrow, Winter Wonderland moved to Tuesday, Thursday is a friend in Coulsdon, suitcase tip removed |
| #48 | Fall Day 3 rebuilt: Settlers Green out, Artist's Bluff, named Kancamagus stops, Cathedral Ledge and Diana's Baths in, plus a hotel to hotel Google Maps route link |
| #47 | Agent context docs (this folder) |
| #46 | Tax-free outlet shopping added to the New Hampshire days |
| #45 | Dublin day rebuilt around Malahide rather than the city centre |
| #44 | Return home via Dublin, with a night there on Dec 8 |
| #43 | Leave a city bag at the Hilton over the Lapland leg |
| #42, #41 | Pages deploy retries, see the deploy jam below |
| #40 | Last night moves from Hotel St. George to the Hilton Helsinki Airport |
| #39 | Calendar days jump to the matching itinerary card |
| #38 | Attractions table, every stop with its address and ticket status |
| #37 | Winter Wonderland to arrival night, Tussauds and tea replace the museum day |
| #36 | Map dropped from the fall foliage page too |
| #35 | Intro gate and map dropped, itinerary runs full width |

Earlier work added the month calendar to both trip pages and rebuilt the winter flights
after the transatlantic moved a day earlier.

---

## Lessons that cost real time

### The GitHub Pages deploy lock jam

The worst incident in the project. Run #41's deploy failed after exactly 10 minutes on the
Pages deployment timeout. The pending deployment for the previous commit then held the
`github-pages` environment lock, and the job log said so verbatim:

```
Deployment request failed for 58d61bf... due to in progress deployment.
Please cancel 5c386e6... first or wait for it to complete.
```

What did not work: cancelling the workflow run returned 409 "Cannot cancel a workflow re-run
that has not yet queued" on four separate attempts, and API re-runs landed in limbo with
zero jobs allocated. A fresh push produced a running build but its deploy was rejected in
one second while the lock held. The lock eventually expired on GitHub's side and a fresh
empty commit deployed in 25 seconds.

**Takeaways.** Identify a jammed environment lock early rather than assuming a slow deploy.
Read the job log, it states the blocking commit explicitly. Do not tell the owner a change
is live until the live URL proves it. Saying "it is building" when it was in fact stuck drew
a justified complaint.

### A bash retry script that could never have worked

A script was written to retry pushes until the change went live. It could not work, because
Pages builds from `main` and reaching `main` needs a PR merge, which bash cannot do here.
It was deleted rather than left lying around.

### Test bugs that looked like page bugs

Several verification failures turned out to be faults in the test, not the page. The full
list is in `verification.md`. The general lesson: when a check fails, confirm the assertion
itself is sound before changing the page.

### A stale blog answered a question wrongly

On Oct 4 the owner was told Artist's Bluff had a half mile shortcut. A later search found the
state closed that short trail and made the loop one way (Oct 2025). The owner then asked for
the bluff to be built into the day, so the error would have shipped. Caught only because the
follow-up search read the newest source instead of the first few. **When a question involves a
trail, park or attraction, look for a dated 2025 or 2026 source before repeating a blog.**
Page copy for Artist's Bluff therefore carries an "as of last autumn, read the sign" caveat.

### A test that passed because it found nothing

While checking the London edit, the first run reported "Winter Wonderland is no longer on Sunday" as a
pass and in the same run failed every ordering check. Both had one cause: the day headings are
uppercased by CSS, so the section splitter found nothing and returned an empty string, and "this text
is absent" is trivially true of an empty string. **Any "X is gone" assertion needs a companion assertion
that the section being searched exists and is non empty.** The harness now asserts section length first.

### The scratchpad gets wiped

Cleared mid-session at least twice, taking `node_modules`, the mocked CDN assets and every
verify script with it. Rebuild instructions are in `verification.md`. Budget two minutes.

---

## Working relationship notes

- The owner sends booking screenshots and expects the itinerary updated to match. Read the
  dates carefully, several bookings were cancelled and rebooked and the superseded
  confirmations still circulate.
- The owner asks short questions and wants a direct answer first, then the supporting detail.
- Do not claim something is done until it is verified. This has been raised explicitly.
- When a booking detail looks wrong, say so plainly rather than quietly working around it.
  Flagging the apparent gap on the night of Dec 1 was correct even though it turned out to
  have been rebooked.
- Research rather than recall for opening hours, age policies and whether a venue still
  exists. See the factual discipline section in `trips.md` for four cases where this mattered.

---

## Tooling notes

- **Gmail connector** is signed in as the owner's own account only. A second family account
  holds many of the hotel confirmations and cannot be reached. Those arrive by forward or
  screenshot. The connector has also lacked read scope at least once, failing every call
  including `list_labels`, which is a scope problem and not a bad query.
- **MCP servers drop and reconnect** mid-session fairly often. Re-run ToolSearch rather than
  concluding a capability is gone.
- **`pdftotext` and `pdftoppm` are not installed.** PDFs cannot be rendered back to images
  for checking. Verify structurally instead, see `verification.md`.
