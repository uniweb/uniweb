# Why Uniweb

You're starting a content-heavy project—marketing site, documentation, portfolio, client work. You reach for Vite and React. Smart choice. Best-in-class DX, component model that scales, ecosystem you know.

Then you hit the content question.

## The Problem You Keep Solving

Where does content live?

- **Hardcoded in components.** Fast to start. Then the client needs to change a headline and they wait for you. You become the bottleneck.

- **JSON files.** Better. But now you're writing loaders, figuring out data flow, handling the mapping yourself.

- **MDX.** Content and code together. Flexible. But content authors need to understand JSX, and your components couple to specific content structures.

- **Headless CMS.** Clean content modeling. But now you're building a frontend separately, wiring up APIs, managing the gap between what authors edit and what ships.

Each approach works. Each has tradeoffs. And each time, you're solving the same problem: *how does content become pages through components?*

Add localization—now you're wiring i18n infrastructure. Add a second similar site—do you copy the repo and maintain two codebases, or extract shared components and manage versioning?

This is undifferentiated work. Infrastructure *around* your product, not your product.

## The Insight

The relationship between content and components is an architectural concern—not something to solve ad-hoc in every project.

What's missing is a system that manages the content–component binding for you. Still Vite. Still React. But with separation enforced by architecture, not discipline.

## What Uniweb Gives You

Same tools you'd choose anyway:

- **Vite** — same dev server, same build
- **React** — same components, same ecosystem
- **Tailwind** — works out of the box

Plus architecture that handles the binding:

```
my-project/
├── site/                # Content
│   ├── pages/           # File-based routing
│   │   └── home/
│   │       └── 1-hero.md
│   └── locales/         # i18n
└── foundation/          # Components
    └── src/
        └── components/
            └── Hero/
```

**Content is markdown.** Frontmatter specifies which component renders it:

```markdown
---
type: Hero
theme: dark
---

# Welcome

Build something great.

[Get Started](#)
```

The body gets semantically parsed into structured data—headings, paragraphs, links, images—organized and ready for your component. No parsing logic to write.

For content that doesn't fit markdown—products, team members, events—use JSON blocks:

````markdown
```json #team-member
{
  "name": "Sarah Chen",
  "role": "Lead Architect"
}
```
````

Natural content stays in markdown. Structured data goes in JSON blocks. Components receive both as clean objects.

**Components are React.** Exposed components—the ones content authors select via `type`—receive `{ content, params }`:

```jsx
export function Hero({ content, params }) {
  const { title } = content.main.header
  const { paragraphs, links } = content.main.body
  const { theme = 'light' } = params

  return (
    <section className={theme === 'dark' ? 'bg-gray-900 text-white' : ''}>
      <h1>{title}</h1>
      <p>{paragraphs[0]}</p>
      {links[0] && <a href={links[0].url}>{links[0].text}</a>}
    </section>
  )
}
```

Internal components use normal React props. No special interface.

**Frontmatter parameters.** Three are standard: `type` (which component renders this content), `theme` (consistent styling across components), and `preset` (predefined parameter combinations for quick selection). Everything else is yours to define. Want a `layout` option? An `emphasis` toggle? Define it in your schema. Content authors configure; you decide what's configurable.

**The system manages the binding.** Markdown is parsed into structured objects. Components receive `{ content, params }`. You don't write loaders or parsers. You build components that render.

## What's Wired In

- **Routing** — Pages are folders. Create `pages/about/` → visit `/about`.
- **Content preprocessing** — Markdown parsed into structured objects.
- **Localization** — Built-in i18n. Content and UI strings handled consistently.
- **Data fetching** — Dynamic data fetched before render. Components receive data ready to use.
- **Dynamic routes** — Pages can define data sources that auto-generate subroutes. `/blog` with a `[slug]` template renders each post.

You focus on components. The system handles plumbing.

## What This Changes

**You're not the bottleneck.** Content authors work in markdown or the visual editor. They can't break your components. You don't block their changes.

**Clients manage their own sites.** Build a Foundation with your design system. Hand it off. They add pages, update content, publish—without calling you.

**Multiple sites, one Foundation.** A Foundation is self-contained—no dependency on any specific site. Use it three ways:

| Mode | How it works |
|------|--------------|
| **Local folder** | Foundation lives in your workspace |
| **npm package** | `pnpm add @acme/foundation` |
| **Runtime link** | Foundation loads from a URL |

Improve a component, every site using that Foundation gets the update. Sites control their own versioning—automatic updates for patches, opt-in for major changes.

**Localization is built in.** Add a language, mirror the content structure, translations flow through.

## When Content Authors Aren't Developers

Markdown files work great—if you're comfortable with Git, VS Code, and file-based workflows.

Content authors can learn these tools. Some do. But Git isn't designed for content work. VS Code isn't designed for writing marketing copy. Translation workflows with JSON files and CLI tools aren't designed for non-technical teams.

It's developer infrastructure repurposed for content authoring. It works, but it's friction.

[uniweb.app](https://uniweb.app) provides an authoring environment designed for content authors from the start.

**Same architecture, proper interface.** Your Foundation doesn't change. Your components doesn't change. The platform reads your schemas and presents your components as the editing interface. Content authors select sections, configure options, write content—on real pages, not in code editors.

**What the platform adds:**

- Visual editing designed for content work
- Database-backed content for frequent updates
- Team collaboration with roles and permissions
- Instant publishing—no builds, no deployments
- Hosting with global CDN

**The path:**

1. **Start with files** — Markdown in Git. Free. Full control.
2. **Add the platform** — When content authors need a proper environment.

You're not locked into either. The architecture is the same throughout.

## When This Fits

- Content-heavy sites: marketing, docs, portfolios, client work
- Teams where non-developers need to publish
- Agencies building similar sites repeatedly
- Projects where content–code separation should be architectural, not just organizational

## When It Doesn't

- Applications where content and code are inherently coupled—dashboards, interactive tools, data visualizations
- Projects requiring routing beyond folders and dynamic segments
- Situations where you need control over every infrastructure decision

## Try It

```bash
npx uniweb@latest create my-site --template marketing
cd my-site
pnpm install
pnpm dev
```

A working site with real components—Hero, Features, Pricing, Testimonials, FAQ. See how content flows to components. Modify a component, see it update. Change content, see it render.

```bash
# Multilingual business site
npx uniweb@latest create my-site --template international

# Academic site
npx uniweb@latest create my-site --template academic

# Documentation site
npx uniweb@latest create my-site --template docs

# Minimal starter
npx uniweb@latest create my-site
```

## The Bottom Line

You're not switching away from Vite and React. You're adding architecture that makes content–component separation explicit and managed.

Same DX. Same component model. Less infrastructure to build yourself. Content authors who don't need you for every change.

[Docs](https://docs.uniweb.io) • [GitHub](https://github.com/uniweb) • [Platform](https://uniweb.app)