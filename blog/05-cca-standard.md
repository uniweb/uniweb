# Component Content Architecture (CCA)

## An open standard for binding content to components

**Abstract**

For the last decade, web development has bifurcated. We have established standards for managing data (structured content, JSON APIs) and standards for rendering interfaces (React, component libraries). But we lack a standard for the *relationship* between them.

Currently, the logic that binds "Content A" to "Component B" is either hard-coded in proprietary themes or glued together in custom frontend code. This creates fragility, vendor lock-in, and the inability to move publishing systems between environments.

This paper introduces **Component Content Architecture (CCA)**: an open architectural pattern for modeling the binding between structured content and UI components. CCA is the technical foundation for a new category of publishing tools—**Component Content Systems**—that manage how content becomes pages through components.

---

## 1. The Missing Layer

In modern web development, we understand two domains well:

**The Content Domain (Meaning)**
Modeled in schemas, stored in databases, delivered via API. This is what headless CMS does well.

**The Component Domain (Language)**
Modeled in JavaScript/TypeScript, stored in git, rendered in the browser. This is what React and modern frontend frameworks do well.

The failure mode of current architectures—Headless, Jamstack, and the rest—is that they treat these two domains as unrelated until the very last moment of rendering. The binding between content and components gets hard-coded into templates, scattered across frontend code, or locked inside proprietary page builders.

**Component Content Architecture** proposes a third domain: **The Binding**.

CCA defines:

- How a specific piece of content maps to a component's inputs
- What constraints govern that mapping (the Capability Boundary)
- How components compose into larger structures without hard-coding

The Binding is not an implementation detail. It's a first-class architectural concern.

---

## 2. The Foundation: CCA's Unit of Portability

In CCA, the primary unit of portability is called a **Foundation**.

A Foundation is a self-contained package that bundles everything needed to define a website's publishing language. In practice, it's a directory with a clear structure:

```
foundation/
└── src/
    └── components/
        ├── Hero/
        │   ├── index.jsx      # The React component
        │   └── schema.json    # What content it accepts
        ├── Features/
        └── Pricing/
```

A Foundation contains:

**Component Definitions**
The actual code—React components (or any rendering technology)—that turns content into UI. These are standard components; no proprietary abstractions.

**Schema Manifests**
The formal definition of what content each component accepts: its parameters, their types, their constraints. The schema is what makes a component "exposed" to content creators.

**Composition Rules**
The logic that defines how components can be arranged: what can contain what, what sequences are valid, what options are exposed.

Because the Foundation defines the *rules* of the site, content can remain purely structural—clean data that doesn't know or care how it will be rendered. Content lives separately:

```
site/
├── pages/
│   └── home/
│       ├── page.yml         # Page metadata
│       └── 1-hero.md        # Section content
└── locales/
    └── es/                  # i18n mirrors pages/
```

**The binding in action.** Content authors write natural markdown:

```markdown
---
type: Hero
theme: dark
---

# Welcome

Build something great.

[Get Started](#)
```

Components receive structured, semantic content—headings, paragraphs, links, images—organized and ready to render. Content authors write naturally. The Foundation defines what structure each component expects and what configuration options (like `theme`) it exposes.

This is the binding layer: the system manages the transformation from natural content to component-ready data. The Foundation defines the contract; content flows through it.

This separation is what enables **domain adaptability**. A Marketing Foundation and a Documentation Foundation can run on the same CCA-compliant system. Swap the Foundation, and the same content infrastructure serves a completely different publishing purpose.

---

## 3. The Capability Boundary

One of CCA's core concepts is the **Capability Boundary**: the architectural line between what content creators can change and what they cannot.

**Inside the boundary** (creator territory):
- Content (text, images, structured data)
- Composition (which components, in what order)
- Exposed options (variants, toggles, emphasis levels)

**Outside the boundary** (protected by the Foundation):
- Visual design (spacing, typography, color systems)
- Component behavior (interactions, responsive rules)
- Structural constraints (valid combinations, nesting rules)

In most systems, these boundaries exist only as policy—documented guidelines that people are expected to follow. In CCA, the boundaries are enforced by the architecture itself. The Foundation defines what's possible; creators work within those possibilities.

This is why we say **"Code is Policy."** You don't hope the marketing team uses the right spacing—you define components where the spacing is already right. The architecture enforces what documentation cannot.

---

## 4. Why CCA Should Be a Standard

The web would benefit from CCA becoming an open standard rather than a proprietary implementation detail. Here's why:

**Portability**
If your content and your Foundation both follow CCA conventions, you can move between implementations—or self-host—without rebuilding your site. Your content is data. Your components are code. The architecture is the contract that binds them.

**Interoperability**
Foundations built to CCA specs could work across multiple CCA-compliant runtimes. A Foundation developed for one system could, in principle, run on another.

**Longevity**
Content outlives products. When your publishing system is built on an open architecture, your investment in content and components survives vendor changes, technology shifts, and organizational pivots.

**Clarity**
CCA gives teams a shared vocabulary for discussing the binding layer. Instead of ad-hoc solutions, you have a principled architecture that separates concerns cleanly.

---

## 5. CCA and Component Content Systems

CCA is an architecture. **Component Content System (CCS)** is the category of products built on that architecture.

A Component Content System is any implementation that manages the binding between content and components as a first-class concern, uses Foundations to define the publishing language, enforces the Capability Boundary architecturally, and enables domain adaptability through Foundation swapping.

Some Component Content Systems are hosted platforms with visual editing and collaboration features. Others are CLI tools and build systems for developers working locally. A CCS could even be a desktop application that works entirely on the local filesystem. The architecture is the same; the deployment model varies.

For a full treatment of the category—what it is, why it exists, and how it differs from traditional CMS, headless, and builders—see [Introducing Component Content Systems](/ccs). This paper focuses on the architectural specification.

---

## 6. The Uniweb Implementation

Uniweb is our implementation of Component Content Architecture. It takes a dual approach:

**The Open-Source Tooling**

At its core, Uniweb provides open tooling for building CCA-compliant Foundations:

```
npx uniweb@latest create my-project
cd my-project
pnpm install
pnpm dev
```

This scaffolds a workspace with the CCA structure already wired:

```
my-project/
├── site/                # Content + configuration
│   ├── pages/           # File-based routing
│   └── locales/         # i18n
└── foundation/          # Your React components
    └── src/
        └── components/
```

Content authors work in markdown. Component authors work in React. The separation is architectural—neither can break the other's work, and component updates flow to every site using them.

The ecosystem includes:
- **`uniweb`** — CLI for creating and building projects
- **`@uniweb/build`** — Foundation build tooling  
- **`@uniweb/runtime`** — Foundation loader and orchestrator

Host anywhere. Because CCA separates content from runtime, you deploy your Foundation and content on any infrastructure you control.

**The Managed Platform (uniweb.app)**

For teams that prioritize speed and collaboration over infrastructure management, Uniweb.app is a hosted Component Content System:

- **Instant Runtime**: Publish a Foundation, get a visual editing environment
- **SaaS Convenience**: Zero config, instant previews, role-based access, global hosting
- **The Native Effect**: Your Foundation's components become native to the editor—content creators work on real pages, not abstract forms

**The Scaling Path**

The structure scales without rewrites:

1. **Single project** — One site, one component library. Most projects stay here.
2. **Multi-site** — One Foundation powers multiple sites. Release once, updates propagate automatically.
3. **Full platform** — Uniweb.app adds visual editing, live content management, and team collaboration. Your Foundation plugs in directly.

**The Eject Button Is Built In**

Because both the managed platform and the open-source tooling implement the same Component Content Architecture, you're never locked in. Move a Foundation from Uniweb.app to self-hosted infrastructure—or vice versa—without rebuilding.

The workflow is the same either way:

```bash
uniweb build      # Build your Foundation
uniweb publish    # Release to uniweb.app (or npm publish for self-hosted)
```

Sites link to Foundations at runtime and control their own update strategy. You own your Foundations and can license them to sites—yours or your clients'.

---

## 7. Getting Started

Start building today:

```bash
# Minimal starter
npx uniweb@latest create my-project

# Feature-rich marketing template
npx uniweb@latest create my-site --template marketing

# Multi-site monorepo
npx uniweb@latest create my-workspace --template multi
```

- **Explore the category**: Read [Introducing Component Content Systems](./03-ccs-category.md) to understand the bigger picture
- **Read the docs**: Full documentation at [uniweb.io/docs](https://uniweb.io/docs)
- **Use the platform**: Deploy to [uniweb.app](https://uniweb.app) for a managed experience
- **View the source**: [github.com/uniweb/cli](https://github.com/uniweb/cli)

---

## Conclusion

Component Content Architecture addresses a gap that has existed since the component era began: we've had standards for content and standards for rendering, but no standard for the binding between them.

CCA proposes that the binding is not a detail to be solved ad-hoc in every project—it's a first-class architectural layer that deserves formal treatment.

Build on CCA, and you get portability, enforced design constraints, domain adaptability, and a clean separation between what creators control and what the system protects.

That's the architecture. The systems built on it—Component Content Systems—are the future of publishing.