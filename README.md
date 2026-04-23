# Riviera Match

A private decision aid for one buyer. Jack has already visited every property.
This tool is his live reminder: he walks through the shortlist again (alone or
with his partner), rates each property on ten features, and sees a ranked match
at the end.

Deployed at **riviera-match.vercel.app**.
Single self-contained `index.html` — vanilla HTML/CSS/JS, no build step.
External assets: Google Fonts (Cormorant Garamond + Montserrat) + Font Awesome 6.5.1 via CDN.

## Flow

1. **Hero** — sets the tone; Jack can name the partner rating with him (defaults to "Jack's wife").
2. **Essentials** — min bedrooms / bathrooms / pets. Pets=Yes excludes Paseo del Sol.
3. **Property review** — five headline properties in alphabetical order.
   - Mobile: one at a time, wizard style, with progress dots.
   - Desktop (≥ 900px): grid of tiles, click to open a modal with full rating UI.
   - Each feature row shows dual ratings: Jack and partner, stacked on mobile,
     side-by-side in columns on desktop.
4. **Results** — combined ranking, agreement insight, individual top 3s for each rater.
5. **Share** — Email, WhatsApp, Copy link. Hash-encoded URL resumes state.

## Change the client name or default partner

Top of the `<script>` in `index.html`:

```js
const CLIENT = { name: "Jack", defaultPartner: "Jack's wife" };
```

`CLIENT.name` drives the hero personal line ("Designed for you, Jack."), the
results title, each share message, and the "Jack" rater label. `defaultPartner`
is only the placeholder — whatever the user types on the hero screen wins.

## Add a property

Paste an object into the `PROPERTIES` array. It appears in alphabetical order
based on `condo` (sort yourself — the array is rendered in insertion order).

```js
{
  id: "la-vela",                              // unique slug used in URL state
  condo: "La Vela",
  unit: "2BR Loft",
  location: "Tulum (jungle-edge)",
  priceDisplay: "$520,000 USD",               // rendered exactly as-is; supports
                                              // "From $13,000,000 MXN", "Price on request", etc.
  beds: 2, baths: 2, m2: 140,                 // beds/baths must pass filter gates
  pets: true, golfCart: false,
  preConstruction: false,                     // optional — adds a gold tag
  standout: "One-line selling point in italic quotes",
  hoa: "$420/mo USD",                         // or null
  tags: ["Pet-friendly"],                     // "Beachfront" auto-added if location matches
  hero: null,                                 // see "Add real photos" below
  gallery: [null, null, null]
}
```

Then the ratings/URL state automatically extend to cover the new property.

## Add real photos

Every property has two slots for images:

```js
hero: "https://.../images/la-vela-hero.jpg",
gallery: [
  "https://.../images/la-vela-terrace.jpg",
  "https://.../images/la-vela-pool.jpg",
  "https://.../images/la-vela-view.jpg"
]
```

Rules:
- **`hero`** — used for the desktop tile thumbnail AND the big card at the top
  of the mobile rating view. Landscape orientation (4:3 or 16:9) looks best.
- **`gallery`** — up to 3 URLs. Missing slots render as gold-bordered placeholders,
  so a half-filled gallery is fine. If at least one image is present, the
  "photos coming soon" caption disappears.
- Any reachable URL works — Vercel static asset, Cloudflare, S3, external CDN.
  For self-hosted images, drop them in the project root (e.g. `/photos/awa-hero.jpg`)
  and reference as `"/photos/awa-hero.jpg"`.
- Quotes inside URLs are escaped automatically.

Leaving `hero: null` and `gallery: [null, null, null]` is valid — the tool
shows styled placeholders in the meantime.

## Scoring

Each of the two raters scores every property on 10 features (1–5).
`jackPct = (avg of Jack's 10 ratings) × 20`, same for partner.
Combined = `((jackAvg + partnerAvg) / 2) × 20`, rounded.

Rankings are sorted by `combinedPct` descending. The agreement insight compares
each rater's top choice (and top 3 set) and shows one of three messages:
- Both #1 same AND top 3 identical → "and you agree on the top three."
- Both #1 same → "a shared favourite."
- Different #1s → "worth a conversation."

## URL state

`#f=beds,baths,petsFlag&partner=Name&r=propId:jjjjjjjjjj|pppppppppp,...`

- `f` — three filter values
- `partner` — URL-encoded partner name
- `r` — comma-separated entries per property: ten digits for Jack's ratings,
  a pipe, ten digits for the partner's — in `FEATURES_FLAT` order
  (location, beach, views, size, finishes, outdoor, price, pool, gym, security)

On load, a valid hash auto-populates state and jumps straight to results.
"Copy link" refreshes the hash and copies.

## Deploy

```
vercel --prod
```

Included `vercel.json` sets `cleanUrls: true`. No other config needed.
