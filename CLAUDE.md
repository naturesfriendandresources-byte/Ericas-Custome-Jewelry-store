# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Assistant identity

When interacting with the owner in this project, the assistant should refer to itself as **Linda**.

## Repository status

This repository is currently a greenfield project — it contains only `README.md` and has no source code, build tooling, tests, dependencies, or CI configuration. Do not assume a stack (framework, language, package manager) has been chosen; ask before scaffolding one.

## Project intent

> Note: the README.md says "custom jewelry," but in conversation with the owner the actual scope was clarified. Treat the description below as authoritative over the README until the README is rewritten.

The owner ("Erica") sells two related but distinct product types:

1. **Vintage costume jewelry that she collects and resells** — rings, bracelets, necklaces, brooches. Each piece is effectively 1-of-1. Research is meaningful here: hallmarks, makers (e.g. Trifari, Coro, Weiss, Eisenberg), era, materials.
2. **Handmade ceramic items she makes herself** — ornaments, pins, magnets. Could be 1-of-1 or small batches; she is the source of truth on what they are, so research-from-photos is less relevant for these.

The system supports:

1. **Image processing** — owner uploads photos; the system applies simple edits (crop, rotate, color/exposure, background cleanup) to produce listing-quality images. AI-assisted editing is a later version.
2. **Multi-platform listing management** — the system syndicates listings to eBay and Etsy, and keeps a sold/unsold status synced across platforms and the portfolio site.
3. **Description generation** — for vintage pieces, the system researches the item from photos and notes (hallmarks, designer, era) and proposes a draft description for the owner to edit. For ceramics, descriptions can come from owner-supplied facts plus AI rewrite.
4. **Audience / CRM / email marketing** — the storefront grows a following: collect customer/follower emails (newsletter signup, post-purchase capture from eBay/Etsy where possible), maintain a customer database, and send email blasts (e.g. new arrivals, era-themed drops). Treat this as a first-class product capability, not an add-on.

## Decisions made so far

- **Form factor**: a web app / web store that Erica owns (not a desktop or mobile app).
- **Storefront role**: the web store is **portfolio-only**. It does **not** take orders, run a cart, or process payments. Every "Buy" action on the storefront links out to the corresponding eBay or Etsy listing.
- **Listing destinations**: portfolio storefront we build, plus syndication to **eBay** and **Etsy**. Mercari was considered and explicitly **dropped** because it has no public seller API.
- **Platform integration rule**: **only integrate with platforms that have an official public listing API.** Browser-automation or third-party-paid integrations are out of scope. This rules out Mercari, Poshmark, and Depop unless they ship a real API.
- **Listing data storage**: a **database owned by the web app** is the source of truth. Erica adds/edits pieces through the web UI; the same record drives the storefront and the eBay/Etsy syncs. Do not propose spreadsheets, CSV-as-source, or per-platform-API-as-source.
- **Sold-status sync**: **auto-remove**. When a piece sells on eBay or Etsy, the system must end the listing on the other platform and hide it from the portfolio. Critical because vintage pieces are 1-of-1 and double-selling must be prevented. Use platform webhooks/notifications where available, fall back to polling.
- **Description generation**: **research + AI draft** for vintage pieces (the system attempts to identify hallmarks/designer/era from photos and notes, then proposes a draft Erica edits). For ceramics, AI rewrites owner-supplied facts. Owner always edits before publishing.
- **Image processing v1**: simpler tools only — crop, rotate, color/exposure adjust, background cleanup. AI-assisted editing (auto-angles, generative cleanup) is planned for a later version, not v1.
- **Storefront name**: **ELM Vintage**.
- **Domain**: not registered yet; use a placeholder host (e.g. a Vercel/Netlify/Render subdomain) during development and acquire a real domain just before launch.
- **Publish flow**: **one-click simultaneous publish** to eBay + Etsy + storefront after the owner approves the draft. No staged/delayed publishing in v1.
- **Pricing**: **same price on every platform** (single `price` field per piece; no per-platform overrides).
- **Shipping**: use **platform-calculated shipping** (buyer ZIP + package weight). Therefore each piece's database record must store **package weight** (and likely dimensions) so eBay and Etsy can compute rates at checkout.
- **Scale target**: medium — 50–300 active listings, 10–30 new pieces per month. Design for this; don't over-engineer for thousands.
- **Hosting budget**: ~$20–50/month, so paid managed services (managed Postgres, object storage, always-on workers) are acceptable; we are not constrained to free tiers.
- **Seller accounts**: Erica has active seller accounts on both **eBay** and **Etsy**, but does **not** yet have developer/API access on either. Registering an eBay Developer Program app and an Etsy app (with OAuth) is a prerequisite for any syndication work.
- **Existing listings**: Erica chose to **re-list from scratch** rather than import current eBay/Etsy listings. The system is the new source of truth from day one; she will manually end old duplicates as she re-lists. Implication: no import job needed, but expect a transition period with stale listings on the platforms.
- **Returns policy default**: **30-day returns, buyer pays return shipping**, applied as the default to every new listing on both eBay and Etsy. Owner can override per-listing later if needed.
- **Categorization axes**: pieces are organized by **type** (necklace, bracelet, brooch, ring, ornament, magnet, pin) and by **era** (Victorian, Art Deco, Mid-Century, 80s, etc.). Maker/designer and material are **not** primary axes in v1, even though the AI-research step may surface them.
- **Admin login**: assumed single-user (Erica only) for v1 unless changed later. The "customer database" mentioned in conversation is a marketing audience (follower emails), not a login system.
- **Email service provider**: **Mailchimp**. Use the Mailchimp Marketing API to push new subscribers and to send/track campaigns. Free tier (≤500 contacts) is fine until the list grows.
- **Newsletter signup placements** (all four enabled in v1): site-wide footer form, homepage modal/popup (frequency-capped — show at most once per visitor per N days; do not nag), dedicated `/subscribe` page, and inline form on each piece-detail page that pre-tags the subscriber by category/era so themed campaigns can target them.
- **Email cadence**: **ad-hoc only** in v1. Build a "compose blast" feature with list/segment picker; do **not** build an automated weekly-digest cron job.
- **Post-purchase email capture** (compliant with CAN-SPAM and platform ToS — buyer must opt in): two channels, both enabled. (a) System generates a printable thank-you card per shipment with a QR code/short URL to the subscribe page; (b) automated post-sale message via eBay's and Etsy's seller-to-buyer messaging APIs containing a subscribe link. Never auto-add a buyer to the list without explicit opt-in.
- **AI provider**: **Claude (Anthropic)** for vintage research and description drafts. The system sends Claude the photos plus owner notes; Claude proposes era/maker/materials/condition cues and writes a draft description. Default to the latest Sonnet for cost/quality balance; reserve Opus for hard cases or batch enrichment.
- **Description voice**: **style-reference flow.** Erica will paste 2–3 of her existing favorite listings into a "voice samples" admin setting; every draft prompt includes those samples as the style reference Claude must match. Do **not** hardcode a tone — her voice is the source of truth, captured by example.
- **Photo capture**: mixed — phone (3–6 MB) for quick listings, dedicated camera (10–20 MB) for special pieces. Upload UX must accept both gracefully; system stores originals and generates web-sized derivatives. Plan for both file-size regimes from day one.
- **Background cleanup**: hosted API (e.g. **remove.bg** or **Photoroom**), per-photo opt-in via a "remove background" button rather than always-on. Budget ~$0.10–$0.20/image so the cost is bounded by listing volume.
- **Condition grading**: standard 6-tier vintage scale — **Mint / Excellent / Very Good / Good / Fair / As-Is** — stored as a structured field on every piece, plus a free-text "Condition notes" field for specifics (e.g. "tiny enamel chip on back"). Claude may **propose** a grade from photos as part of the draft; the owner must confirm before publishing.
- **Storefront aesthetic**: **vintage direction** chosen at the high level; the specific look (warm-editorial vs. antique-shop vs. magazine) is to be picked from real HTML/CSS mockups generated after the stack is scaffolded. Reference shops the owner can browse for inspiration include 1stDibs, Ruby Lane, and curated vintage Etsy shops.
- **Journal / blog**: **out of v1 scope**. Launch without it; revisit in v2. Do not build a CMS or markdown post system in v1.
- **Shipping zone**: **US only** in v1. Do not enable international shipping options on eBay or Etsy listings created by the system. (This simplifies shipping config; revisit before any expansion.)
- **Sales analytics dashboard (v1 scope)**: includes **all four** metrics — total revenue (month / year / all time), a chronological sales feed (what sold, where, when, for how much), time-to-sell broken down by type and era, and a slow-mover list of pieces unsold for >60 days. Build the dashboard read-only on top of the platform sale events that drive sold-status sync.
- **Slow-mover handling**: **flag-only.** The system surfaces stale pieces on a "needs attention" view; the owner decides whether to drop price, re-photograph, or relist. Do **not** auto-drop prices or auto-relist.
- **Mobile experience**: **mobile-first**. Design the storefront for phones first (followers will arrive from email and Instagram links), then scale up to desktop. The admin app can be desktop-first since Erica works on a computer to upload and edit.
- **Buyer bulk discount**: **store-wide, always on** — "buy 2+ pieces, get 10% off." System configures the standing promotion on both eBay (Multi-buy) and Etsy (Sales and discounts) and reapplies if it gets cleared. The threshold (2+) and discount (10%) live in admin settings and can be changed; the default is what's stated here.
- **Voice samples**: deferred. Owner will provide 2–3 favorite existing descriptions when the AI description-drafting feature is being built. Until then, do not attempt to imitate a tone — block the feature on having samples.
- **Admin bulk operations**: undecided — owner asked for clarification. Do not assume bulk-publish, bulk-price-change, or bulk-relist are in scope yet.
- **Tech stack**: deferred. Do not pick a language/framework yet; revisit once image-tool scope and listing-API integrations are clearer.

### Future platforms to evaluate (still gated on the API-only rule)

- **Facebook / Instagram Shopping** — Meta Commerce / Catalog API exists; could be in scope later.
- **Amazon Handmade** — has APIs, but requires application/approval.

## Working in this repo

- Before writing code, confirm any still-undecided items above with the user (especially listing-data storage and the tech stack).
- There is no `package.json`, `requirements.txt`, `Makefile`, or equivalent — when one is added, update this file with the actual build / test / lint / run commands.
- The active development branch (per task instructions when this file was created) is `claude/add-claude-documentation-vl0eJ`. The default branch is `main`.
