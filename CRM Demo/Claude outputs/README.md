# OCS CRM — live demo (demo.optimizedcompanysystems.com)

One file: `index.html`. No build step, no backend, no database, no API keys. Everything a prospect does lives in their browser tab and disappears on reload or **Reset**.

---

## 1. Run it locally

From the folder that contains `index.html`:

```bash
python3 -m http.server 8080
```

Then open **http://localhost:8080**. The command keeps running and stays silent until the browser asks for the page; that's normal. Press `Ctrl+C` to stop it.

Double-clicking `index.html` also works.

---

## 2. Deploy

Drop `index.html` (nothing else) into any static host:

| Host | How |
|---|---|
| **Cloudflare Pages** | Pages → Create project → *Direct upload* → drag the folder in → add the custom domain. |
| **Netlify** | *Add new site → Deploy manually* → drag the folder in → add the subdomain. |
| **Vercel** | `vercel deploy --prod` in the folder, or drag and drop it in the dashboard. |

Size: about 750 KB raw, **about 265 KB gzipped**. The two OCS logo animations are embedded in the file.

Measured: first paint is 70–300 ms, including on throttled phones, because the intro animation is plain HTML at the top of the file. The CRM finishes loading behind it.

---

## 3. What's in the demo

- **Intro:** a dark OCS logo reveal on the first visit of each session. Visitors can skip it (button, click, Esc or Enter). It never shows on the client-facing signing and review pages, and visitors who prefer reduced motion see a static logo instead.
- **Business Pulse:** see section 6.
- **The full CRM cycle:** lead → client → estimate → job → invoice → paid. A step-by-step "What the client sees" view covers the email, pay page, PDF, receipt, text messages, contract signing and both branches of the review funnel.
- **Maps:** OpenStreetMap tiles shown through Leaflet and styled dark. **No key is needed.** They appear on client profiles, the jobs map, the route map and the **property measuring tool**.
  - The measuring tool: outline an area on the map to get square feet, acres and perimeter, then "Start an estimate" adds it as a line item.
  - Find it from the dashboard ("Measure a property on the map"), Estimates ("Measure & quote"), any client profile ("Measure"), any job's menu, or the line-item editor ("Measure a property").
- **Book a call with OCS:** the banner across the top opens a panel with name, business, phone, email, state, industry and an optional note. It's styled as OCS's, not as a CRM button. Branded versions also appear at the end of each client view, in the Tier 2 explainers and in the Settings notice.

---

## 4. Book a call → your email and your CRM

In the `CONFIG` block near the bottom of `index.html`:

```js
BOOK_A_CALL_URL: 'https://cal.com/your-link',   // or Calendly
LEAD_ENDPOINTS: [
  'https://formspree.io/f/xxxxxxx',              // → your email
  'https://<your-crm-intake-endpoint>',          // write-only lead intake you control
],
```

On submit, the demo:
1. Opens the calendar with the name and email already filled in.
2. POSTs JSON to every endpoint in the list. If a POST fails, the visitor sees it on screen with a **Send again** button.

**Your CRM:** expose a small endpoint that only *inserts* a lead (rate-limited). The demo never holds your CRM's code or keys.

**Your email:** use Formspree or Web3Forms.

Payload: `name, business, phone, email, state, industry, note, submitted_at, source, opened_from`, plus a `journey` summary. The summary lists screens visited and time spent on each, milestones reached in the cycle, which client views were opened, Tier 2 screens viewed, device, landing page, referrer and UTM tags. It's only sent when the visitor submits the form themselves, and the form says so.

---

## 5. Analytics (Plausible)

Plausible sets no cookies, collects no personal data, and supports custom events with properties. Turn it on with `ANALYTICS.domain: 'demo.optimizedcompanysystems.com'`, then add these as Goals in Plausible and build Funnels from them. With `domain` left empty, the demo makes no analytics calls at all.

| Event | Properties | Tells you |
|---|---|---|
| `Intro` | `action`: completed / skipped / static | Whether people watch the intro |
| `Module opened`, `Screen time`, `Engaged time` | `module`/`screen`, `nth`, `bucket` | What gets opened, in what order, and for how long |
| **`Pulse expanded`**, **`Pulse report viewed`**, **`Pulse moved`**, **`Pulse recommendation`**, **`Pulse scenario`**, **`Pulse preview new CRM`** | `from`, `delta`, `to`, `action`, `on` | Whether Pulse lands with prospects, and which demo controls they try |
| `Milestone` | `step`: `lead_converted`, `estimate_sent`, `estimate_accepted`, `job_created`, `job_completed`, `invoice_drafted`, `invoice_sent`, `partial_payment`, `invoice_paid`, `contract_sent`, `contract_signed`, `review_requested`, `review_public`, `review_private`, `book_call_submitted`, `cycle_complete` | How far through the cycle people get |
| `Client view opened` / `step` / `closed`, `Handoff` | `type`, `mode`, `channel`, `step`, `to` | Which client views people open and where they stop |
| `Sign page opened`, `Contract signed`, `Review page opened`, `Review rating chosen`, `Review funnel`, `Returned from signing`, `Returned from review page`, `Client page exit` | `device`, `rating`, `branch` | Whether people act as the customer and come back |
| `Send`, `Converted`, `Lead moved`, `Lead created`, `Estimate accepted`, `Payment recorded` | `type`, `channel`, `from`/`to`, `via`, `partial` | Depth of use of the core CRM |
| `PDF downloaded`, `Measure opened`, `File uploaded`, `Jobs view`, `Report tab`, `Form opened` | `kind`, `live`, `view`, `tab`, `form` | Which features catch attention |
| `Tier 2 opened` | `screen` | Which deeper features people seek out |
| `Book a call opened` → `started` → `submitted` / `abandoned` / `invalid` / `post failed` | `where`, `industry`, `state`, `filled`, `fields` | The conversion funnel, and which entry point worked |
| `Tour`, `Tips`, `Tips shown`, `Tips toggled`, `Banner dismissed` | `action`, `step`, `screen` | Whether onboarding helps or gets skipped |
| `Reset`, `Maps fallback`, `Screen error`, `JS error`, `Powered by clicked` | `reason`, `where`, `msg` | Health of the live site |

---

## 6. Business Pulse in the demo

- **Dashboard card:** the score, its band, the single biggest drag on the score, the confidence level, and the baseline it's measured against. Click the card for the full breakdown: six pillars, every measure with its value, what's helping, what's hurting, and recommended actions that link to the right screen.
- **Reports → Business Pulse:** everything above, plus 90 days of score history with override days marked, and an override timeline.
- **Live recompute:** recording a payment, completing a job, moving a lead or winning an estimate updates the score immediately. On other screens a notice shows the change ("Business Pulse 76 → 78").
- **"Move it yourself"** (in the breakdown): record a payment on the oldest overdue invoice, lose a recurring client, skip ahead 30 days doing nothing, or switch between thriving, struggling and original scenarios. These change the *data*; the score always recomputes from it.
- **Settings → Business Pulse:** shows the owner's targets, and has a toggle that previews a brand-new CRM ("Establishing baseline").

How Pulse scores is proprietary and deliberately not documented here. In the shipped file the scoring configuration is encoded and the engine is minified.

---

## 7. Editing the seed data

Everything sample-related is in the block marked **`SEED DATA — edit freely`**.

- **Dates are relative.** `ago(14)` means two weeks before the page loads; `ahead(2, 9)` means the day after tomorrow at 9:00.
- **Money is in cents.** `38500` is $385.00.
- **Company:** the `company` block (name `OCS CRM`, phone, address, sending domain) flows into every email, PDF, pay page and signing page. Its `targets` drive the "vs target" measures in Pulse.
- **Pulse inputs:**
  - `ledger.invoices`: about three years of seasonal payment history, each with crew wages and materials.
  - `costs`: job costs, overhead with coverage periods, and weekly miscellaneous costs.
  - `recurring`: plans, one of them cancelled this month.
  - Lead timing fields on `leads`.
- **Below the seed:** `buildScenario` (the thriving and struggling scenarios), `buildPulseHistory` (the 90-day story) and `shiftTime` (the 30-day skip).
- **Map pins** use each record's `lat`/`lng`.

---

## 8. Origins to allowlist (WAF / CSP)

React 18 and htm are inlined in the file, along with the logos and loader animations.

| Origin | What | When |
|---|---|---|
| `https://cdn.jsdelivr.net` | jsPDF, jspdf-autotable, signature_pad, Leaflet (JS + CSS) | Only when a PDF, the signing page or a map is first needed |
| `https://tile.openstreetmap.org` | Map tiles | Only when a map is shown |
| `https://plausible.io` | Analytics | Only if `ANALYTICS.domain` is set |
| Your `BOOK_A_CALL_URL` host and `LEAD_ENDPOINTS` hosts | Calendar and form POSTs | Only when someone submits Book a call |

```
default-src 'self';
script-src 'self' 'unsafe-inline' https://cdn.jsdelivr.net https://plausible.io;
style-src 'self' 'unsafe-inline' https://cdn.jsdelivr.net;
img-src 'self' data: blob: https://tile.openstreetmap.org;
media-src 'self' blob: data:;
connect-src 'self' https://plausible.io https://formspree.io https://<your-crm-intake-host>;
frame-src 'self' blob:;
```

**Map tiles:** OpenStreetMap's public tile servers are free and need no key. They are run for normal, light use, and require the © OpenStreetMap credit (it's shown on every map). A demo site is well within that. If traffic ever grows large, swap the `TILES` URL in the maps code for a commercial provider.

**What leaves the browser:** no email, text or payment is ever sent. Downloading the libraries and map tiles carries nothing about what the visitor does. Analytics (anonymous) and a Book-a-call submission are the only calls that carry anything about the visit.

---

## 9. Known behaviours

- If the map library or tiles can't load (offline, or a firewall), maps fall back to an address card, the jobs and route maps fall back to a schematic pin view, and the measuring tool switches to a sample plan. Nothing shows an error box.
- On a laptop the signing page is signed with a mouse or trackpad; on a phone, with a finger.
- Pulse override caps sit inside the score bands as defined: a score capped at 59 reads as "Attention Needed", and anything below 55 reads as "Action Required".
