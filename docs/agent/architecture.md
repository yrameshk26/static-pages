# Architecture and patterns

How the trip pages are built, the data shapes inside them, and the traps that have already
cost time. Read with `CLAUDE.md`.

**Last updated:** 2026-10-02

---

## Shape of a trip page

One `index.html`, no build step. Order inside the file:

1. `<head>`: meta, title, Google Fonts (Plus Jakarta Sans), Tailwind via CDN, FontAwesome CSS
2. A large inline `<style>` block with custom CSS on top of Tailwind utilities
3. Markup: a `flex-none` header in normal flow, then `<div id="feed">` which takes the
   remaining height
4. One large inline `<script>` holding the data arrays and all render functions

### Dependencies, all CDN, no local vendoring

- `https://cdn.tailwindcss.com` (Tailwind play CDN, configured inline)
- `https://cdnjs.cloudflare.com/ajax/libs/font-awesome/6.4.0/css/all.min.css`
- `https://fonts.googleapis.com/css2?family=Plus+Jakarta+Sans:wght@300..800`
- Unsplash images by URL for hero art

**Leaflet and Three.js were both removed** in Oct 2026. No page loads a map or a 3D scene
any more. Do not reintroduce them. The owner asked for maps gone from every itinerary page,
and the fall page's `mapStops` data was checked stop by stop before deletion to confirm no
unique content was lost.

### Layout contract

The header is `flex-none` in normal document flow, not absolutely positioned. `#feed` sizes
itself from the remaining height. Inside `#feed` sits a single
`max-w-5xl mx-auto` column. This is what makes the full width, no-sidebar layout work.

`#feed` carries a mobile-critical CSS block:

```css
#feed {
    -webkit-overflow-scrolling: touch;
    touch-action: pan-y;
    overflow-x: hidden;        /* required, see pitfalls */
    overscroll-behavior-y: contain;
}
```

---

## Data shapes

### `days[]` (both trip pages)

One object per itinerary card.

```js
{
  id: 'day3', day: 'Day 3', date: 'Sunday, Oct 11',
  from: '2026-10-11', to: '2026-10-11',     // ISO, drives calendar to card mapping
  route: 'Gilford to North Conway', drive: '~2.5 hr total scenic drive',
  image: 'https://images.unsplash.com/...',
  color: '#ef4444', colorName: 'crimson',
  activities: [
    { text: '...', badge: 'Stroller friendly stops', badgeType: 'stroller' },
    { text: '...' }                           // badge and badgeType optional
  ],
  hotel: { name: '...', confirmed: true, address: '...', room: '...' }  // or null
}
```

Cards render with `data-from` and `data-to` attributes. That is how the calendar finds them.

`badgeType` keys into a `badgeCls` map. Existing types include `carrier` and `stroller`.
Badge text renders uppercase via CSS, so `innerText` returns it uppercased. Match
case-insensitively when asserting on it.

### `STAY_CALENDAR[]`

Drives the month calendar. One entry per stay.

```js
{ short: 'Towne',                              // short label, must fit a narrow bar at 375px
  name: 'TownePlace Suites Laconia Gilford',
  area: 'Gilford, NH. Free breakfast',
  from: '2026-10-10', to: '2026-10-11',        // checkout date, exclusive
  color: '#84cc16',
  travel: true,                                // optional, styles it as a travel segment
  when: 'Mon Oct 12, no hotel' }               // optional, overrides the computed range line
```

Week start differs per page: the fall page starts weeks on **Sunday**, the winter page on
**Monday**. Do not unify them without being asked.

### `ATTRACTIONS[]` (winter page only)

39 rows grouped by city, each with a `status` of `booked`, `book`, `onday` or `free`.
Rendered twice from one array: a real `<table class="att-table">` inside a
`hidden md:block` wrapper, and stacked `.att-card` blocks inside an `md:hidden` wrapper.
Change the array, both views follow.

Current counts: 7 `booked`, 7 `book`, 7 `onday`, 18 `free`.

### Booking checklist

Seven checkbox items on the winter page under the heading "Activities to book", persisted
to `localStorage`.

---

## localStorage keys

| Page | Key pattern | Notes |
|---|---|---|
| `uk-finland-trip` | `ukfin-book-<id>` | one key per checklist item |
| `fall-foliage-trip` | `nefall-book-<id>` | one key per checklist item |
| `mora-camping-2026` | `trail-manifest-checked-v1` | single JSON blob. **Never rename**, real users have state here |

---

## Calendar to itinerary jump

Clicking a calendar day or stay bar scrolls the feed to the matching card. Only trip days
get `data-date`; non-trip days in the grid are inert. On the fall page that means 35
`.cal-day` cells but only 8 elements carrying `data-date` (4 days plus 4 bars).

Mapping is plain ISO string comparison, which works because the format is zero padded:

```js
const card = [...cards.querySelectorAll('[data-from]')]
    .find(c => iso >= c.dataset.from && iso <= c.dataset.to);
```

### The scroll fix, do not simplify this

`scrollIntoView` overshoots by 100 to 260 px and stays overshot, because `.reveal-3d`
applies `perspective(900px) rotateX(9deg) translateY(46px) scale(.97)` to unrevealed cards
and the browser measures that transformed box. The working approach suppresses the
transition, reveals the card, forces one reflow to get the true layout position, then
scrolls to that:

```js
const t = card.style.transition;
card.style.transition = 'none';
card.classList.add('in');
void card.offsetWidth;                                  // force reflow
const top = feed.scrollTop + card.getBoundingClientRect().top
          - feed.getBoundingClientRect().top - 12;
card.style.transition = t;
feed.scrollTo({ top: Math.max(0, top), behavior: reduceMotion ? 'auto' : 'smooth' });
```

Verified landing offset is a consistent 12 px.

---

## Date handling

All calendar maths uses `Date.UTC` and ISO strings. Never construct dates from local time
or the grid shifts by a day depending on the viewer's timezone.

Stay bars are laid out with CSS grid, `grid-column: {col} / span {span}`, with lane packing
so overlapping stays do not collide.

---

## Edit discipline

Page files are large and a bad regex can silently corrupt them. Use a Python script in the
scratchpad doing token based replacement, with an assert on the match count for every edit:

```python
def sub(old, new, n=1, label=""):
    global s
    c = s.count(old)
    assert c == n, "expected %d of %r, found %d" % (n, (label or old)[:70], c)
    s = s.replace(old, new)
```

A failed match then aborts the whole run instead of writing a half-edited file. Finish with
`assert "\u2014" not in s and "\u2013" not in s` before writing.

---

## Known pitfalls, all hit at least once

1. **`scrollIntoView` overshoots.** Cause and fix above. Same 3D transform is the culprit.
2. **Horizontal pan on mobile.** `.reveal-3d` inflates the bounding box of unrevealed cards
   by about 25 px, letting the feed pan sideways. Fixed by `overflow-x: hidden` on `#feed`.
   Note two tests that give false passes: `scrollWidth` still reports overflow under
   `overflow-x: hidden`, and scripted `scrollLeft` still moves. The only valid test is a
   real gesture, `page.mouse.wheel(400, 0)`, then assert `scrollLeft === 0` on feed, document
   and header.
3. **Stays rendered twice across a month boundary.** A stay spanning the end of a month
   appeared in both the overflow week and the next month. Fix is to clip each segment to the
   month being drawn:
   `const a = Math.max(s.f, wk, moFrom), b = Math.min(s.t, wk + 7 * MS, moTo);`
4. **"1 night" on a travel day with no night.** Use the optional `when` field to override
   the computed range line.
5. **Calendar bar labels truncate at 390 px.** Keep `short` values genuinely short. Existing
   shortenings: Hilton HEL to Hilton, Glass Igloo to Igloo, Lomavekarit to Lomav,
   TownePlace to Towne.
6. **Header wrapped to three lines at 375 px** on the fall page. Fixed with `min-w-0` plus
   `truncate` on the text block and a badge that shortens below `sm`:
   `<span class="hidden sm:inline">Hotels </span>Booked`.
7. **GitHub Pages deploy lock jam.** See `docs/agent/worklog.md`, it cost a lot of time once.
