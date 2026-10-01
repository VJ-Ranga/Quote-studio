# CloudyCode Social Media Pricing Calculator — Build Spec

Build a single-page pricing calculator for CloudyCode's social media services. A staff member picks platforms, content quantities and extras; the page shows a line-item quote, the total, and which standard package (if any) is better value.

The pricing logic below reproduces CloudyCode's three published package prices exactly (Essential Rs 15,000, Premium Rs 35,000, Ultimate Rs 75,000), now with ad campaign setups included (2 / 4 / 6). Ad campaign setup is Rs 2,000 each; the ad budget itself is always separate. Every other price, including the new in-between packages, comes from the same formula.

---

## 1. Tech requirements

- One self-contained `index.html` (HTML + CSS + vanilla JS, no build step, no backend).
- All prices and rules live in one `CONFIG` object at the top of the script so they can be edited without touching logic.
- Currency: Sri Lankan Rupees, displayed as `Rs 15,000` (comma thousands, no decimals).
- Works on desktop and phone (single column under 640px).
- Print stylesheet so "Print / Save as PDF" gives a clean one-page quote.
- Include a test runner (section 9) that runs on page load when the URL has `?test=1` and logs pass/fail to the console and a small on-page panel.

---

## 2. Two pricing modes

| Mode | Use for | What's charged |
|---|---|---|
| **Monthly management** (default) | Ongoing monthly social media clients | Content + ad campaign setup + platform management fee + extras, then rounded |
| **One-off campaign** | Events and short campaigns (e.g. a 10-day event push) | Content + ad campaign setup + extras only. No platform management fee, no rounding |

---

## 3. Rate card (CONFIG)

```js
const CONFIG = {
  currency: "Rs",

  content: {
    staticPost:      { label: "Static post",               price: 1000 },
    reel:            { label: "Reel / short video",         price: 2000 },
    adCampaign:      { label: "Ad campaign setup & management", price: 2000 },
  },

  // Extras: NOT part of the published packages. Defaults are placeholders — confirm before going live.
  extras: {
    story:           { label: "Story design",               price: 500,  placeholder: true },
    carousel:        { label: "Carousel post (up to 5 slides)", price: 2500, placeholder: true },
    youtubeLong:     { label: "YouTube long video (2–5 min edit)", price: 8000, placeholder: true },
    eventCoverage:   { label: "Event photo/video coverage (half day)", price: 10000, placeholder: true },
  },

  platforms: {
    facebook:  { label: "Facebook" },
    instagram: { label: "Instagram" },
    tiktok:    { label: "TikTok",   requiresVideo: true },
    youtube:   { label: "YouTube",  requiresVideo: true },
    linkedin:  { label: "LinkedIn" },
  },

  // Monthly mode only. Fee is charged PER SELECTED PLATFORM, based on total monthly pieces.
  // pieces = staticPosts + reels + carousels + youtubeLong  (stories and ads do NOT count)
  managementTiers: [
    { maxPieces: 6,        feePerPlatform: 1500 },
    { maxPieces: 12,       feePerPlatform: 4250 },
    { maxPieces: 18,       feePerPlatform: 6500 },
    { maxPieces: Infinity, feePerPlatform: 9000 },
  ],

  // Monthly mode only: round the final total to the nearest Rs 1,000 (exact half rounds UP).
  roundTo: 1000,

  // Optional prepayment discount (monthly mode). Applied after rounding.
  prepayDiscounts: [
    { months: 1,  pct: 0 },
    { months: 3,  pct: 5 },
    { months: 6,  pct: 10 },
  ],

  // Suggest a package if it costs no more than this % above the custom quote.
  upsellThresholdPct: 10,
};
```

Show a small "placeholder rate" badge in the UI next to any extra with `placeholder: true`, so staff know it isn't a confirmed price.

---

## 4. Calculation (monthly mode)

```
contentCost   = posts × 1,000 + reels × 2,000 + carousels × 2,500 + stories × 500 + youtubeLong × 8,000
adCost        = adCampaigns × 2,000
extrasCost    = eventCoverage × 10,000
pieces        = posts + reels + carousels + youtubeLong
feePerPlat    = first tier where pieces ≤ maxPieces
managementFee = feePerPlat × numberOfSelectedPlatforms
subtotal      = contentCost + adCost + extrasCost + managementFee
total         = roundHalfUp(subtotal, 1000)          // floor((subtotal + 500) / 1000) × 1000
discounted    = total × (1 − prepayPct/100), rounded to nearest Rs 100 (half up: 33,250 → 33,300)
```

**One-off campaign mode:** `total = contentCost + adCost + extrasCost` (no management fee, no rounding, no prepay discount).

**Ad budget** (the money paid to Meta/TikTok/Google) is a separate optional input. Show it on the quote as "Ad budget (paid by client directly to the platform)". **Never add it to the CloudyCode total.**

### Worked check: the formula reproduces the published packages

| Package | Platforms | Posts | Reels | Ads | Content + ads | Pieces → fee | Mgmt fee | Subtotal | **Total** |
|---|---|---|---|---|---|---|---|---|---|
| Essential | 2 | 4 | 2 | 2 | 12,000 | 6 → 1,500 | 3,000 | 15,000 | **15,000** |
| Premium | 3 | 6 | 4 | 4 | 22,000 | 10 → 4,250 | 12,750 | 34,750 | **35,000** |
| Ultimate | 4 | 15 | 6 | 6 | 39,000 | 21 → 9,000 | 36,000 | 75,000 | **75,000** |

---

## 5. Standard packages (presets)

Each preset is a button that fills the form. Prices are computed by the formula, not hard-coded. The test suite (section 9) confirms they come out as listed.

| # | Package | Platforms | Posts | Reels | Ads | Price | Status |
|---|---|---|---|---|---|---|---|
| 1 | Essential | FB, IG | 4 | 2 | 2 | Rs 15,000 | Published |
| 2 | Essential Plus | FB, IG | 5 | 3 | 2 | Rs 24,000 | New |
| 3 | Growth | FB, IG, TikTok | 6 | 3 | 2 | Rs 29,000 | New |
| 4 | Premium | FB, IG, TikTok | 6 | 4 | 4 | Rs 35,000 | Published (Most popular) |
| 5 | Premium Plus | FB, IG, TikTok | 10 | 5 | 4 | Rs 48,000 | New |
| 6 | Pro | FB, IG, TikTok, YouTube | 10 | 5 | 4 | Rs 54,000 | New |
| 7 | Elite | FB, IG, TikTok, YouTube | 12 | 6 | 5 | Rs 60,000 | New |
| 8 | Ultimate | FB, IG, TikTok, YouTube | 15 | 6 | 6 | Rs 75,000 | Published |

All monthly packages include caption creation, hashtag research, profile management and a monthly report. List these as "Included in every monthly plan" on the page and on the quote. They are not priced separately.

LinkedIn is in no preset. Staff add it to any preset by ticking it; the fee recalculates automatically, which works as the LinkedIn add-on.

---

## 6. UI layout

**Left column: inputs**
1. Mode toggle: Monthly management / One-off campaign.
2. Preset buttons (8 packages). Clicking one fills everything below and shows "Based on: Premium". Editing any field changes the label to "Custom (based on Premium)".
3. Platforms: 5 checkboxes with platform icons (inline SVG).
4. Content steppers (− / number / +, min 0, max 50): static posts, reels, carousels, stories.
5. Ad campaigns stepper (0–10).
6. Extras steppers: YouTube long videos, event coverage.
7. Ad budget input (optional, Rs), with helper text "Paid by client to the platform. Not included in the total."
8. Prepay dropdown (monthly only): 1 / 3 / 6 months.
9. Client name (optional): appears on the printed quote.

**Right column: live quote (sticky on desktop)**
- Line items: label, quantity × rate = amount. Hide zero lines.
- Management fee line: "Platform management — 3 platforms × Rs 4,250 (7–12 pieces/month)".
- Rounding line if non-zero: "Rounding +Rs 500".
- **Total**, large. "/month" suffix in monthly mode.
- Prepay line if selected: "3-month prepay (5% off): Rs 33,300/month · Rs 99,900 total".
- Ad budget line, visually separated, marked "not included".
- Warnings (section 7).
- Upsell suggestion (section 8).
- Buttons: Copy quote as text (WhatsApp-friendly, plain text with line breaks), Print / Save PDF, Reset.

**Branding:** CloudyCode. White background, blue `#2F6BFF` and purple `#8B3DFF` accents with a blue→purple gradient on the main heading, dark text `#1A1D29`, rounded cards. Contact in footer: info@cloudycode.net · +94 76 635 1053.

---

## 7. Validation and warnings

Show warnings inline in the quote. They never block the price.

| Condition | Warning |
|---|---|
| No platform selected (monthly mode) | "Select at least one platform." Total shows Rs 0. |
| No content (posts + reels + carousels + YouTube long = 0) | "Add at least one post or video." |
| TikTok or YouTube selected but reels + YouTube long = 0 | "TikTok/YouTube need video content. Add at least 1 reel." |
| Ad campaigns > posts + reels + carousels | "More ad campaigns than content pieces. Each campaign needs a creative." |
| Pieces is exactly 6, 12 or 18 | "Tier limit: one more piece moves management to the next tier (+Rs X)." Compute X. |
| Any placeholder-priced extra used | "Includes placeholder rates. Confirm before sending." |

---

## 8. Package suggestion (upsell)

Monthly mode only. After computing the custom total, check every preset. A preset qualifies if **all** of these are true:
- It includes every platform the custom quote selected. LinkedIn is the exception: if it's selected, add it to the preset and recompute the preset's price with LinkedIn included.
- Its posts, reels and ad campaigns are each ≥ the custom quote's.
- Its price (recomputed with extras and LinkedIn carried over) is ≤ custom total × (1 + 10%).

If one qualifies, show the cheapest one: "For Rs X more you get the Growth package: +1 post, +1 reel." If a preset gives at least as much for the same or lower price, say "Growth gives you more for the same price". Clicking the suggestion loads that preset, keeping any extras and LinkedIn.

---

## 9. Test cases (must all pass with `?test=1`)

| # | Mode | Platforms | Posts | Reels | Ads | Other | Expected total |
|---|---|---|---|---|---|---|---|
| T1 | Monthly | FB, IG | 4 | 2 | 2 | – | 15,000 |
| T2 | Monthly | FB, IG, TikTok | 6 | 4 | 4 | – | 35,000 |
| T3 | Monthly | FB, IG, TikTok, YouTube | 15 | 6 | 6 | – | 75,000 |
| T4 | Monthly | FB, IG, YouTube, LinkedIn | 6 | 0 | 2 | – | 16,000 + YouTube-needs-video warning |
| T5 | Monthly | FB, IG, TikTok, LinkedIn | 4 | 2 | 2 | – | 18,000 (Essential + TikTok + LinkedIn) |
| T6 | One-off | FB, IG | 3 | 2 | 2 | – | 11,000 |
| T7 | Monthly | FB, IG | 5 | 2 | 0 | – | 18,000 (subtotal 17,500, rounds up) |
| T8 | Monthly | FB, IG, TikTok | 6 | 4 | 4 | 3-month prepay | 35,000 → 33,300/month |
| T9 | Monthly | FB, IG | 4 | 2 | 2 | Ad budget 20,000 | Total stays 15,000; ad budget shown separately |
| T10 | Monthly | FB, IG | 6 | 0 | 0 | – | 9,000 + "Tier limit" warning showing +Rs 7,000 (a 7th post totals 16,000) |
| T11 | Presets | each of the 8 presets | | | | | 15,000 / 24,000 / 29,000 / 35,000 / 48,000 / 54,000 / 60,000 / 75,000 |

The tier-limit figure X = (rounded total with one more static post) − (current rounded total). Compute it in code; never hard-code it.

---

## 10. Copy-as-text quote format

```
CloudyCode – Social Media Quote
Client: Range Global Education
Plan: Custom (based on Premium) · Monthly

Platforms: Facebook, Instagram, TikTok, LinkedIn
6 static posts × Rs 1,000 ........ Rs 6,000
4 reels × Rs 2,000 ................ Rs 8,000
1 ad campaign setup × Rs 2,000 .... Rs 2,000
Platform management (4 × Rs 4,250)  Rs 17,000
TOTAL ............................. Rs 33,000 / month

Included: caption creation, hashtag research, profile management, monthly report
Ad budget (paid directly to the platform, not included): Rs 20,000

info@cloudycode.net · +94 76 635 1053
```

---

## 11. Out of scope for v1

- Saving quotes or client history (no backend).
- Multiple currencies.
- Logins.
