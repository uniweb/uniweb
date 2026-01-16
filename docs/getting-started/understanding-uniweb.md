# Understanding Uniweb: Bridging Development and Content Creation

Uniweb operates at a higher level of abstraction than typical web frameworks. Rather than building websites directly, you build **Foundations**—collections of components designed to interface with content creators, not just developers.

**The key insight:** Most frameworks give components a single interface for developers (JavaScript APIs, props, imports). Uniweb adds a second interface layer for content creators (markdown frontmatter, schemas, declarative composition). This dual-interface design enables clean content/code separation at any scale—from solo developers organizing their own site to teams serving multiple projects.

## Why "Foundation" Instead of "Component Library"?

The term "component library" suggests code meant to interact with other code—packages you `npm install` and import into your code. Think React, NextJS, Material-UI, shadcn/ui. These are tools developers use to build websites.

**Foundations are fundamentally different.** They're designed to bridge from the developer world into the content creation world.

When you build a Foundation, you're creating an interface that content creators interact with—through markdown frontmatter, declarative configuration, and optionally through visual editors. The components aren't code that developers import; they're building blocks that content teams compose with.

This cross-domain bridging is the key distinction:

- **Component library**: Developer imports → Developer uses in code → Developer deploys
- **Foundation**: Developer builds → Content creator references → Content creator composes pages

## Two Types of Interfaces

To understand Foundations, you need to see the difference between developer-facing and content-facing interfaces:

### Developer-Facing Interface (Traditional Component Libraries)

```jsx
import { HeroSection } from "@my/library";

function MyPage() {
  return <HeroSection layout="centered" background="dark" title="Welcome" />;
}
```

The consumer is a developer. They write code, manage imports, and deploy applications.

### Content-Facing Interface (Uniweb Foundations)

```markdown
---
type: HeroSection
layout: centered
background: dark
---

# Welcome to Our Platform
```

The consumer is a content creator. They write markdown, configure through frontmatter, and compose pages. No code. No imports. No deployment pipelines.

**This is the fundamental shift.** Foundations are designed for the second pattern—where components are referenced declaratively by people who don't write code. Even working solo on a single site, this pattern brings clean separation: your content lives in markdown, your components in React. The architecture scales when your needs do.

## The Core Concept

Most web frameworks help you build _a website_. Uniweb helps you build _a system for building websites_.

This might seem like over-engineering for a single site, but the architecture provides immediate benefits: content lives in markdown (easy to edit, version-controlled), components are standard React (familiar patterns), and the separation prevents code and content from tangling. It's no more complex than other React frameworks, just better organized.

The abstraction level also enables scenarios that traditional frameworks make difficult: building multiple related sites, separate teams for development and content, component improvements that propagate automatically, and version control over breaking changes. These capabilities are available when needed, not required upfront.

**Think of it this way:**

- **Traditional approach**: Build a website → Deploy it → Developers maintain code and content together
- **Uniweb approach**: Build a Foundation → Compose sites with it → Architecture scales from one site to many

## Three-Tier Architecture

Uniweb enforces a clean separation across three layers:

### 1. Content (Sites)

Markdown files with YAML frontmatter that specify _which_ component to use and _how_ to configure it. Content teams work here exclusively.

```markdown
---
type: HeroSection
layout: centered
background: dark
---

# Welcome to Our Platform

Discover innovative solutions for your business.
```

### 2. Components (Foundations)

React component libraries that provide the implementations. Developers work here, building the components that content teams will reference.

### 3. Runtime Connection (Module Federation)

Webpack's Module Federation technology dynamically loads Foundations at runtime, enabling dependency sharing and instant updates. When you improve your Foundation, connected sites receive the update on their next page load—no redeployment needed.

## Inside vs Outside the Boundary

A Foundation contains two types of components:

**Exposed components** sit at the boundary with content creators. These are defined by schemas and can be referenced in markdown frontmatter. This is where the Framework's architecture matters—these components receive preprocessed content, follow the content-facing interface conventions, and must be declared in schemas.

**Internal components** are pure React implementation details. They're imported and used by your exposed components using standard React patterns. Import any npm packages you want. Use any styling approach. Compose components however you prefer. This is just regular React development—the Framework doesn't prescribe anything here.

```jsx
// Internal component - standard React, no framework involvement
import { Button } from "some-ui-library";

function CallToAction({ text, href }) {
  return (
    <Button variant="primary" href={href}>
      {text}
    </Button>
  );
}

// Exposed component - this is the content-facing interface
export default function HeroSection({ block }) {
  const { title, subtitle, ctaText, ctaLink } = block.main.content;

  return (
    <section>
      <h1>{title}</h1>
      <p>{subtitle}</p>
      <CallToAction text={ctaText} href={ctaLink} />
    </section>
  );
}
```

Content creators only interact with `HeroSection` (because it's in a schema). The `CallToAction` component is invisible to them—it's internal implementation. This boundary keeps concerns separated: your markdown content references `HeroSection`, your code composes `HeroSection` from internal components.

**This is why "Foundation" fits better than "component library."** You're not exposing every component to content creators. You're building a foundation with a carefully designed surface area—the exposed components—while the internal structure is entirely up to you and standard React practices.

## The Content-Facing Interface

The key architectural insight is that **a Foundation's interface is designed for content creators, not developers**.

When you expose a component in your Foundation, you're not creating a JavaScript API for other developers to import. You're creating an option that content creators can reference in markdown frontmatter:

```markdown
---
type: HeroSection
layout: centered
background: dark
---
```

This is the contract boundary: content declares _what_ component to use and _how_ to configure it, while the Foundation provides the implementation. Content creators never touch JavaScript—they compose with components as if they were native features.

**Only exposed components appear in this interface.** Internal components—the React components you use to build your exposed components—remain invisible to content creators. They're standard React implementation details, composed using normal import patterns and npm packages. The Framework only cares about the boundary layer.

### Component Schemas Define the Contract

Component schemas define this content-facing interface:

- At minimum: the component's name (what content creators can reference)
- For parameterized components: available options content creators can configure
- For rich integration: schemas become visual editor UIs (optional benefit, not required)

This schema-driven approach serves multiple audiences:

- **Build-time**: Validates that content references valid components and options
- **Content creators**: Defines what's possible through frontmatter configuration
- **Visual editors** (optional): Can translate components into native UI building blocks

The Foundation's interface isn't `import { HeroSection } from '@my/foundation'`—it's the set of component names and options that appear in markdown frontmatter. This is a content interface, not a code interface.

### Structured Content Beyond Markdown

Regular markdown is more powerful than it appears. The Framework parses markdown into structured data that components receive: headings define sections and subsections, content gets organized hierarchically, and standard elements (lists, links, images, videos) are all accessible as structured data. This means content creators can write naturally—as sections with text, images, and lists—while components receive well-structured content to work with.

For many components, this is sufficient. A feature section with headings, paragraphs, and images? Natural markdown. A blog post with subsections and embedded media? Natural markdown.

However, some content models don't map naturally to markdown's patterns. A team member with name, role, bio, and avatar? A product with SKU, price, variants, and specifications? These need explicit structure.

For these cases, Uniweb supports JSON code blocks with schema hashbangs:

````markdown
```json #team-member
{
  "name": "Sarah Chen",
  "role": "Lead Architect",
  "bio": "10+ years building distributed systems",
  "avatar": "/assets/sarah.jpg"
}
```
````

The `#team-member` schema tag references a content schema that validates structure and ensures type safety. Your component receives this structured data:

```jsx
export default function TeamMember({ block }) {
  const member = block.getBlockData("#team-member");
  // member = { name: "Sarah Chen", role: "Lead Architect", ... }

  return (
    <div className="team-member">
      <img src={member.avatar} alt={member.name} />
      <h3>{member.name}</h3>
      <p className="role">{member.role}</p>
      <p className="bio">{member.bio}</p>
    </div>
  );
}
```

**The design philosophy**: Use markdown's natural structure for most content. Reach for JSON blocks only when the content model requires it.

## Progressive Complexity

The framework supports the entire spectrum of use cases:

**Simple end**: Build hardcoded React components for a single site. Use minimal schemas (just component names). Work like any React framework. This is a legitimate, production-ready approach—many sites stay here permanently.

**Complex end**: Create comprehensive design systems with rich schemas, parameterized components, theming, presets, and deep integration with visual editors. Serve multiple sites with content creators who never touch code.

Start anywhere on this spectrum and grow as needs evolve. The architecture doesn't dictate your complexity—it accommodates it. You're not compromising by starting simple; you're making a pragmatic choice that preserves future options.

**The Framework is unopinionated about implementation.** How you build your exposed components internally—what npm packages you use, how you structure internal components, what styling approach you take—is entirely standard React development. The Framework only defines the boundary layer where components meet content. Inside that boundary, it's your choice.

## Single Source of Truth

A Foundation is the canonical implementation. When you update a component:

1. The improvement exists in one place (the Foundation)
2. All connected sites can receive the update (based on their version strategy)
3. No per-site redeployment is required (runtime loading)

Even with a single site, your components live in one place. Update your Foundation during development, refresh the page, see the changes. For multiple sites, this architectural decision eliminates the traditional problem of component libraries that require explicit upgrades and redeployment across consuming projects.

**Sites control their update strategy** — choosing between automatic updates, conservative policies (minor/patch only), or pinned versions. This configuration is evaluated at runtime, giving sites control over their update stability.

## Why This Abstraction Level?

Traditional web frameworks optimize for building individual websites. Uniweb optimizes for **clean content/code separation** with an optional scaling path.

The abstraction level enables scenarios that traditional frameworks make difficult:

- Building multiple related sites (different clients, brands, or products)
- Separate teams for development and content
- Component improvements that propagate automatically across sites
- Version control over breaking changes
- Vertical specialization (a framework for documentation, marketing, real estate, etc.)

These capabilities are available when needed, not required upfront. Start with one site and standard React development. The architecture accommodates growth without requiring rewrites.

## In Practice: Two Worlds, One System

**Developers** build Foundations—creating components with content-facing interfaces. Whether you're working solo on your own site or building for content teams, you're designing how components will be referenced and configured in markdown.

**Content creators** compose sites by referencing your components through markdown frontmatter or visual editors. To them, your `HeroSection` component is as native as a heading or paragraph—just a building block they can use and configure. (If you're working solo, you're both the developer and the content creator—but the separation still provides clean architecture.)

**Sites** control their update strategy—choosing between automatic updates, conservative policies (minor/patch only), or pinned versions.

This is fundamentally different from traditional component libraries:

- **Component library**: Developers consume the API in code
- **Foundation**: Content creators consume the interface through declarative markup

The separation eliminates bottlenecks: Developers aren't blocked by content changes. Content teams aren't waiting for developer availability to launch pages. Each role works in their domain—code for developers, content for creators—connected through the Foundation's interface.

## The Broader Ecosystem

While the Framework is open source and works standalone, it integrates with a commercial ecosystem:

- **Uniweb App**: Professional visual editor and managed hosting
- **Foundation Registry**: Publish and share Foundations
- **Community**: Shared interfaces, specifications, and best practices

You can use the Framework entirely on your own infrastructure, or leverage the ecosystem as needs grow. The architecture avoids lock-in while providing a path to managed services.

---

## What's Next?

Now that you understand the architectural concepts, here's where to go:

**Start building:**

- **[Uniweb for Developers](../developers-guide.md)** — Technical guide for building Foundations
- **[Documentation](https://docs.framework.uniweb.app)** — Complete API reference and tutorials
- **[Examples](https://github.com/uniwebcms/examples)** — Sample Foundations and component patterns

**Explore the ecosystem:**

- **[Uniweb App](https://uniweb.app)** — Visual content editor and hosting platform
- **[Community Interfaces](https://github.com/uniwebcms/interfaces)** — Standard component specifications
- **[Framework Website](https://framework.uniweb.app)** — Guides and comprehensive resources

The best way to internalize these concepts is to create a project and experiment. Start simple, add complexity as you need it, and let the architecture guide your decisions.
