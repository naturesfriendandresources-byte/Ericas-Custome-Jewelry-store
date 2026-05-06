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
- **Scale target**: medium — 50–300 active listings, 10–30 new pieces per month. Design for this; don't over-engineer for thousands.
- **Hosting budget**: ~$20–50/month, so paid managed services (managed Postgres, object storage, always-on workers) are acceptable; we are not constrained to free tiers.
- **Seller accounts**: Erica has active seller accounts on both **eBay** and **Etsy**, but does **not** yet have developer/API access on either. Registering an eBay Developer Program app and an Etsy app (with OAuth) is a prerequisite for any syndication work.
- **Tech stack**: deferred. Do not pick a language/framework yet; revisit once image-tool scope and listing-API integrations are clearer.

### Future platforms to evaluate (still gated on the API-only rule)

- **Facebook / Instagram Shopping** — Meta Commerce / Catalog API exists; could be in scope later.
- **Amazon Handmade** — has APIs, but requires application/approval.

## Working in this repo

- Before writing code, confirm any still-undecided items above with the user (especially listing-data storage and the tech stack).
- There is no `package.json`, `requirements.txt`, `Makefile`, or equivalent — when one is added, update this file with the actual build / test / lint / run commands.
- The active development branch (per task instructions when this file was created) is `claude/add-claude-documentation-vl0eJ`. The default branch is `main`.
