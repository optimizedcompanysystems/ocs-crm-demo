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

## 2. Deploy (Vercel + Cloudflare DNS + Web3Forms)

The folder to deploy contains:

```
CRM Demo/
  index.html        ← the demo (paste your Web3Forms access key into CONFIG first)
  .vercelignore     ← keeps this README off the public site
```

1. In Vercel, create a **separate project** (framework preset "Other", no build command). No environment variables are needed.
2. Attach `demo.optimizedcompanysystems.com` with a CNAME record in Cloudflare (DNS only, grey cloud).

The whole demo, including the Book a call form, works from a local `python3 -m http.server` too.

## 3. What's in the demo

- **Intro:** a dark OCS logo reveal on the first visit of each session. Visitors can skip it (button, click, Esc or Enter). It never shows on the client-facing signing and review pages, and visitors who prefer reduced motion see a static logo instead.
- **Business Pulse:** see section 6.
- **The full CRM cycle:** lead → client → estimate → job → invoice → paid. A step-by-step "What the client sees" view covers the email, pay page, PDF, receipt, text messages, contract signing and both branches of the review funnel.
- **Maps:** OpenStreetMap tiles shown through Leaflet and styled dark. **No key is needed.** They appear on client profiles, the jobs map, the route map and the **property measuring tool**.
  - The measuring tool: outline an area on the map to get square feet, acres and perimeter, then "Start an estimate" adds it as a line item.
  - Find it from the dashboard ("Measure a property on the map"), Estimates ("Measure & quote"), any client profile ("Measure"), any job's menu, or the line-item editor ("Measure a property").
- **Book a call with OCS:** the banner across the top opens a panel with name, business, phone, email, state, industry and an optional note. It's styled as OCS's, not as a CRM button. Branded versions also appear at the end of each client view, in the Tier 2 explainers and in the Settings notice.

---

## 4. Book a call → your inbox (Web3Forms)

In the `CONFIG` block near the bottom of `index.html`, set:

```js
WEB3FORMS_ACCESS_KEY: 'your-access-key',
```

Web3Forms access keys are designed to be public, so the key can sit in the page.

Each request arrives by email with the subject "Demo call request — Name (Business)". It contains:
- Name, business, industry, state, phone and email.
- Their note.
- Where they clicked Book a call.
- A summary of what they explored: time on the demo, screens by time, milestones reached, client views opened, and "Built to your workflow" screens viewed.

The form also:
- Drops bots that fill its hidden honeypot field.
- Shows "Not sent yet" with a **Send again** button if Web3Forms can't be reached.
- Tells the visitor it isn't connected if no key has been set, rather than losing the request silently.

Optional settings:
- `LEAD_ENDPOINTS`: URLs that also receive each request as JSON, for example a lead-intake endpoint on your CRM.
- `BOOK_A_CALL_URL`: a Cal.com or Calendly link that opens after submitting, with the name and email filled in.

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
| `https://api.web3forms.com`, plus your calendar host if set | Book-a-call submission | Only when someone submits Book a call |

```
default-src 'self';
script-src 'self' 'unsafe-inline' https://cdn.jsdelivr.net https://plausible.io;
style-src 'self' 'unsafe-inline' https://cdn.jsdelivr.net;
img-src 'self' data: blob: https://tile.openstreetmap.org;
media-src 'self' blob: data:;
connect-src 'self' https://plausible.io https://api.web3forms.com;
frame-src 'self' blob:;
```

**Map tiles:** OpenStreetMap's public tile servers are free and need no key. They are run for normal, light use, and require the © OpenStreetMap credit (it's shown on every map). A demo site is well within that. If traffic ever grows large, swap the `TILES` URL in the maps code for a commercial provider.

**What leaves the browser:** no email, text or payment is ever sent. Downloading the libraries and map tiles carries nothing about what the visitor does. Analytics (anonymous) and a Book-a-call submission are the only calls that carry anything about the visit.

---

## 9. Known behaviours

- If the map library or tiles can't load (offline, or a firewall), maps fall back to an address card, the jobs and route maps fall back to a schematic pin view, and the measuring tool switches to a sample plan. Nothing shows an error box.
- On a laptop the signing page is signed with a mouse or trackpad; on a phone, with a finger.
- Pulse override caps sit inside the score bands as defined: a score capped at 59 reads as "Attention Needed", and anything below 55 reads as "Action Required".
