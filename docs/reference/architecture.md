# Architecture Decisions

This document explains the key architectural decisions made for the Uniweb build system.

## Context

Uniweb separates content from code, allowing:
- Developers to build component systems (foundations)
- Content creators to edit content visually
- Sites to be rendered from content + foundation

The build system must support:
1. Excellent developer experience (Vite-level DX)
2. Runtime composition (load foundations dynamically)
3. Static builds (bundle everything for performance)
4. Asset handling (icons, images, CSS with Tailwind)

## Decision 1: Vite as the Primary Build Tool

### Options Considered

| Tool | Strengths | Weaknesses |
|------|-----------|------------|
| **Webpack** | Battle-tested, Module Federation | Slow, complex config |
| **Vite** | Fast DX, simple config, modern | Less mature for edge cases |
| **esbuild** | Extremely fast | Limited plugin ecosystem |
| **Rollup** | Excellent tree-shaking | No dev server |

### Decision

**Use Vite for development and both build modes.**

Vite provides:
- Near-instant dev server startup (native ESM in dev)
- Hot Module Replacement that preserves state
- Rollup-based production builds (excellent optimization)
- Built-in PostCSS/Tailwind support
- Simple configuration

The DX improvement over Webpack is substantial. Since our runtime linking needs are simple (see Decision 2), we don't need Webpack's Module Federation.

## Decision 2: Native ESM + Import Maps over Module Federation

### The Problem

When a foundation is loaded at runtime, it needs to use the host's React, not bundle its own copy. Two mechanisms can achieve this:

**Module Federation** (Webpack)
```javascript
// webpack.config.js
new ModuleFederationPlugin({
  shared: { react: { singleton: true } }
})
```

**Import Maps** (Browser native)
```html
<script type="importmap">
{ "imports": { "react": "/shared/react.mjs" } }
</script>
```

### Analysis

| Aspect | Module Federation | Import Maps |
|--------|-------------------|-------------|
| Build complexity | Higher (federation plugin) | Lower (standard ESM) |
| Runtime | Federation runtime required | Native browser |
| Version negotiation | Automatic | Manual (host controls) |
| Bidirectional sharing | Yes | No (host → module only) |
| Tooling lock-in | Webpack (or Vite plugin) | Any ESM-capable bundler |

### Decision

**Use native ESM + Import Maps.**

Reasoning:
1. **Our needs are simple** — Host provides React, foundation consumes it. We don't need version negotiation or bidirectional sharing.
2. **We control the host** — Uniweb controls the runtime environment, so we can mandate React versions.
3. **Standards-based** — Import maps are a web standard, not a bundler feature.
4. **Vite-native** — No plugins required; foundations are standard ESM library builds.
5. **Future-proof** — Browser support is now excellent (95%+).

### Tradeoffs Accepted

- No automatic version negotiation. If a foundation needs React 19 features and the host has React 18, it breaks. Mitigation: document supported React versions per foundation.
- No sophisticated chunk loading across module boundaries. Mitigation: foundations handle their own internal chunking.

## Decision 3: Foundations as Self-Contained Bundles

### Options Considered

1. **Preserve module structure** — Foundation exposes individual component files
2. **Single bundle** — Foundation is one file with all components
3. **Hybrid** — Entry point plus lazy-loaded chunks

### Decision

**Single bundle with CSS injected, assets inlined or hashed.**

The runtime treats foundations as black boxes. It calls `getComponent('Hero')` and renders whatever comes back. The runtime doesn't need to know about internal structure.

Benefits:
- Simpler loading (one import)
- CSS is bundled, no separate stylesheet to load
- Icons/assets resolved at build time
- Tree shaking happens at foundation build time

Vite config:
```javascript
export default {
  build: {
    lib: {
      entry: 'src/index.js',
      formats: ['es'],
      fileName: 'foundation'
    },
    rollupOptions: {
      external: ['react', 'react-dom', 'react/jsx-runtime']
    },
    cssCodeSplit: false  // Inject CSS into JS
  }
}
```

## Decision 4: Component Interface

### Design

Foundations expose a minimal interface:

```typescript
interface Foundation {
  getComponent(name: string): React.ComponentType<{ content: any }> | undefined
  listComponents(): string[]
  getSchema?(name: string): ComponentSchema | undefined
  getAllSchemas?(): Record<string, ComponentSchema>
}
```

Components receive content as a prop:

```typescript
interface ComponentProps {
  content: Record<string, any>
}
```

### Rationale

- **`content` as single prop** — Keeps the runtime simple. It doesn't parse content; it just passes it through.
- **Schema separate from rendering** — Schemas are for the editor, not the runtime. They're optional exports.
- **No lifecycle hooks** — Components are pure renderers. Side effects belong elsewhere.

## Decision 5: Development Preview

### Problem

Foundation developers need to preview components with HMR, but foundations aren't apps—they're libraries.

### Solution

Each foundation includes a `dev-preview.jsx` that:
- Imports components directly (not through the library interface)
- Provides sample content for each component
- Renders a preview UI with component switcher

This file is excluded from the production build. It's purely for development.

```javascript
// dev-preview.jsx (not in prod bundle)
import { Hero, FAQ } from './index.js'

const sampleContent = { /* ... */ }

function DevPreview() {
  return <Hero content={sampleContent.Hero} />
}
```

## Decision 6: Static vs. Runtime as Configuration

### Design

The same foundation source supports both build modes:

**Runtime mode** (library build):
```javascript
// vite.config.js
build: {
  lib: { entry: 'src/index.js', formats: ['es'] },
  rollupOptions: { external: ['react', 'react-dom'] }
}
```

**Static mode** (consumed as dependency):
```javascript
// site's vite.config.js - no special config
// Just import components: import { Hero } from 'foundation-example'
```

### Rationale

Foundation authors don't choose the mode—site builders do. A foundation should be usable both ways without modification.

## Future Considerations

### SSR / SSG

Currently, this PoC demonstrates client-side rendering only. Production would add:
- Server-side rendering for initial page load
- Static site generation for pure static sites
- Hydration strategies

### TypeScript

The PoC uses JavaScript for simplicity. Production would use TypeScript throughout, with:
- Typed component props
- Typed schemas
- Type-safe content loading

### Schema Collection

The PoC shows schemas as static properties on components. Production would:
- Collect schemas at build time
- Generate a `schema.json` manifest
- Validate content against schemas

### Monorepo Tooling

For larger implementations, consider:
- Turborepo or Nx for build orchestration
- Changesets for versioning
- Shared ESLint/Prettier configs
