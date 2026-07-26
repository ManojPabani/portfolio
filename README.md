# Manoj Kumar — Portfolio

A premium, single-page portfolio website built with hand-crafted HTML5, CSS3, and
vanilla JavaScript (ES6) — no frameworks, no build step, no runtime Markdown parsing.

## Project Structure

```
portfolio/
│
├── index.html              # All markup and content (sourced from the .md files below)
├── css/
│   └── styles.css          # Design tokens, layout, components, responsive + print styles
├── js/
│   └── app.js               # Theme toggle, nav, scroll effects, counters, copy-to-clipboard
├── assets/
│   ├── avatar.svg            # Hero developer-workspace illustration
│   ├── favicon.svg           # Browser tab icon
│   └── icons/                 # Standalone reference copies of the SVG icons used inline
├── experience.md            # Source material only — not loaded at runtime
├── projects.md               # Source material only — not loaded at runtime
├── skills.md                  # Source material only — not loaded at runtime
├── education.md              # Source material only — not loaded at runtime
├── certifications.md         # Source material only — not loaded at runtime
├── contact.md                 # Source material only — not loaded at runtime
└── README.md
```

The six `.md` files are kept in the project purely as source-of-truth reference
material. Their content has already been extracted and embedded directly into
`index.html` — the site does **not** fetch, parse, or render Markdown at runtime.

## Running Locally

No build tools or dependencies are required. Any static file server works, e.g.:

```bash
npx serve .
# or
python -m http.server 8080
```

Then open `http://localhost:8080` (or the printed URL) in your browser.
Opening `index.html` directly via `file://` also works.

## Adding Your Resume

The hero "Download Resume" button links to `assets/resume.pdf`. Add your resume PDF
at that path (`portfolio/assets/resume.pdf`) to enable the download — the button is
already wired up in `index.html`.

## Features

- Sticky, responsive navigation with active-section highlighting and a mobile hamburger menu
- Dark theme by default with a persisted light/dark toggle (`localStorage`)
- Scroll progress indicator, scroll-to-top button, and scroll-triggered fade-in reveals
- Animated stat counters (Years of Experience, Enterprise Projects, Certifications)
- Copy-to-clipboard email button
- Semantic HTML, ARIA labels, visible focus states, and print-friendly resume styling
- Fully responsive from mobile through desktop, max content width of 1200px

## Deployment

The project is entirely static and deploys as-is to GitHub Pages, Vercel, Netlify,
or any static host. Point the host at `portfolio/index.html` as the entry file.
