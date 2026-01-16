# Static Site Generation (SSG)

Uniweb supports Static Site Generation (SSG) - pre-rendering your pages to static HTML at build time. This provides faster initial page loads and better SEO while maintaining full React interactivity.

## Overview

| Mode | Description | Use Case |
|------|-------------|----------|
| **SPA** (default) | Single-page app, renders in browser | Admin panels, apps, draft previews |
| **SSG** | Pre-rendered static HTML + hydration | Production sites, SEO-critical pages |

## Quick Start

```bash
# Standard build (SPA)
cd site
pnpm build

# Build with pre-rendering (SSG)
uniweb build --prerender
```

## How It Works

### Standard Build (SPA)

```
Request → index.html (shell) → JavaScript loads → React renders page
```

The browser receives an empty HTML shell, then JavaScript renders the content. Search engines may not see the full content.

### SSG Build

```
Build time: Each page → React renderToString() → Static HTML file

Request → Pre-rendered HTML (full content) → JavaScript hydrates → React takes over
```

Users see content immediately. Search engines see the full page. React then "hydrates" the HTML to enable interactivity.

## Output Structure

**Standard Build:**
```
dist/
├── index.html          # Empty shell for all routes
├── site-content.json   # Page data loaded by JS
└── assets/
    └── *.js, *.css
```

**SSG Build:**
```
dist/
├── index.html          # Pre-rendered home page
├── about/
│   └── index.html      # Pre-rendered /about
├── pricing/
│   └── index.html      # Pre-rendered /pricing
├── site-content.json   # Still included for hydration
└── assets/
    └── *.js, *.css     # Still included for hydration
```

## What Gets Pre-rendered

Each page in your `pages/` directory becomes a static HTML file:

| Page Directory | Route | Output File |
|---------------|-------|-------------|
| `pages/home/` | `/` | `dist/index.html` |
| `pages/about/` | `/about` | `dist/about/index.html` |
| `pages/pricing/` | `/pricing` | `dist/pricing/index.html` |
| `pages/blog/intro/` | `/blog/intro` | `dist/blog/intro/index.html` |

## HTML Enhancements

Pre-rendered pages include:

1. **Full HTML content** - All components rendered to markup
2. **Page title** - From `page.yml` `title` field
3. **Meta description** - From `page.yml` `description` field
4. **Hydration data** - `__SITE_CONTENT__` script tag for React

Example pre-rendered HTML:
```html
<!DOCTYPE html>
<html lang="en">
<head>
  <title>About Us</title>
  <meta name="description" content="Learn more about our company">
  <script id="__SITE_CONTENT__" type="application/json">
    {"pages":[...],"config":{...}}
  </script>
  <script type="module" src="/assets/index.js"></script>
</head>
<body>
  <div id="root">
    <main>
      <div id="Section1" class="context__dark">
        <section class="hero">
          <h1>About Us</h1>
          <p>We build amazing things.</p>
        </section>
      </div>
    </main>
  </div>
</body>
</html>
```

## CLI Options

```bash
uniweb build --prerender [options]
```

| Option | Description |
|--------|-------------|
| `--prerender` | Enable SSG pre-rendering |
| `--foundation-dir <path>` | Custom foundation path (default: `../foundation`) |

## When to Use SSG

**Use SSG when:**
- SEO matters (marketing sites, blogs, landing pages)
- Fast initial page load is important
- Content doesn't change per-user
- You want to deploy to static hosting (Vercel, Netlify, GitHub Pages)

**Use SPA when:**
- Building admin panels or dashboards
- Content is user-specific
- You need draft/preview functionality
- Pages have real-time data

## Hydration

After the pre-rendered HTML loads, React "hydrates" the page:

1. Browser receives pre-rendered HTML (content visible immediately)
2. JavaScript bundle loads
3. React attaches event handlers to existing HTML
4. Page becomes fully interactive

This gives you the best of both worlds: fast initial render + full React functionality.

## Deployment

Pre-rendered sites work with any static hosting:

```bash
# Build with SSG
uniweb build --prerender

# Deploy dist/ folder to:
# - Vercel
# - Netlify
# - GitHub Pages
# - AWS S3 + CloudFront
# - Any static file server
```

## Limitations

- **Build-time only** - Content is rendered at build time, not request time
- **No per-request data** - All users see the same pre-rendered content
- **Rebuild for changes** - Content changes require a new build

For dynamic content that changes per-request, consider keeping those routes as SPA while pre-rendering static pages.

## Future: Server-Side Rendering (SSR)

SSR (rendering at request time) is planned for future releases. This will enable:
- Per-request dynamic content
- Draft/preview modes
- Personalized pages

SSG covers most production use cases. SSR will be added when the need arises.
