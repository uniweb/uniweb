# SSR/SSG Guide

This guide explains how to use Server-Side Rendering (SSR) and Static Site Generation (SSG) in the Uniweb framework using React Router 7.

## Overview

The `site-ssr` package provides:

- **SSR (Server-Side Rendering)**: Pages are rendered on the server at request time. Ideal for dynamic content, previews, and draft pages.
- **SSG (Static Site Generation)**: Pages are pre-rendered at build time to static HTML. Ideal for production sites with content that doesn't change frequently.

Both modes use the same codebase and components, with React hydration enabling client-side interactivity.

## Architecture

```
┌─────────────────────────────────────────────────────────────────┐
│                        Request Flow                              │
├─────────────────────────────────────────────────────────────────┤
│                                                                  │
│   Request → Route Loader → Content Parser → Page Renderer        │
│                              │                    │              │
│                              ▼                    ▼              │
│                         YAML/Markdown      Foundation            │
│                         (content/)         Components            │
│                              │                    │              │
│                              └────────┬──────────┘              │
│                                       ▼                          │
│                              Server-Rendered HTML                │
│                                       │                          │
│                                       ▼                          │
│                              Client Hydration                    │
│                                                                  │
└─────────────────────────────────────────────────────────────────┘
```

## Project Structure

```
packages/site-ssr/
├── app/
│   ├── content/
│   │   └── pages/
│   │       └── home/
│   │           ├── page.yml          # Page metadata
│   │           ├── 1-hero.md         # Section content
│   │           └── 2-features.md     # Section content
│   ├── lib/
│   │   ├── content.server.ts         # Content loader
│   │   └── renderer.tsx              # Page renderer
│   ├── routes/
│   │   └── page.tsx                  # Catch-all route
│   ├── routes.ts                     # Route configuration
│   └── root.tsx                      # App shell
├── react-router.config.ts            # SSR/SSG configuration
├── vite.config.ts                    # Vite configuration
└── package.json
```

## Content Format

### Page Metadata (`page.yml`)

```yaml
title: Home
description: Welcome to Uniweb
```

### Section Content (`1-hero.md`)

```markdown
---
component: Hero
title: Welcome to Uniweb
subtitle: Build modern websites with ease
ctaText: Learn More
ctaUrl: "#features"
---

Optional markdown body content goes here.
```

### Arrays in YAML

The content parser supports YAML arrays for components like Features:

```markdown
---
component: Features
title: Why Vite + ESM?
features:
  - title: Native ES Modules
    description: Standard browser modules
  - title: Vite Dev Server
    description: Lightning-fast HMR
---
```

## Configuration

### React Router Config (`react-router.config.ts`)

```typescript
import type { Config } from "@react-router/dev/config";
import { readdir } from "node:fs/promises";
import { join } from "node:path";

export default {
  ssr: true,
  appDirectory: "app",
  buildDirectory: "build",

  async prerender() {
    const pagesDir = join(process.cwd(), "app", "content", "pages");
    const entries = await readdir(pagesDir, { withFileTypes: true });

    return entries
      .filter((e) => e.isDirectory())
      .map((e) => (e.name === "home" ? "/" : `/${e.name}`));
  },
} satisfies Config;
```

### Vite Config (`vite.config.ts`)

```typescript
import { reactRouter } from "@react-router/dev/vite";
import { defineConfig } from "vite";
import tsconfigPaths from "vite-tsconfig-paths";

export default defineConfig({
  plugins: [reactRouter(), tsconfigPaths()],
  server: {
    port: 3002,
  },
  optimizeDeps: {
    include: ["foundation-example"],
  },
});
```

## Usage

### Development (SSR Mode)

```bash
cd packages/site-ssr
pnpm dev
```

The dev server renders pages on the server at request time, providing:

- Hot Module Replacement (HMR)
- Instant feedback on content changes
- Full SSR with hydration

### Production Build (SSG Mode)

```bash
cd packages/site-ssr
pnpm build
```

This pre-renders all pages defined in `prerender()` to static HTML files:

```
build/
├── client/
│   ├── index.html          # Pre-rendered home page
│   ├── about/index.html    # Pre-rendered about page (if exists)
│   └── assets/             # JavaScript, CSS bundles
└── server/
    └── index.js            # Server bundle (for SSR fallback)
```

### Serving the Built Site

```bash
# Serve with SSR fallback for non-prerendered routes
pnpm start

# Or serve static files only (no SSR)
npx serve build/client
```

## Route Configuration

The `routes.ts` file defines a single catch-all route:

```typescript
import { type RouteConfig, route } from "@react-router/dev/routes";

export default [route("*?", "routes/page.tsx")] satisfies RouteConfig;
```

The `*?` pattern matches:

- `/` (home)
- `/about`
- `/blog/post-1`
- Any other path

## Loader Pattern

The page route uses a loader to fetch content:

```typescript
// routes/page.tsx
import { loadPageContent } from "~/lib/content.server";
import { PageRenderer } from "~/lib/renderer";
import type { Route } from "./+types/page";

export async function loader({ request }: Route.LoaderArgs) {
  const url = new URL(request.url);
  const path = url.pathname === "/" ? "/home" : url.pathname;
  const pageName = path.slice(1);

  const content = await loadPageContent(pageName);

  if (!content) {
    throw new Response("Page not found", { status: 404 });
  }

  return { page: content };
}

export default function Page({ loaderData }: Route.ComponentProps) {
  return <PageRenderer page={loaderData.page} />;
}
```

## Page Renderer

The renderer maps sections to foundation components:

```typescript
// lib/renderer.tsx
import * as foundation from "foundation-example";

function SectionRenderer({ section }: { section: PageSection }) {
  const Component = foundation.getComponent(section.component);

  if (!Component) {
    return <div>Unknown component: {section.component}</div>;
  }

  const content = { ...section.params, _body: section.content };
  return <Component content={content} />;
}

export function PageRenderer({ page }: PageRendererProps) {
  return (
    <main>
      {page.sections.map((section, index) => (
        <SectionRenderer key={section.id || index} section={section} />
      ))}
    </main>
  );
}
```

## Adding New Pages

1. Create a new directory in `app/content/pages/`:

```bash
mkdir app/content/pages/about
```

2. Add page metadata:

```yaml
# app/content/pages/about/page.yml
title: About Us
description: Learn more about our company
```

3. Add section content:

```markdown
# app/content/pages/about/1-hero.md

---
component: Hero
title: About Us
subtitle: Our story and mission
---
```

4. The page is automatically available at `/about`

## SSR vs SSG Decision Guide

| Use SSR When                   | Use SSG When                   |
| ------------------------------ | ------------------------------ |
| Content changes frequently     | Content is mostly static       |
| Personalized content needed    | Same content for all users     |
| Draft/preview functionality    | Production published sites     |
| Real-time data required        | Performance is critical        |

## Hybrid Approach

React Router 7 supports a hybrid approach:

- Pre-render known static pages with `prerender()`
- Fall back to SSR for dynamic or new pages
- The server bundle handles routes not pre-rendered

This gives the best of both worlds: fast static pages where possible, with dynamic SSR as a fallback.
