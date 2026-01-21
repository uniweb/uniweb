# Uniweb

A component content architecture for React. Build sites where content authors and component developers can't break each other's work—and scale from local files to visual editing without rewrites.

Uniweb separates **content** (markdown files, structured data) from **components** (React, portable across sites). The architecture enforces this separation, so teams can work independently and foundations can power multiple sites—locally or on the managed platform.

## Quick Start

```bash
npx uniweb@latest create my-site --template marketing
cd my-site
pnpm install
pnpm dev
```

See the [CLI documentation](https://github.com/uniweb/cli) for templates, commands, and configuration.

## Repositories

| Repository                                              | Description                                           |
| ------------------------------------------------------- | ----------------------------------------------------- |
| [uniweb/cli](https://github.com/uniweb/cli)             | CLI for creating and building projects (`npx uniweb`) |
| [uniweb/build](https://github.com/uniweb/build)         | Foundation build tooling                              |
| [uniweb/runtime](https://github.com/uniweb/runtime)     | Foundation loader and orchestrator for sites          |
| [uniweb/templates](https://github.com/uniweb/templates) | Official project templates                            |

## Resources

**For developers**

- [uniweb.io](https://uniweb.io) — Developer hub
- [uniweb.io/docs](https://uniweb.io/docs) — Framework documentation
- [CLI README](https://github.com/uniweb/cli) — Detailed usage and reference

**For teams and content creators**

- [uniweb.app](https://uniweb.app) — Managed platform with visual editing
- [docs.uniweb.app](https://docs.uniweb.app) — Platform documentation

## The Architecture

Uniweb implements **Component Content Architecture (CCA)**—a standard for modeling the relationship between content and components. Instead of hard-coding bindings in templates or scattering them across frontend code, CCA treats the binding as a first-class architectural layer.

The result: content stays portable, components stay reusable, and the same foundation can power local development, static builds, or a full visual editing platform.

- [Component Content Architecture](/blog/cca) — The architectural pattern
- [Component Web Platforms](/blog/cwp) — The product category

## License

Apache 2.0

---

Uniweb is a trademark of [Proximify Inc.](https://proximify.com)
