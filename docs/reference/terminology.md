# Uniweb Terminology Reference

This reference clarifies how Uniweb Framework uses key terms, many of which have different meanings in other frameworks or contexts.

## Core Architecture Terms

### Foundation

**In Uniweb:** A comprehensive collection of React components designed to work together as a cohesive design system. Foundations have a content-facing interface—they're designed to be referenced declaratively by content creators through markdown frontmatter and visual editors.

**Different from:** Generic component libraries where individual components are imported separately by developers in code. Traditional component libraries have developer-facing interfaces (JavaScript APIs, props, imports), while Foundations add a content-facing interface layer.

**Examples:**

- A marketing Foundation with hero sections, feature grids, and testimonial components
- A documentation Foundation with article layouts, code blocks, and navigation
- A corporate Foundation with team profiles, service listings, and contact forms

### Module

**In Uniweb:** The technical packaging and delivery mechanism that connects a Foundation to a site at runtime. Foundations are built as ES modules that can be loaded dynamically via `import()`, with shared dependencies (React) resolved through browser import maps.

**Different from:** Traditional bundled applications where components are compiled into the site. Uniweb keeps Foundations separate, loaded at runtime.

**Technical detail:** Foundations output `foundation.js` (bundled components) and `schema.json` (component metadata). The browser's import map ensures all modules share the same React instance:

```html
<script type="importmap">
  { "imports": { "react": "https://esm.sh/react@18" } }
</script>
```

**Distribution modes:**
- **Local workspace** — Foundation as npm dependency, full HMR during development
- **Runtime loading** — Foundation loaded via URL and import maps (production)
- **Dynamic backend** — Platform injects foundation URL and content (uniweb.app)

### Site

**In Uniweb:** An independent website with its own content, configuration, and chosen Foundation. Sites are primarily content—pages written in markdown, organized in a directory structure, with metadata defining their Foundation connection.

**Different from:** Template-based sites in traditional CMSs where content and presentation are tightly coupled. In Uniweb, sites are pure content that connects to Foundations at runtime.

**Examples:**

- A company marketing site using a marketing Foundation
- A product documentation site using a docs Foundation
- A personal blog using a blogging Foundation

### Project

**In Uniweb:** A repository with one or many sites and/or Foundations, each managed as an npm/yarn/pnpm workspace.

**Different from:** Single-app projects in traditional frameworks. Uniweb projects are designed to manage multiple related but independent workspaces.

**Common patterns:**

- Multi-workspace: Multiple sites and Foundations in one repository
- Content-only: Just sites, using remote Foundations
- Development: One Foundation being developed with test sites

### Registry

**In Uniweb:** A catalog of licensed Foundations, each representing a complete design system that sites can connect to. The registry enables Foundation distribution and versioning.

**Different from:** Component marketplaces where individual components are mixed and matched. Uniweb registries list proprietary Foundations that are designed to work as cohesive systems.

**Optional:** Foundations can also be distributed via direct URLs or local development.

## Content Structure Terms

Uniweb has two distinct content types: **page content** (sections) and **reusable content** (collections).

### Page

**In Uniweb:** A folder whose path maps to a URL path, containing markdown files (sections) and a `page.yml` file defining page properties. Pages organize sections and define metadata like title, description, and layout.

**Different from:** Template files in traditional systems where the page structure is code. In Uniweb, pages are pure content organization.

**URL mapping:**

- `pages/home/` → `/` (home page, when `index: home` in site.yml)
- `pages/about/` → `/about`
- `pages/blog/` → `/blog`
- `pages/blog/[slug]/` → `/blog/:slug` (dynamic route)

### Section

**In Uniweb:** A content unit represented by a markdown file within a page folder, rendered by a component specified in the frontmatter. Sections are **page-local**—they belong to exactly one page and define that page's structure.

**Different from:** Collections, which are reusable content that can appear on multiple pages.

**Key characteristics:**

- Lives in `pages/<page-name>/<section>.md`
- Has a numeric prefix for ordering: `1-hero.md`, `2-features.md`
- Belongs to exactly one page
- Rendered by a component specified in frontmatter `type:`

**Example:**

```markdown
---
type: Hero
layout: centered
---

# Welcome

Your content here.
```

### Collection

**In Uniweb:** A set of reusable content items stored outside the page hierarchy. Collections are for content that appears in multiple places—like blog articles shown as cards on the home page, as a list on the blog page, and as full articles on individual pages.

**Different from:** Sections, which are page-local. A section exists in one place; a collection item can be referenced anywhere.

**Key characteristics:**

- Lives in `library/<collection-name>/`
- Declared in `site.yml` under `collections:`
- Built to `public/data/<collection-name>.json`
- Referenced by pages using `data: <collection-name>`

**Common examples:**

- Blog articles (`library/articles/`)
- Team members (`library/team/`)
- Products (`library/products/`)
- Testimonials (`library/testimonials/`)

### Collection Item

**In Uniweb:** A single entry in a collection, represented by a markdown file with frontmatter metadata and body content. Each item has a `slug` (derived from filename) that serves as its stable identifier.

**Structure:**

```markdown
---
title: Getting Started with Uniweb
date: 2025-01-15
author: Sarah Chen
tags: [tutorial, beginner]
---

Learn how to build your first site with Uniweb.

## Installation

First, create a new project...
```

**Output fields:**

| Field | Source |
|-------|--------|
| `slug` | Filename without extension |
| `title`, `date`, etc. | Frontmatter |
| `content` | Parsed body (ProseMirror JSON) |
| `excerpt` | Auto-generated or from frontmatter |

### Library

**In Uniweb:** The `library/` folder where collections are stored. Each subfolder is a collection.

```
site/
├── pages/           # Page hierarchy (sections)
└── library/         # Reusable content (collections)
    ├── articles/
    ├── team/
    └── products/
```

**Different from:** `pages/`, which defines the site's navigation structure. Library content has no inherent URL—it's data that pages can fetch and display.

### Block

**In Uniweb:** The runtime JavaScript representation of a section. When a section is loaded, the framework creates a Block instance that provides content, params, context, and methods to components.

**Different from:** UI blocks or widgets in other frameworks. Uniweb blocks are runtime instances with lifecycle, state management, and parent-child relationships.

**Component props** (what components receive):

- `content` — Structured content from markdown (title, paragraphs, links, imgs, items, etc.)
- `params` — Configuration from frontmatter with defaults applied
- `block` — The Block instance for context access

**Block properties** (accessing via `block`):

- `block.page` — The Page containing this block
- `block.website` — The Website instance
- `block.childBlocks` — Nested blocks (subsections)

**Fetched data** is accessed via `content.data`, not directly on the block:

```jsx
function BlogList({ content, params, block }) {
  const articles = content.data?.articles || []
  // ...
}
```

### Section vs Collection: When to Use Which

| Use Case | Section | Collection |
|----------|---------|------------|
| Hero banner on home page | ✓ | |
| About page introduction | ✓ | |
| Blog post list | | ✓ |
| Individual blog articles | | ✓ |
| Team member profiles | | ✓ |
| Contact form | ✓ | |
| Product catalog | | ✓ |
| Testimonials (reused on multiple pages) | | ✓ |
| Page-specific testimonial | ✓ | |

**Rule of thumb:** If the content appears in one place, it's a section. If it appears in multiple places or needs to be queried/filtered, it's a collection.

## Component Terms

### Component

**In Uniweb:** A React component that renders content according to the Uniweb interface—receiving `{ content, params, block }` props. Components come in two types: exposed (available to content creators) and internal (implementation details).

**Different from:** Standard React components without the Uniweb content/params/block interface. While Uniweb components can use standard React components internally, the exposed layer must follow the Uniweb interface.

**The interface:**

```jsx
function MyComponent({ content, params, block }) {
  // content: Structured data from markdown
  // params: Configuration from frontmatter
  // block: Runtime context and utilities
  return <div>...</div>;
}
```

### Exposed Component

**In Uniweb:** A component that content creators can reference in markdown frontmatter. These components sit at the boundary between content and code, requiring a `meta.js` file that defines their interface.

**Also called:** User-facing component, boundary component

**Key characteristics:**

- Defined in Foundation's component directory
- Has a `meta.js` file
- Follows the `{ content, params, block }` interface
- Visible to content creators and visual editors

### Internal Component

**In Uniweb:** Regular React components used within exposed components for rendering logic. These are standard React—they can use any props structure, hooks, and patterns. They're not directly available to content creators.

**Key characteristics:**

- Standard React components
- No schema files needed
- Not exposed in the Foundation's interface
- Can use any npm packages and patterns
- Composed by exposed components

## Extension Terms

### Plugin

**In Uniweb:** A build-time extension that can modify content processing, customize build configuration, or extend CLI functionality. Plugins focus exclusively on build-time processes, not runtime behavior.

**Different from:** Traditional CMS plugins that often provide runtime functionality and may mix concerns across the entire stack. Uniweb plugins are specifically scoped to the build process.

**Common types:**

- Asset optimization (image compression, CSS/JS minification)
- Content transformation (markdown extensions, AI translation)
- Build customization (Vite plugins, development tools)
- CLI extension (additional commands and capabilities)

### Schema

**In Uniweb:** Defines the interface of a component or content structure. Component schemas (`meta.js`) specify parameters, presets, and metadata. Content schemas (for JSON blocks) define data structure and validation.

**Purpose:**

- Build-time validation of frontmatter
- Visual editor integration (parameters become UI controls)
- Documentation generation
- Type safety for structured content

**Three types:**

1. **Component schemas** - Define exposed component interfaces
2. **Content schemas** - Define structured data in JSON blocks
3. **Foundation schema** - Defines Foundation-level configuration

## Runtime Terms

### Runtime Connection

**In Uniweb:** The dynamic binding of content to components at runtime rather than build time. Unlike traditional frameworks that compile content and templates together, Uniweb maintains separation until the moment of rendering.

**Different from:** Static template compilation in traditional frameworks where content and presentation are bundled together during build.

**Benefits:**

- Foundations can be updated without rebuilding sites
- Content can be updated without touching component code
- Same Foundation can power multiple sites
- Version control over updates (sites choose update strategy)

### Version Strategy

**In Uniweb:** Site-level configuration that controls how Foundation updates are applied. Evaluated at runtime, giving sites control over their update stability.

**Available strategies:**

- **automatic** - All updates (major, minor, patch)
- **minor-only** - Minor and patch updates only
- **patch-only** - Patch updates only
- **pinned** - No automatic updates (specific version locked)

**Configuration example:**

```yaml
# Site configuration
foundation:
  module: username/marketing
  version: "^1.2.0" # npm semver notation
  strategy: minor-only
```

## Ecosystem Terms

### Uniweb Framework

**What it is:** The open-source framework for building Foundations and sites. Published as `uniweb` on npm, with supporting packages (`@uniweb/build`, `@uniweb/runtime`, `@uniweb/kit`, `@uniweb/core`).

**What it provides:**

- CLI tools for creating and managing projects
- Development server with hot reload (Vite)
- Build system with SSG pre-rendering
- Runtime for connecting content and components
- File-based routing and i18n support

### Uniweb App

**What it is:** Commercial visual editor and managed hosting platform (optional, separate from the Framework).

**What it provides:**

- Professional visual editing experience
- Managed hosting with CDN
- Automatic deployments
- Foundation and site management
- Free for drafts, paid for published sites

### Community Interfaces

**What they are:** Shared specifications for Foundation interfaces, enabling interoperability and best practices across the ecosystem.

**Purpose:**

- Standard component naming conventions
- Common parameter patterns
- Consistent content structure expectations
- Reusable component specifications

**Example:** A "marketing" interface specification might define standard components like Hero, Features, Testimonials with expected parameters and content structures.

## Common Misconceptions

### "Sites must use Uniweb App"

**False.** The Framework works standalone. Uniweb App is optional visual editing, dynamic content database, and managed hosting.

### "Foundations are just component libraries"

**Not quite.** Foundations have a dual interface—both for developers (code) and content creators (schemas, frontmatter). Traditional component libraries only have developer interfaces.

### "You need to write custom components for every site"

**False.** You can use pre-built Foundations based on their license. Custom Foundations are for specialized needs.

### "Content and code are bundled together"

**False.** They remain separate until runtime. Sites load Foundations dynamically—content in markdown, components in the Foundation bundle.

### "Plugins can add runtime features"

**False.** Plugins are build-time only. Runtime features come from Foundation components or the Framework runtime itself.

### "Each site needs its own Foundation"

**False.** Multiple sites can share the same Foundation. That's a key benefit—build once, deploy to many.

## Related Documentation

- [CLI README](https://github.com/uniweb/cli) — Getting started, templates, commands
- [Content Structure](https://github.com/uniweb/cli/blob/main/docs/content-structure.md) — How markdown becomes component props
- [Component Metadata](https://github.com/uniweb/cli/blob/main/docs/component-metadata.md) — The meta.js schema reference
- [Runtime Linking](../sites/runtime-linking.md) — How foundations load via import maps
- [Dual-Mode Development](../sites/dual-mode-development.md) — Standard import vs runtime loading
