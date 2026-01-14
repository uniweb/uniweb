# Uniweb

**Component Web Platform**

The content–component bridge for domain-adaptable publishing.

---

## Quick Start

Create a new Uniweb project with a starter template:

```bash
npx uniweb@latest create my-project --template workspace
```

This scaffolds a Vite project with everything wired—routing, state, and build pipeline ready. You get a site and a Foundation in a monorepo, pre-configured to work together.

```bash
cd my-project
pnpm install
pnpm dev
```

## What is Uniweb?

Uniweb manages how content becomes pages through components. It bridges structured content and purpose-built component systems (called **Foundations**) into a cohesive publishing experience.

There's no heavy framework to learn. Foundations are React component libraries built with Vite and styled with Tailwind. Sites are Vite apps that load content and options from markdown files with frontmatter. The CLI handles the wiring so you can focus on components and content.

- **Foundations** define the vocabulary: available section types, exposed options, rendering rules, and design constraints
- **Content** flows through Foundations without being coupled to presentation
- **The Capability Boundary** protects coherence as sites evolve—creators control content and composition while the system preserves design integrity

## Open Source Packages

| Package                                                | Description                                                   |
| ------------------------------------------------------ | ------------------------------------------------------------- |
| [`uniweb`](https://github.com/uniweb/cli)              | CLI for creating and building Uniweb projects                 |
| [`@uniweb/build`](https://github.com/uniweb/build)     | Foundation build tooling (schema discovery, image processing) |
| [`@uniweb/runtime`](https://github.com/uniweb/runtime) | Vite plugins and loader for sites                             |

## How It Works

**For Foundation Developers:**
Build a Foundation once—a React component library with metadata that defines what creators can do. Components receive content as props and render it within your design system.

**For Site Developers:**
Use `@uniweb/runtime` to load Foundations dynamically via import maps, or bundle them statically for maximum performance.

**For Teams:**
Foundations enforce design coherence architecturally. Multiple editors can publish safely because the Capability Boundary defines what's changeable and what's protected.

## The Uniweb App

These open source packages power the Foundation development workflow. For a complete publishing experience with visual editing, content management, and collaboration, see [uniweb.app](https://uniweb.app).

The Uniweb app is the "head" in this architecture—a real publishing surface where creators compose pages from Foundation components without touching code.

## Documentation

- [Foundation Development Guide](https://github.com/uniweb/cli/blob/main/docs/foundation-development.md)
- [CLI Reference](https://github.com/uniweb/cli/blob/main/docs/cli-reference.md)
- [Architecture Overview](https://github.com/uniweb/cli/blob/main/docs/architecture.md)

## License

Apache 2.0 - See [LICENSE](LICENSE) for details.
