# ELM Vintage — v1 Scope

What's in v1, what's deferred, and what we're waiting on.

The authoritative source for individual decisions is [`/CLAUDE.md`](../CLAUDE.md). This document organizes those decisions by capability so it's easy to read top-to-bottom.

---

## Product summary

ELM Vintage is a portfolio web store for Erica's vintage costume jewelry (resold) and handmade ceramic ornaments/pins/magnets. It does not take orders — every "Buy" links out to the corresponding eBay or Etsy listing. The system manages listings, syndicates them to eBay and Etsy, and grows an email audience.

The four capabilities:

1. **Image processing** — owner uploads photos; system applies simple edits and generates web-quality derivatives.
2. **Multi-platform listing management** — create once, publish to eBay + Etsy + storefront; sold-status auto-syncs across platforms.
3. **Description generation** — Claude analyzes photos + notes, identifies era/maker/materials, drafts a description in Erica's voice.
4. **Audience / email marketing** — Mailchimp-powered newsletter with multiple signup placements, ad-hoc blasts, compliant post-purchase capture.

---

## v1 scope

### Image processing
- Upload from phone or dedicated camera (3–6 MB and 10–20 MB sources both supported).
- Editing tools: crop, rotate, color/exposure adjust.
- Background removal via hosted API (remove.bg or Photoroom), opt-in per photo.
- Non-destructive editing: originals preserved, edits stored as separate derivatives.
- Web derivatives include a subtle "ELM Vintage" corner watermark (configurable, originals never watermarked).
- Photo coaching: Claude reviews uploads and surfaces non-blocking suggestions ("hallmark photo blurry," "add a back-view").

### Listing management
- Pieces have: title, type (necklace/bracelet/brooch/ring/ornament/magnet/pin), era (Victorian/Art Deco/Mid-Century/80s/etc.), condition (Mint/Excellent/Very Good/Good/Fair/As-Is) + condition notes, photos, description, **price**, **cost** (what Erica paid), **package weight + dimensions**, **physical location** ("Box 3, top tray"), and quantity (always 1 for vintage; configurable per design for ceramics).
- Single price across eBay, Etsy, and storefront (no per-platform overrides).
- Platform-calculated shipping based on buyer ZIP + package weight.
- US shipping only.
- Default returns policy: 30-day, buyer pays return shipping.
- Single-page add-piece form (no wizard); Claude drafts description and suggests price range from eBay sold-comps.
- Approve → 24-hour subscribers-only early access window → auto-publish to eBay + Etsy + public storefront.
- Edits in admin auto-sync to all platforms.
- Sold-status auto-removes from other platforms (and storefront) when one platform reports a sale; for ceramic batches, only when quantity hits 0.
- Order cancellation auto-revives the piece (re-list + alert).
- Standing buyer promotion: "buy 2+ get 10% off" managed by the system on eBay (Multi-buy) and Etsy (Sales and discounts).
- No-sale removal requires a reason tag (KEPT/GIFTED/BROKEN/LOST/PHOTO_DUPLICATE/OTHER); piece is archived in DB, not hard-deleted.
- Per-piece full audit log: every state change with timestamp.
- Publish-failure handling: auto-retry with exponential backoff; only persistent failure surfaces to admin.

### Description generation
- AI provider: Claude (Anthropic), Sonnet for normal use; Opus reserved for hard cases.
- Voice: matches 2–3 example listings Erica provides (style-reference flow). Feature is **gated on having voice samples**; do not ship a hardcoded tone.
- For vintage: Claude analyzes photos + Erica's notes, proposes era/maker/materials/condition cues, drafts the description.
- For ceramics: Claude rewrites Erica's facts (no research-from-photos needed; Erica is the source of truth).
- Owner always edits before publishing.
- Hard monthly Claude budget cap (default $25/mo, configurable); 80% warning email; cap blocks new draft requests when hit.

### Audience & email
- ESP: Mailchimp via Marketing API.
- Newsletter signup placements: footer, frequency-capped homepage modal, dedicated `/subscribe` page, inline form on each piece-detail page (pre-tagged by category/era).
- Compose blast modes (Erica picks per email): AI-drafted "new arrivals" shell, blank canvas with piece-picker, named templates ("New Arrivals," "Era Spotlight," "Sale Alert," "Behind the Studio").
- Email design: branded HTML matching the storefront, with plain-text fallback.
- Ad-hoc cadence (no auto-digest cron).
- Frequency cap: hard 1-blast-per-subscriber-per-7-days.
- Email sender domain: custom domain after launch, with SPF/DKIM/DMARC setup wizard.
- Post-purchase email capture, both compliant channels: (a) printed thank-you card with QR/short URL, (b) automated post-sale message via eBay/Etsy seller-to-buyer messaging APIs. **Never auto-add a buyer to the list without explicit opt-in.**
- Voice for emails: same voice samples as listings.

### Public storefront
- Mobile-first, vintage aesthetic (specific look picked from real mockups during build).
- Pages: home, piece grid (with filters), piece detail, sold archive (browseable, with SOLD watermark and CTA to current inventory), About Erica, FAQ, Contact, Subscribe, Privacy.
- Filters & search: by type, by era, by price range, plus free-text search.
- Recommendations: "you might also like" similar-piece module on detail pages + cookie-based personalized homepage rail.
- Buyer click on grid: quick-look modal on desktop, full piece-detail page on mobile (same URL shape).
- Per-piece "Ask about this piece" inquiry form with unchecked-by-default newsletter opt-in.
- "Save to Pinterest" button on every piece detail page.
- Cookie consent banner with manage-preferences flow.
- SEO basics: sitemap.xml, Open Graph + Twitter Cards, Product structured data on detail pages; sold pieces return 200 with availability=SoldOut.
- Footer: newsletter form + quick links (About/FAQ/Contact/Privacy) + platform/social links (Pinterest, Instagram, eBay store, Etsy store) + copyright & legal strip.
- Storefront does **not** show alt text in v1 (deferred — see Open risks).

### Admin app
- Mobile-friendly (so Erica can capture pieces from her phone at estate sales) but desktop is the primary work surface.
- Single user (Erica) — simple email/password (or passkey) auth.
- Landing page: "needs attention" dashboard with top-line stats (revenue, active listings, subscribers) over an action-oriented feed (drafts not yet published, slow-movers >60 days, recent sales, cancellations to investigate, post-purchase capture results).
- Sales analytics dashboard: total revenue (month/year/all time), sales feed (what sold, where, when, for how much), time-to-sell broken down by type and era, slow-mover list (>60 days unsold) — flagged only, no auto-discount.
- Profit/margin views: cost field per piece + estimated platform fees.
- Voice samples admin setting (paste 2–3 favorite listings).
- AI budget cap configurable.
- Per-platform watermark toggle.
- Bulk-discount threshold/percent configurable.

### Cross-cutting
- Database in the web app is the source of truth.
- Daily automated database backups, 30-day retention.
- Original photos kept indefinitely.
- Hosting budget: ~$20–50/month (managed services OK).
- Notifications: email for important events (sales, cancellations, slow-mover crossings, post-purchase opt-ins). No SMS.
- Scale target: 50–300 active listings, 10–30 new pieces per month.

---

## Out of v1 scope (deferred)

- AI-assisted image editing (auto-angles, generative cleanup, multi-angle synthesis).
- Importing existing eBay/Etsy listings.
- Admin bulk operations (bulk-publish, bulk-price-change, bulk-relist).
- Unified shipping queue / label printing in the admin (use eBay/Etsy native tools).
- Journal / blog section.
- International shipping.
- Custom-order requests.
- "Notify me when a similar piece comes in" capture on sold pages.
- Instagram cross-posting / auto-posting.
- Pinterest auto-pinning to a board (the per-piece "Save to Pinterest" button is in scope).
- Image alt text (deferred; see Open risks below).
- Per-platform pricing overrides.
- Buyer accounts on the storefront.
- Auto-drop-price on slow-movers (flagged-only is in scope).
- SMS notifications.

---

## Open dependencies (need to start now or before launch)

- **eBay Developer Program app** — Erica needs to register; OAuth flow setup. Long-pole item; can begin now.
- **Etsy app registration** — Erica needs to register and get reviewed. Long-pole item.
- **Mailchimp account** — free tier under 500 contacts is fine to start.
- **Domain purchase** — placeholder during dev; buy `elmvintage.com` (or chosen alternative) just before launch.
- **DNS for email sending** — SPF/DKIM/DMARC records on the chosen domain.
- **Voice samples** — 2–3 of Erica's favorite existing eBay/Etsy descriptions, pasted into the admin "voice samples" setting before the description-drafting feature ships.
- **eBay store name + Etsy shop name** — used for "View on eBay" / "View on Etsy" links and the platform OAuth setup.
- **Tech stack** — deferred until image-tool scope and listing-API patterns are firmer; pick before scaffolding.

---

## Open risks worth flagging

- **No image alt text in v1** — accessibility regression: screen-reader users cannot use the storefront, Google Images will rank pieces poorly without alts, and US ecommerce sites have been targeted by ADA accessibility lawsuits. The AI photo-critique infrastructure can generate alt text cheaply when re-enabled. Recommend revisiting before any meaningful public-launch push.
- **Mercari listings will go stale** — Erica is choosing to re-list everything in the new system rather than import; expect a transition period during which old eBay/Etsy listings (and any Mercari listings) remain live and need manual ending.
- **Subscribers' early-access expectations** — promising "24 hours early" creates a clock; the publish scheduler must be reliable. Plan for an admin override ("publish now, override the wait").
- **Mailchimp free tier ceiling** — 500 contacts is generous; once Erica crosses it, the pricing jumps. Plan a list-hygiene strategy (remove unengaged subscribers periodically).
- **eBay API approval timing** — registration can take days to weeks. If we wait until the build is done, we will be blocked. Start in parallel.

---

## v1 acceptance criteria

We can call v1 "done" when Erica can:

1. Photograph and upload a vintage piece from her phone.
2. See AI-drafted description and price suggestion based on the photos and her notes.
3. Edit the draft and approve.
4. See the piece appear in the subscribers-only early-access view.
5. See the piece auto-publish to eBay, Etsy, and the public storefront 24 hours later.
6. Receive an email when the piece sells on either platform.
7. See the other platform's listing automatically end and the piece move to the public sold archive.
8. Compose and send an email blast to her list (AI-drafted from new arrivals) and have it delivered without exceeding the frequency cap.
9. View profit, time-to-sell, and slow-mover analytics on the admin dashboard.
10. Have all of the above work on mobile.
