# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Project

Marketing website for LeftClick, an AI automation agency. Four static HTML pages, no build step.

- **Live**: https://leftclick-agency.netlify.app

## Commands

```bash
# Open locally (no server needed)
open index.html

# Deploy preview
netlify deploy

# Deploy to production
netlify deploy --prod

# Take a screenshot for visual iteration
python3 -c "
from playwright.sync_api import sync_playwright
with sync_playwright() as p:
    b = p.chromium.launch()
    page = b.new_page(viewport={'width': 1280, 'height': 900})
    page.goto('file:///path/to/index.html')
    page.wait_for_timeout(1500)
    page.evaluate(\"document.querySelectorAll('.fade').forEach(e => e.classList.add('in'))\")
    page.screenshot(path='shot.png', full_page=True)
    b.close()
"
```

## Architecture

**Single-file per page** — all CSS lives in a `<style>` block and all JS in a `<script>` block at the bottom of each HTML file. No external stylesheets, no bundler, no npm.

### CSS token system
All design tokens are CSS custom properties on `:root` at the top of each file's `<style>` block:
```css
--bg, --white, --dark, --text, --muted, --faint
--green, --green-dk
--border, --sh, --sh-lg
```
Always use these variables rather than raw hex values.

### Layout pattern
Content sections use a `.wrap` div (`max-width: 1100px; margin: 0 auto`) for centering. Section padding is consistently `100px 40px`. Responsive breakpoints are at `900px` and `540px` only.

### Animation pattern
Scroll-triggered fade-ins use the `.fade` / `.fade.in` class pair driven by a single `IntersectionObserver` instance at the bottom of each page. Add `class="fade"` to any element that should animate in; `threshold: 0.08` triggers it. When screenshotting, force-add `.in` to all `.fade` elements or they render invisible.

### Scroll progress bar
`#spb` is a fixed 2px green bar at the top of the viewport, width driven by a `scroll` event listener.

### pages
- `index.html` — landing page (646 lines); hero with three floating angled `.fcard` elements, ticker, tagline, stacked `.ccard` case-study cards with a `.sticky` note overlay, pricing split, big statement, two-panel visuals, services grid, dark CTA
- `services.html`, `about.html`, `contact.html` — not yet built; nav links reference them

### .claude/ structure
Detailed rules, skills, and agents live in `.claude/` and are auto-loaded by Claude Code:
- `rules/` — design system, page structure, git workflow (loaded every session)
- `skills/deploy`, `skills/new-page` — invokable via `/deploy`, `/new-page`
- `agents/design-reviewer`, `agents/html-dev` — specialized subagents
