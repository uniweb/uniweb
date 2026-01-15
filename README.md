# Uniweb

**Structured Vite + React, ready to go.**

Content/code separation, file-based routing, localization, semantic markdown—all wired up. No framework to learn. Foundation-ready from day one.

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

Both are proper packages. Add dependencies where they belong. Scale when you need to.

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

Standard React. Standard Tailwind. Content is automatically parsed from markdown into structured objects.

## Why This Structure

| Feature | What It Means |
|---------|---------------|
| **Two packages** | Clear separation—content deps vs component deps |
| **File-based routing** | `pages/about/` → `/about` |
| **Localization built-in** | `locales/es/about/` → Spanish version |
| **Semantic content** | Markdown parsed into structured objects |
| **Foundation-ready** | Components are already extractable/publishable |

## Scaling Up

The structure is designed to grow with you.

### Multiple Sites

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

### Visual Editing

For teams with non-developers, [uniweb.app](https://uniweb.app) provides a visual editor that reads your Foundation and presents components as drag-and-drop building blocks. Same content format, no code changes needed.

## Packages

| Package | Description |
|---------|-------------|
| [`uniweb`](https://github.com/uniweb/cli) | CLI for creating and building projects |
| [`@uniweb/build`](https://github.com/uniweb/build) | Foundation build tooling |
| [`@uniweb/runtime`](https://github.com/uniweb/runtime) | Site runtime and Vite plugins |

## The Bigger Picture

Uniweb is a **Component Web Platform**—it manages how content becomes pages through components.

- **Foundations** define the vocabulary: section types, options, constraints
- **Sites** are pure content that flows through Foundations
- **The Capability Boundary** protects design coherence as sites evolve

Start with a single project. Graduate to multi-site architecture when you need it. Add visual editing when your team grows. The foundation you build today works at every scale.

## Documentation

- [Getting Started](docs/getting-started/understanding-uniweb.md)
- [Foundation Development](docs/foundations/development.md)
- [CLI Reference](docs/reference/cli.md)

## License

Apache 2.0 — See [LICENSE](LICENSE) for details.

---

Uniweb is a trademark of [Proximify Inc.](https://proximify.com)
