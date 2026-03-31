# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Project Overview

Static demo/landing page for a Merrick × DevRev service desk transformation pitch. Single-page site showcasing AI-powered ticket deflection, automated license provisioning, and approval routing scenarios.

## Architecture

- **`public/`** — All static assets served to the browser
  - `index.html` — The entire site (HTML + inline CSS, no JS framework)
  - `computer.svg` — Logo asset
  - `404.html` / `50x.html` — Error pages
- **`settings/config.toml`** — Static Web Server configuration (port 8080, compression, caching)
- **`wasmer.toml`** — Wasmer Edge deployment manifest; uses `wasmer/static-web-server`
- **`Staticfile`** — Declares `public` as the document root
- **`Autodesk_AutoCAD.zip`** — Downloadable asset linked from the demo page

## Deployment

Deployed as a static site on **Wasmer Edge**. No build step — files in `public/` are served directly.

To run locally with Wasmer:
```
wasmer run . --net
```
The server starts on port 8080.

## Key Details

- No build tools, bundlers, or package managers — edit `public/index.html` directly.
- Fonts loaded from Google Fonts CDN (Space Grotesk, JetBrains Mono).
- All styling is inline in `index.html` via a `<style>` block; no external CSS files.
- Responsive breakpoints at 968px and 640px.
