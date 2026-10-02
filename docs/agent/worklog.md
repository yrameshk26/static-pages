# Worklog

Decisions, history and what is still outstanding. Newest context at the top of each section.

**Last updated:** 2026-10-02

---

## Open tasks

### Needs the owner to decide or supply

1. **Fall page, Kancamagus stops.** Offered and never answered: adding Sabbaday Falls,
   Rocky Gorge, Lower Falls, Kancamagus Pass and Sugar Hill as named items, plus a note that
   there is no gas, food or cell signal across those 34.5 miles, and that the overlook
   pull-offs fill up on an October Sunday.
2. **Fall page, Best Buy Concord.** Offered and never answered: adding it to Saturday's
   optional Tanger Tilton run, since both sit on the same I-93 stretch.
3. **Seven winter attractions still unbooked.** Listed in `trips.md`. December dates sell out.
4. **Print-friendly ledger.** The PDF is a single tall dark page. A light A4 version
   paginated over three or four pages was offered and not yet requested.

### Agent can act once unblocked

5. **Winter page logistics additions.** Five concrete details are ready to ship, listed at the
   end of the winter section in `trips.md`. The owner has been told they are ready. This is
   the most obviously useful next change.

### Loose ends on the private record, not blocking any page work

6. Whether the cancelled Helsinki to Toronto return was actually refunded in points and cash.
   No confirmation exists in either inbox. Needs the Atmos activity log and an Amex statement.
7. The Avios cost of the two Finnair Lapland legs. Zero cash is certain, the Avios figure was
   never recorded anywhere. Finnair Plus shows it under the booking.

---

## Shipped so far

Fourteen changes, all via PR and squash merge into `main`. Most recent first.

| PR | Change |
|---|---|
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
