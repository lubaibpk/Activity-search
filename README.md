# TaxitWorld Business Activity Finder

A static, Vercel-only web app for searching Saudi Business Center (SBC/MISA)
commercial activities and ISIC codes. Built for TaxitWorld to share with clients.

## The real hierarchy (fixed — this was a major gap in earlier versions)
The Saudi Business Center form actually has **three levels**, not two:
1. **Main Activity** — 19 broad categories (e.g. "09 – Accommodation and food
   service activities")
2. **Sub Activity** — 377 ISIC-coded groups (e.g. "5610 – Restaurants and
   mobile food service activities")
3. **Activity** — 2,669 specific, actually-registrable codes (e.g. "561010 –
   Restaurants with service"). **This is the real commercial registration
   code** that goes on the CR. Earlier versions of this tool stopped at level
   2 and missed this entirely.

Every activity also carries a **"Requires additional license"** flag where
the SBC source noted it — 2,028 of the 2,669 activities need extra licensing
beyond standard commercial registration. This shows as a badge on the
activity card, in the cart, and in the WhatsApp message.

Browsing now drills down properly: Main Activity → Sub Activity → Activity,
matching the real form. Search bypasses the drill-down and searches directly
across all 2,669 activities, showing the full breadcrumb on each result.

## Features
- **Smart search** — understands business terms, not just exact text (see below)
- **Cart** — tap "+ Add to cart" on any activity (in search results, category
  browse, or the related-activities list). A floating cart button shows the
  count; open it to review selections. Each entry shows both the **Main
  Activity** (with its official 2-digit code) and the **specific activity**
  (with its full 4-digit code) it belongs to.
- **Send via WhatsApp** — from the cart, optionally enter the client's
  WhatsApp number, then tap "Send via WhatsApp" to open a pre-filled message
  listing every selected activity (main + sub), ready to send. Leave the
  number blank to use WhatsApp's contact picker instead.
- Cart persists in the browser (localStorage) so it survives a page refresh
  during a client conversation.
- Every listing always shows the **full activity code** (not abbreviated).

## What it does
- **Smart search** — understands business terms, not just exact text.
  Search "restaurant", "car rental", "cleaning company", "HR staffing agency"
  and it ranks the closest-matching official activities, even when the
  wording doesn't match exactly.
- Search by ISIC code directly (e.g. "6201")
- Browse all 19 main activity categories
- Tap any activity to see **all related activities**, ranked by relevance
  across the entire dataset — not limited to one fixed category

## How the "intelligence" works
Runs entirely in the visitor's browser, with no AI API and no backend:
- TF-IDF (term frequency–inverse document frequency) vectors built from
  all 377 activity descriptions at page load
- Cosine similarity ranking between the search query and every activity
- A built-in business-term synonym dictionary (e.g. "car" ↔ "motor vehicle",
  "consultancy" ↔ "advisory", "HR" ↔ "employment/staffing") so everyday
  language finds the right official ISIC wording
- Exact code/name matches are always boosted to the top

This keeps the whole app static — safe, fast, free to host, no API keys,
and it still qualifies as "Vercel only."

## Deploy to Vercel (2 minutes)

### Option A — Vercel dashboard (no coding needed)
1. Go to https://vercel.com/new
2. Choose "Deploy without Git" / drag-and-drop
3. Drag this whole folder in and deploy
4. Vercel auto-detects it as a static site — no build settings needed

### Option B — Vercel CLI
```
npm i -g vercel
cd taxitworld-activity-finder
vercel --prod
```

### Option C — GitHub + Vercel (recommended for updates later)
1. Push this folder to a new GitHub repo (e.g. `lubaibpk/activity-finder`)
2. Import the repo at https://vercel.com/new
3. Framework preset: "Other" (static) — no changes needed
4. Deploy

## Files
- `index.html` — the whole app: UI + the search engine, no build step, no dependencies
- `activities_data.json` — the extracted activity dataset (377 activities, 19 categories)
- `vercel.json` — clean URLs config

## Updating the data later
If SBC adds/changes activities, replace `activities_data.json` with a new
export in the same format and redeploy. No code changes needed.

## Tuning the smart search
The synonym dictionary lives near the top of the `<script>` block in
`index.html` (the `SYNONYMS` object). It already covers common Gulf-Malayali
client business terms: money exchange/remittance, gold & jewellery, saree/
tailoring/abaya/embroidery, manpower/labour supply, cargo & shipping to
Kerala, typing centers & attestation/documentation offices, wedding halls &
event catering, tea shops, cold storage, general trading, driving schools,
interior decoration, plumbing/electrical/AC maintenance, and more.

**Important:** dictionary keys must be plain, unstemmed words exactly as a
person would type them (e.g. `typing`, not `typed`) — the code looks up
synonyms *before* stemming, specifically to avoid the stemmer mangling
short keys. Add new terms as `keyword: ['related','words','here']` and it
takes effect immediately, no rebuild required.
