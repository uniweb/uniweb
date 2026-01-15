# Uniweb

**Structured Vite + React, ready to go.**

Content/code separation, file-based routing, localization, semantic markdown—all wired up. No framework to learn. Ready for visual editing when your team grows.

---

## Quick Start

```bash
npx uniweb create my-project
cd my-project
pnpm install
pnpm dev
```

You get a workspace with two packages:
- **`site/`** — Content, pages, entry point
- **`foundation/`** — Your React components

Content authors work in markdown. Component authors work in React. Neither can break the other's work, and component updates flow to every site using them. Start simple, scale to multi-site when needed.

## What You Get

```
my-project/
├── site/                     # Content + configuration
│   ├── pages/                # File-based routing
│   │   └── home/
│   │       ├── page.yml      # Page metadata
│   │       └── 1-hero.md     # Section content
│   ├── locales/              # i18n (mirrors pages/)
│   │   └── es/
│   └── public/               # Static assets
│
└── foundation/               # Your components
    └── src/
        └── components/
            └── Hero/
                └── index.jsx
```

**Pages are folders.** Create `pages/about/` with a markdown file inside → visit `/about`. That's the whole routing model.

### Content as Markdown

```markdown
---
component: Hero
theme: dark
---

# Welcome

Build something great.

[Get Started](#)
```

Frontmatter specifies the component and configuration. The body contains the actual content—headings, paragraphs, links, images—which gets semantically parsed into structured data your component receives.

### Components as React

```jsx
export function Hero({ content, params }) {
  const { title } = content.main.header;
  const { paragraphs, links } = content.main.body;
  const { theme = 'light' } = params;

  return (
    <section className={`py-20 text-center ${theme === 'dark' ? 'bg-gray-900 text-white' : ''}`}>
      <h1 className="text-4xl font-bold">{title}</h1>
      <p className="text-xl text-gray-600">{paragraphs[0]}</p>
      {links[0] && (
        <a href={links[0].url} className="mt-8 px-6 py-3 bg-blue-600 text-white rounded inline-block">
          {links[0].text}
        </a>
      )}
    </section>
  );
}
```

Standard React. Standard Tailwind. Content is automatically parsed from markdown into structured objects—with sensible defaults and configurable mapping patterns when you need more control.

**Note:** The `{ content, params }` interface is only for *exposed* components—the ones content creators select in markdown. Internal components (the majority of your codebase) use regular React props.

## Why This Structure

| Feature | What It Means |
|---------|---------------|
| **Two packages** | Components are publishable from day one—no extraction needed |
| **File-based routing** | `pages/about/` → `/about` |
| **Localization built-in** | `locales/es/about/` → Spanish version |
| **Semantic content** | Markdown → structured objects, with configurable mappings |
| **SPA, SSR, or SSG** | Client-side by default, with server rendering and static generation options |
| **Dynamic or static** | Load Foundation at runtime (updates propagate) or bundle at build time—your choice |
| **No lock-in** | Standard React components, leave anytime |

## Scaling Up

The structure is designed to grow with you.

### Multiple Sites

Every project is pre-configured for growth—no config changes needed. Add more sites or foundations alongside your existing structure:

```bash
mkdir -p sites/docs
mkdir -p foundations/documentation
```

Or migrate to a multi-site structure:

```bash
mv site sites/marketing
mv foundation foundations/marketing
# Update package names and dependencies
```

Starting fresh? Use the multi template:

```bash
npx uniweb create my-workspace --template multi
```

```
my-workspace/
├── sites/
│   ├── marketing/       # Main marketing site
│   └── docs/            # Documentation site
└── foundations/
    ├── marketing/       # Marketing components
    └── documentation/   # Docs components
```

### Publish Your Foundation

Your `foundation/` folder is already a complete package. When ready:

```bash
cd foundation
uniweb build
npm publish
```

Other projects can use your components. Updates propagate automatically.

## Packages

| Package | Description |
|---------|-------------|
| [`uniweb`](https://github.com/uniweb/cli) | CLI for creating and building projects |
| [`@uniweb/build`](https://github.com/uniweb/build) | Foundation build tooling |
| [`@uniweb/runtime`](https://github.com/uniweb/runtime) | Site runtime and Vite plugins |

## The Bigger Picture

The structure you start with scales without rewrites:

1. **Single project** — One site, one component library. Most projects stay here.
2. **Multi-site** — One Foundation powers multiple sites. Release it once, updates propagate automatically—no rebuilds needed.
3. **Full platform** — [uniweb.app](https://uniweb.app) adds visual editing, live content management, structured data, and team collaboration. Your Foundation plugs in and its components become native to the editor. Content creators work on real pages instantly.

Start with local markdown files deployed to Vercel. Grow to a collaborative content platform when you need it. Everything is wired from the start; you just connect it.

The separation between content (`site/`) and components (`foundation/`) is what makes this possible. Content authors can't break components; component updates flow to all sites using them.

## Documentation

Full documentation at **[docs.uniweb.io](https://docs.uniweb.io)**:

- Getting Started — project setup, adding pages, creating components
- Content Authoring — markdown format, semantic structure, localization
- Component Development — props, content mapping, presets
- Deployment — Vercel, static hosting, SSR options

## License

Apache 2.0 — See [LICENSE](LICENSE) for details.

---

Uniweb is a trademark of [Proximify Inc.](https://proximify.com)
