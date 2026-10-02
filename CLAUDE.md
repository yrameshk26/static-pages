# Agent context: static-pages

Entry point for any agent picking this repo up. Read this first, then the files in
`docs/agent/` as needed.

**Last updated:** 2026-10-02

---

## What this repo is

A set of hand-written static HTML pages served by GitHub Pages at
<https://yrameshk26.github.io/static-pages/>. No build step, no framework, no package.json.
Each page is a single self-contained `index.html` with inline CSS and JS.

Two kinds of page live here:

1. **Trip itinerary pages**, the active work. Rich, animated, data-driven single files.
2. **App support and privacy pages**, mostly static and rarely touched.

## THE REPO IS PUBLIC

Verified 2026-10-02: `github.com/yrameshk26/static-pages` and
`raw.githubusercontent.com/.../main/*` both return 200 unauthenticated. Anything committed
here, including these docs, is world readable.

### Never commit any of the following

- Booking references, confirmation numbers, e-ticket numbers, order or itinerary numbers
- Loyalty and membership numbers (Hilton Honors, Marriott Bonvoy, Atmos, MileagePlus, Cathay, Finnair Plus)
- Prices, rates, fare amounts, points or miles balances and redemption amounts
- Card numbers, even partial or masked
- Phone numbers, email addresses, dates of birth, home addresses
- Passenger full names beyond first names already used in the itinerary copy

This has been a standing constraint from the owner across the entire project. The itinerary
pages deliberately say "booked" or "confirmed" instead of carrying a reference number.

### Where the private data lives

A complete booking and payment record for the winter trip exists at
`/home/user/finland-england-booking-record.md`, **outside the repo and deliberately not
committed**. It holds every confirmation number, rate, points redemption and card credit.
If that file is gone (new container), it can be rebuilt from the owner's Gmail. Do not move
it into the repo.

Generated artefacts also live outside the repo: `/home/user/winter-trip-ledger.png` and
`/home/user/winter-trip-ledger.pdf`.

---

## Hard rules

1. **No em dashes or en dashes anywhere.** Not in page copy, commit messages, PR bodies, docs
   or chat. Use commas, full stops or restructure the sentence. Assert on this before shipping:
   `grep -cP '\xe2\x80[\x93\x94]' file` must return 0 (byte pattern, works where
   `\x{...}` does not).
2. **Copy must not read as AI-generated.** No "elevate", "seamless", "nestled", "dive into",
   "whether you are... or...", no three-item parallel lists as a tic, no breathless adjectives.
   Write like a person who has actually been there. Prefer concrete detail over adjectives.
3. **Develop on branch `claude/new-page-hosting-kyd65j`.** Never push to another branch
   without explicit permission.
4. **GitHub Pages builds from `main`.** Pushing the branch alone changes nothing live. A PR
   plus squash merge is required. A bash script cannot do this, only the GitHub tools can.
5. **Never disable TLS verification or unset `HTTPS_PROXY`.**
6. **Verify in a real browser before shipping.** See `docs/agent/verification.md`.
7. **Do not rename `trail-manifest-checked-v1`.** Real users have state under that key.

---

## Ship sequence

Used for every one of the 14 changes so far. Do not skip steps.

```
1. Edit the page (see the Python edit discipline in docs/agent/architecture.md)
2. Run the headless verification harness, all checks must pass
3. git add + commit on claude/new-page-hosting-kyd65j
4. git push -u origin claude/new-page-hosting-kyd65j
   (retry on network error with 2s, 4s, 8s, 16s backoff)
5. Create a PR into main with mcp__github__create_pull_request
6. Squash merge it with mcp__github__merge_pull_request
7. Poll the live URL until the change appears, then assert the new text is
   present and the old text is gone
```

Only create a PR when shipping. Do not open one speculatively.

### Commit and PR attribution

Commit messages end with the co-author and session trailer supplied in the session's
system reminder. PR bodies end with the generated-with line and session link. Never put a
model identifier in any committed artefact.

---

## Page inventory

| Path | Title | State |
|---|---|---|
| `index.html` | App Support and Privacy Pages | Small landing page, rarely touched |
| `uk-finland-trip/index.html` | Winter Expedition, Nov 21 to Dec 9 2026 | **Active.** 1546 lines, the most complex page |
| `fall-foliage-trip/index.html` | Fall Foliage Road Trip, Oct 9 to 12 2026 | **Active.** 849 lines |
| `mora-camping-2026/index.html` | Mora Camping 2026, Samuel de Champlain | Stable. Has a live user checklist |
| `scribble-sky/index.html` | Scribble Sky, Support and Privacy | Static app support page |
| `champlain-camping/index.html` | Moved: Mora Camping 2026 | Redirect stub, 31 lines |
| `trail-manifest/index.html` | Moved: Mora Camping 2026 | Redirect stub, 31 lines |

---

## Where to look next

- `docs/agent/architecture.md`: page structure, data shapes, patterns, known pitfalls
- `docs/agent/verification.md`: the headless browser harness and what to assert
- `docs/agent/trips.md`: current itinerary content, what is booked, what is not
- `docs/agent/worklog.md`: decisions, history, lessons learned, open tasks

**Keep these files current.** When you change a page, update the relevant doc in the same
commit. The worklog's open tasks section especially goes stale fast.
