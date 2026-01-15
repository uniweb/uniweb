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

You get a working Vite + React site with:
- **`site/`** — Content, pages, entry point
- **`foundation/`** — Your React components

**Why two packages?** Your components are immediately publishable—no extraction step when you want to reuse them across projects or share with others. For small projects, you'll barely notice the separation. For larger ones, you'll appreciate it.

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

## Why This Structure

| Feature | What It Means |
|---------|---------------|
| **Two packages** | Components are publishable from day one—no extraction needed |
| **File-based routing** | `pages/about/` → `/about` |
| **Localization built-in** | `locales/es/about/` → Spanish version |
| **Semantic content** | Markdown → structured objects, with configurable mappings |
| **No lock-in** | Standard React components, leave anytime |

## Scaling Up

The structure is designed to grow with you.

### Multiple Sites

When you need multiple sites or foundations, use the workspace template:

```bash
npx uniweb create my-workspace --template workspace
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
2. **Multi-site** — Share components across projects. Your `foundation/` becomes a dependency.
3. **Visual editing** — [uniweb.app](https://uniweb.app) reads your components and lets non-developers build pages. Optional, and the CLI works fine without it.

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
