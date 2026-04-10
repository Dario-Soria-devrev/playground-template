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
│   ├── customers/
│   │   ├── fintech.json    # Fintech prospect config
│   │   └── example.json    # Blank template / reference config
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

1. Create a new file at `public/customers/<slug>.json` (the slug becomes the URL path).
2. Fill in the fields — see the schema below.
3. Commit and push to `main`. Vercel deploys automatically.
4. Share the URL: `https://your-domain.com/<slug>`

### Customer JSON Schema

```jsonc
{
  // Display name shown in the hero heading
  "customer": "Acme Corp",

  // DevRev PLuG widget app ID (from your DevRev org settings)
  "widgetAppId": "YOUR_PLUG_APP_ID",

  // Hero section subtitle
  "heroSubtitle": "AI-powered support tailored for your needs.",

  // Note shown inside the "How This Works" card
  "howItWorksNote": "Click the chat widget in the bottom right to try it out.",

  // Sample questions — each group becomes one lavender card
  "questions": [
    {
      "title": "General Support",
      "items": [
        "How do I get started?",
        "How do I submit a ticket?"
      ]
    },
    {
      "title": "Billing",
      "items": [
        "How do I update my billing information?",
        "How do I cancel my subscription?"
      ]
    }
  ],

  // Challenge cards — each becomes a "COMPUTER FOR" lavender card
  "problems": [
    {
      "title": "Ticket Overload",
      "description": "50–150 tickets per day with only 4 technicians.",
      "stat": "25x above industry standard",
      "icon": "⚡"
    }
  ],

  // Demo video (YouTube URL + thumbnail)
  "videoUrl": "https://youtu.be/VIDEO_ID",
  "videoThumbnail": "https://img.youtube.com/vi/VIDEO_ID/hqdefault.jpg",

  // "Learn More" link in the Resources section
  "learnMoreUrl": "https://devrev.ai/support",

  // Contact details for the "Book a Demo" CTA
  "contactEmail": "your.name@devrev.ai",
  "contactWhatsApp": "15551234567"   // digits only, no + or spaces
}
```

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
- **Clickable challenge cards** — clicking a card opens PLuG with "How can DevRev help me with [card title]?"

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
