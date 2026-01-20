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

**In Uniweb:** The technical packaging and delivery mechanism that connects a Foundation to a site at runtime. Modules use Webpack Module Federation to load dynamically, enabling instant updates without redeployment.

**Different from:** JavaScript modules (ESM/CommonJS) or CMS plugins in other systems. Uniweb modules are specifically about runtime delivery of complete design systems.

**Technical detail:** Modules enable dependency sharing between Foundations and sites, allowing for efficient loading and consistent React versions.

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

### Page

**In Uniweb:** A folder whose path that maps to a URL path and contains markdown files representing sections, and a `page.yml` file defining page properties. Pages organize sections and define metadata like title, description, and section hierarchy.

**Different from:** Template files in traditional systems where the page structure is code. In Uniweb, pages are pure content organization.

**URL mapping:**

- `pages/index/` → `/` (home page)
- `pages/about/` → `/about`
- `pages/blog/post-name/` → `/blog/post-name`

### Section

**In Uniweb:** A content unit represented by a markdown file, rendered by a component specified in the frontmatter. Sections are the atomic units of content that content creators work with.

**Different from:** Page regions or blocks in traditional CMSs that are often tied to specific layouts.

**Example:**

```markdown
---
type: HeroSection
layout: centered
---

# Welcome

Your content here.
```

### Block

**In Uniweb:** The runtime JavaScript representation of a section. When a section is loaded, the framework creates a block object that provides content, params, context, and methods to components.

**Different from:** UI blocks or widgets in other frameworks. Uniweb blocks are runtime instances with lifecycle, state management, and parent-child relationships.

**Key properties:**

- `block.id` - Unique identifier
- `block.content` - Content to render (same as the explicit `content` prop)
- `block.params` - Parameters defining rendering (same as the explicit `params` prop)
- `block.input` - Dynamically fetched data
- `block.page` - The page object that contains the block
- `block.site` - The website object that contains the page that contains the block
- `block.getChildBlocks()` - Access child blocks

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
- Build customization (webpack configuration, development tools)
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

**What it is:** The open-source (GPL-3.0) framework for building Foundations and sites. It includes a Runtime Engine (RTE) to render sites with their chosen Foundations.

**What it provides:**

- CLI tools for creating and managing projects
- Development server with hot reload
- Build system (Webpack, Babel, PostCSS)
- Runtime for connecting content and components
- Module Federation for dynamic loading

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

**False.** They remain separate until runtime. Sites reference Foundations dynamically via Module Federation.

### "Plugins can add runtime features"

**False.** Plugins are build-time only. Runtime features come from Foundation components or the Framework runtime itself.

### "Each site needs its own Foundation"

**False.** Multiple sites can share the same Foundation. That's a key benefit—build once, deploy to many.

## Related Documentation

- [Understanding Uniweb](../getting-started/understanding-uniweb.md) - Conceptual architecture overview
- [Developers Guide](../developers-guide.md) - Technical guide for building Foundations

## Contributing to This Reference

This terminology reference is maintained to help clarify Uniweb's specific terminology. If you notice ambiguity or have suggestions for additional terms, please contribute via GitHub.
