# Merrick Demo — Powered by Computer, by DevRev

A customer-facing demo site for the DevRev **Computer** product. Each prospect gets their own branded URL (e.g. `/fintech`, `/acme`) that loads a tailored experience: custom headline, sample questions, challenge cards, a demo video, and a live AI chat widget — all in a single static site with zero build step.

---

## Live URLs

| Environment | URL |
|---|---|
| Production | `https://playground-template-nine.vercel.app/<customer-slug>` |
| Local dev | `http://localhost:8080/<customer-slug>` |

Navigate to a customer slug to see a fully populated demo. The root path `/` renders a blank template state.

---

## How It Works

```
Browser loads /<customer-slug>
        │
        ▼
index.html served (same file for all routes)
        │
        ▼
JS reads window.location.pathname → "fintech"
        │
        ▼
fetch /customers/fintech.json
        │
        ▼
Populates: hero title, subtitle, questions,
           challenge cards, video, contact links
        │
        ▼
Initialises DevRev PLuG chat widget
with widgetAppId from the JSON
```

---

## Project Structure

```
merrick-demo/
├── public/
│   ├── index.html          # The entire site — HTML, CSS, JS in one file
│   ├── favicon.webp        # Site favicon + nav logo
│   ├── computer.svg        # Original Computer product icon (unused in nav)
│   ├── assets/             # Local card/preview images used by customer configs
│   ├── customers/
│   │   ├── black-duck.json # Current default template reference
│   │   ├── bycoders.json   # Live customer config
│   │   ├── fintech.json    # Financial Services config
│   │   ├── template.json   # Generic starter template (copy this for new pages)
│   │   └── template-blackduck.json # Black-Duck-style starter template
│   ├── 404.html            # Custom 404 page
│   └── 50x.html            # Custom 5xx page
├── settings/
│   └── config.toml         # Static Web Server config (port 8080)
├── wasmer.toml             # Wasmer Edge deployment manifest
├── Staticfile              # Declares public/ as document root
└── README.md
```

---

## Adding a New Customer

1. Copy the default template:
   - `public/customers/template-blackduck.json` (recommended)
   - or `public/customers/template.json`
2. Save as `public/customers/<slug>.json` (the slug becomes the URL path).
3. Fill in the customer-specific fields — see the schema below.
4. Add any referenced local images to `public/assets/`.
5. Commit and push to `main`. Vercel deploys automatically.
6. Share the URL: `https://your-domain.com/<slug>`

### Default Template Policy

New pages now default to the **Black Duck template structure**:
- 2 question groups
- 4 use-case cards in the challenge section
- optional "Try Computer Live" panel below questions
- 4 customer story cards in resources
- optional non-clickable use-case cards via `disableCardClicks`

### Customer JSON Schema (Black Duck template)

```jsonc
{
  "customer": "Acme Corp",
  "agentLabel": "CX Agent",
  "disableCardClicks": true,
  "widgetAppId": "YOUR_PLUG_APP_ID",
  "heroSubtitle": "AI-powered support tailored for your needs.",
  "howItWorksNote": "Click the chat widget in the bottom right to try it out.",

  "questions": [
    { "title": "Group 1", "items": ["Question 1", "Question 2"] },
    { "title": "Group 2", "items": ["Question 1", "Question 2"] }
  ],

  "problems": [
    {
      "title": "Use Case Title",
      "description": "Use-case copy shown in the challenge section.",
      "stat": "Short stat/value line",
      "icon": "🤖"
    }
  ],

  "livePanel": {
    "title": "Try Computer Live",
    "subtitle": "Short explanatory copy",
    "ctaLabel": "LAUNCH EX AGENT",
    "url": "https://app.devrev.ai/<demo>/computer",
    "mockPreviewImage": "/assets/preview.png"
  },

  "stories": [
    {
      "title": "Customer story title",
      "image": "/assets/story.png",
      "url": "https://devrev.ai/customers/example"
    }
  ],

  "videoUrl": "https://youtu.be/VIDEO_ID",
  "videoThumbnail": "https://img.youtube.com/vi/VIDEO_ID/hqdefault.jpg",
  "learnMoreUrl": "https://devrev.ai/support",
  "contactEmail": "your.name@devrev.ai",
  "contactWhatsApp": "15551234567"
}
```

### Legacy Notes

- `problems` can still use the richer 3-section format:
  - `pain`, `successCriteria[]`, `proof`
- If `livePanel` is omitted, no panel is rendered.
- If `stories` is omitted, only Demo Video + Learn More are shown.

---

## Design System

The site uses DevRev's brand design language throughout.

### Colors

| Token | Value | Used for |
|---|---|---|
| `--yellow-bright` | `#FFE000` | Hero background, nav at top, CTA section |
| `--yellow-mid` | `#FFF5B0` | "How This Works" section |
| `--yellow-pale` | `#FFFDE0` | "Sample Questions" section |
| `--yellow-faint` | `#FEFEF5` | "The Challenge" section |
| `--lavender` | `#EDE6FF` | All cards (questions + challenge) |
| `--indigo-title` | `#3730A3` | Card titles, links, stat badges |
| `--indigo-label` | `#5B21B6` | "COMPUTER FOR" micro-labels |
| `--text-dark` | `#111111` | Primary text on yellow/white |
| `--text-body` | `#333333` | Body copy |

### Typography

| Element | Font | Weight |
|---|---|---|
| Headings (`h1`, `h2`, card titles) | Bricolage Grotesque | 800 |
| Body text, nav, subtitles | Inter | 400 / 500 |
| Stat badges | JetBrains Mono | 400 |

### Key UI Behaviours

- **Nav hide-on-scroll** — the navigation bar slides upward when scrolling down and reappears when scrolling up.
- **Nav colour transition** — yellow at the top of the page; switches to frosted white once the hero is scrolled past.
- **Clickable questions** — clicking any sample question opens the PLuG chat widget and pre-fills that question.
- **Challenge card click behavior** — controlled by `disableCardClicks` per customer config.
- **Live panel** — optional `livePanel` can render a centered "Try Computer Live" panel below questions.

---

## Running Locally

### With Wasmer (production-equivalent)
```bash
wasmer run . --net
# → http://localhost:8080/<customer-slug>
```

### With Node (npx serve)
```bash
npx serve public --single --listen 8080
# → http://localhost:8080/<customer-slug>
```

The `--single` flag is required so that unknown paths (e.g. `/fintech`) fall back to `index.html` instead of returning a 404.

---

## Deployment

The site is deployed on **Vercel** as a static site. Pushing to `main` triggers an automatic deployment — no build step required. Files in `public/` are served directly.

The `wasmer.toml` manifest also supports deployment to **Wasmer Edge** using the `wasmer/static-web-server` package.

---

## Editing the Site

All styling and layout lives in the `<style>` block inside `public/index.html`. There are no external CSS files, no bundlers, and no frameworks — edit the file directly.

Responsive breakpoints:
- `968px` — collapses two-column grids to single column
- `640px` — hides nav links, reduces font sizes
