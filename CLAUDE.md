# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Assistant identity

When interacting with the owner in this project, the assistant should refer to itself as **Linda**.

## Repository status

This repository is currently a greenfield project — it contains only `README.md` and has no source code, build tooling, tests, dependencies, or CI configuration. Do not assume a stack (framework, language, package manager) has been chosen; ask before scaffolding one.

## Project intent (from README.md)

The owner ("Erica") plans to sell custom jewelry — rings, bracelets, necklaces, brooches, etc. The system being built is expected to support:

1. **Image processing** — owner uploads photos of pieces; the system modifies them to produce good angles / listing-quality images.
2. **Multi-platform listing management** — the system manages listings across multiple selling platforms.
3. **Description generation** — for each piece, the system researches the item and generates a sales description.

Treat these three capabilities as the product surface when discussing architecture or proposing changes.

## Decisions made so far

- **Form factor**: a web app / web store that Erica owns (not a desktop or mobile app).
- **Listing destinations**: a web store we build, plus syndication to **eBay** and **Mercari**. Additional platforms under consideration: **Etsy** (strong fit, has a public API), **Poshmark**, **Depop**, **Facebook Marketplace / Instagram Shopping**, **Amazon Handmade**.
- **Image processing v1**: simpler tools only — crop, rotate, color/exposure adjust, background cleanup. AI-assisted editing (auto-angles, generative cleanup) is planned for a later version, not v1.
- **Tech stack**: deferred. Do not pick a language/framework yet; revisit once image-tool scope and listing-API integrations are clearer.
- **Listing data storage**: undecided — owner needs more information before choosing. Do not assume a database, spreadsheet, or per-platform-API-as-source approach until this is settled.

### Platform API caveats (important)

- **eBay** and **Etsy** have official listing APIs.
- **Mercari** does **not** have a public seller API. Any Mercari integration will require browser automation or a third-party paid service — flag this trade-off before building.
- **Poshmark** also lacks a public listing API.

## Working in this repo

- Before writing code, confirm any still-undecided items above with the user (especially listing-data storage and the tech stack).
- There is no `package.json`, `requirements.txt`, `Makefile`, or equivalent — when one is added, update this file with the actual build / test / lint / run commands.
- The active development branch (per task instructions when this file was created) is `claude/add-claude-documentation-vl0eJ`. The default branch is `main`.
