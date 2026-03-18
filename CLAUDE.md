# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Build & Development Commands

```bash
npm start                # Start dev server (gatsby develop)
npm run build            # Build static site to /public
npm run deploy           # Build and deploy to gh-pages branch
npm run clean            # Clean Gatsby cache (useful when builds fail)
npm run serve            # Serve production build locally
npm run format           # Format with Prettier
```

**Setup:** Requires Node v18. Install with `npm install --legacy-peer-deps` (required due to peer dependency conflicts).

**Pre-commit hook:** Husky runs lint-staged on JS/CSS/JSON/MD files.

## Architecture

**Gatsby v3 static site** — a personal portfolio with blog, using React 17, styled-components, and Markdown content.

### Page Structure

- `src/pages/index.js` — Single-page homepage rendering section components (Hero, About, Jobs, Featured, Projects, Contact) inside a Layout wrapper
- `src/pages/archive.js` — Blog listing page
- `src/pages/pensieve/` — Blog/notes section
- `src/templates/post.js` and `src/templates/tag.js` — Dynamic pages created via `gatsby-node.js` from Markdown content

### Content Pipeline

Markdown files in `/content/` (posts, featured, jobs, projects) are transformed by `gatsby-transformer-remark` into GraphQL nodes. `gatsby-node.js` creates blog post and tag pages dynamically.

### Key Configuration

- `src/config.js` — Site-wide config: colors, nav links, social media, email, ScrollReveal settings
- `gatsby-config.js` — Gatsby plugins, site metadata, filesystem sources
- `gatsby-node.js` — Dynamic page creation + webpack path aliases (`@components`, `@config`, `@styles`, `@hooks`, `@utils`, etc.)

### Styling

- **Styled-components** for component-scoped CSS
- `src/styles/variables.js` — CSS custom properties (colors, font sizes, spacing)
- `src/styles/mixins.js` — Reusable styled-components mixins (`flexCenter`, `flexBetween`, button variants, `inlineLink`)
- `src/styles/theme.js` — Breakpoints for responsive design
- Custom fonts (Calibre, SF Mono) loaded via `src/styles/fonts.js`

### Animations

- **ScrollReveal** for on-scroll reveal effects (wrapped in `src/utils/sr.js`)
- **React Transition Group** (CSSTransition) for entrance animations
- **Anime.js** for the logo loader animation
- `usePrefersReducedMotion` hook respects user accessibility preferences

### Custom Hooks

- `useScrollDirection` — Detect scroll up/down for nav hide/show
- `usePrefersReducedMotion` — Respect reduced-motion OS setting
- `useOnClickOutside` — Close mobile menu on outside click

### Icon System

SVG icons are React components in `src/components/icons/`. The base `Icon` component (`icon.js`) renders by name string lookup.
