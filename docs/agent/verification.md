# Verification

Every change ships only after a real browser has rendered it. This discipline has caught
genuine bugs repeatedly, and several of the "failures" it reported were bugs in the test
rather than the page, so read the false-negative section before trusting a red result.

**Last updated:** 2026-10-02

---

## Environment

- Chromium is pre-installed at `/opt/pw-browsers/chromium`. **Never run `playwright install`.**
- `PLAYWRIGHT_BROWSERS_PATH=/opt/pw-browsers` and `PLAYWRIGHT_SKIP_BROWSER_DOWNLOAD=1` are set.
- Install the driver only: `npm install playwright-core` in the scratchpad.
- Launch with `chromium.launch({ executablePath: '/opt/pw-browsers/chromium' })`.

## The scratchpad gets wiped

It has been cleared mid-session at least twice. Assume the harness is gone and be ready to
rebuild it in about two minutes:

```bash
cd "$SCRATCHPAD"
npm install playwright-core --silent --no-audit --no-fund
mkdir -p assets
curl -sS -L -o assets/tailwind.js https://cdn.tailwindcss.com
curl -sS -L -o assets/fa.css https://cdnjs.cloudflare.com/ajax/libs/font-awesome/6.5.1/css/all.min.css
```

### Cheap fallback when the harness cannot be rebuilt

Extract the largest `<script>` block from the page to a `.js` file and run `node --check`
on it. That proves no syntax breakage, nothing more. Say so plainly if that is all that was
run, and do not describe it as verified rendering.

---

## Harness shape

Load the page over `file://` and fulfil every external request from the local mocks, so the
test does not depend on the network:

```js
await page.route('**/*', async route => {
    const u = route.request().url();
    if (u.startsWith('file:')) return route.continue();
    if (u.includes('tailwindcss')) return route.fulfill({ path: DIR + '/assets/tailwind.js', contentType: 'application/javascript' });
    if (u.includes('font-awesome') || u.endsWith('.css')) return route.fulfill({ path: DIR + '/assets/fa.css', contentType: 'text/css' });
    return route.fulfill({ status: 200, body: '' });   // images etc
});
```

Always test at **two viewports**: desktop 1280x900 and iPhone 375x780.

## What to assert

- Expected number of itinerary cards (`[data-from]`)
- Every new string renders, and every replaced string is **gone**
- New content sits on the **correct day**, by reading `innerText` per `[data-from]` card
- Calendar integrity: `.cal-day` cell count, and `[data-date]` click target count
- A calendar click lands on the right card, offset within about 40 px (expect 12)
- No en or em dashes in rendered text: `!/[\u2013\u2014]/.test(body)`
- Zero JS errors: listen on both `pageerror` and `console` with `type() === 'error'`
- On mobile only: no sideways pan after a real `page.mouse.wheel(400, 0)`
- Where relevant: localStorage persistence across a reload

## After merging

Poll the live URL until the change appears, then assert the new text is present **and** the
old text is gone:

```bash
curl -sS "https://yrameshk26.github.io/static-pages/<page>/" | grep -c "<new string>"
curl -sS "https://yrameshk26.github.io/static-pages/<page>/" | grep -c "<removed string>"
```

Do not report a change as live until this passes. A deploy that is merely "building" is not
done, and saying otherwise has already caused a complaint.

---

## False negatives seen so far

These were test bugs, not page bugs. Check against this list before chasing a red result.

| Symptom | Real cause |
|---|---|
| Badge text "not found" | CSS uppercases it, so `innerText` returns "NO SALES TAX". Match case-insensitively |
| Zero calendar cells found | Cells use `data-date`, not `data-iso`. Only trip days carry it |
| Calendar target count "too low" | The fall page tags only 8 elements (4 days + 4 bars) out of 35 cells |
| Still panning under `overflow-x: hidden` | `scrollWidth` and scripted `scrollLeft` both lie. Only a real wheel gesture is valid |
| Element "overflows" the viewport | Boxes clipped by an `overflow-hidden` ancestor, or inflated by a 3D transform, report misleading rects |
| First visible text is wrong | A naive probe landed on the hero image. Scan for the first element that actually has text |
| Expected city name not visible | The visible slice showed the date chip, not the heading. Assert on what is actually in view |

---

## Rendering a poster or PDF

The trip ledger graphic is built from `scratchpad/poster.html` and rendered with Playwright.

**PNG:** measure `.page` height, resize the viewport to it, screenshot at
`deviceScaleFactor: 2`. Latest output is 3000 x 5868.

**PDF:** inject a print stylesheet with `print-color-adjust: exact` plus
`break-inside: avoid` on sections, call `page.emulateMedia({ media: 'print' })`, then
`page.pdf()` with explicit `width`/`height` matching the artwork, `printBackground: true`
and zero margins. Confirm the measured height under print media matches the screen height,
which proves the layout did not shift.

Note that `pdftotext` and `pdftoppm` are **not installed**, so a PDF cannot be rendered back
to an image for checking. Verify instead by counting `/Type /Page`, reading `/MediaBox`, and
confirming `/ToUnicode` CMaps exist so the text is selectable. Subset fonts mean raw
content streams will not contain readable ASCII, which is expected and not a fault.
