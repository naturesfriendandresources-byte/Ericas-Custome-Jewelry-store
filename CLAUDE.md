# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Repository status

This repository is currently a greenfield project — it contains only `README.md` and has no source code, build tooling, tests, dependencies, or CI configuration. Do not assume a stack (framework, language, package manager) has been chosen; ask before scaffolding one.

## Project intent (from README.md)

The owner ("Erica") plans to sell custom jewelry — rings, bracelets, necklaces, brooches, etc. The system being built is expected to support:

1. **Image processing** — owner uploads photos of pieces; the system modifies them to produce good angles / listing-quality images.
2. **Multi-platform listing management** — the system manages listings across multiple selling platforms (specific platforms not yet specified).
3. **Description generation** — for each piece, the system researches the item and generates a sales description.

Treat these three capabilities as the product surface when discussing architecture or proposing changes.

## Working in this repo

- Before writing code, confirm with the user: target platform(s) (web, mobile, desktop), language/framework, where listings will be published (eBay, Etsy, Shopify, Poshmark, etc.), and how images will be processed (local library, hosted API, generative model).
- There is no `package.json`, `requirements.txt`, `Makefile`, or equivalent — when one is added, update this file with the actual build / test / lint / run commands.
- The active development branch (per task instructions when this file was created) is `claude/add-claude-documentation-vl0eJ`. The default branch is `main`.
