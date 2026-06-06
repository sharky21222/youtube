# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Project Overview

A single-page Russian-language donation/order form for YouTube video promotion. Users enter their name, a YouTube video URL, and select a donation amount; the app generates a unique rouble amount (with random kopeck cents) for Tinkoff Bank (T-BANK) transfer and persists the order via the JSONBin.io API.

## Architecture

The entire application lives in a single file: **`index.html`** (~530 lines). There is no build step, no bundler, and no external JavaScript dependencies.

The file is structured in three sequential sections:

1. **CSS** (lines 8–320): Embedded in `<style>`. Dark glass-morphism UI with a mobile breakpoint at 480px. Tinkoff Bank yellow `#FFDD2D` is the accent colour.

2. **HTML** (lines 325–382): Two `<div>` sections toggled by JS — `#orderSection` (the input form) and `#resultSection` (the generated payment details). The form has fields for name, YouTube URL, and amount buttons (100 / 200 / 500 / 1000 ₽).

3. **JavaScript** (lines 385–524): Embedded in `<script>`. Key globals:
   - `JSONBIN_BIN_ID` / `JSONBIN_API_KEY` — credentials for the JSONBin.io REST API (currently hardcoded in source).
   - `selectedAmount` / `generatedAmount` — application state.
   - `generateUniqueAmount(base)` — appends a random 01–99 kopeck suffix and checks existing orders to guarantee uniqueness.
   - `extractVideoId(url)` — handles `youtube.com/watch?v=`, `youtu.be/`, `youtube.com/shorts/`, and raw 11-char IDs.
   - `loadOrders()` / `saveOrders(orders)` — GET/PUT to JSONBin.io.

## Development

No build, lint, or test tooling is configured. To develop locally, serve `index.html` with any static file server, for example:

```bash
npx serve .
# or
python3 -m http.server 8080
```

## Deployment

Deployed on **Vercel**. `vercel.json` rewrites `/order` and all other paths to `index.html`.

## Key Constraints

- **JSONBin API key is exposed client-side.** Any change that touches `JSONBIN_API_KEY` or introduces new secret values must not embed them in the HTML — route them through a Vercel serverless function or environment variable instead.
- **All UI text is in Russian (Cyrillic).** Keep user-facing strings in Russian.
- **No framework.** Do not introduce npm packages or a bundler without first adding a build step to `package.json`.
- The unique-amount algorithm depends on kopeck granularity (two decimal places). Tinkoff Bank uses this to match incoming transfers to orders; never round or truncate the generated amount.
