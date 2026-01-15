# Dual-Mode Architecture

This document explains the technical design decisions behind Uniweb's dual-mode foundation loading system.

## Design Goals

1. **Developer Experience First** - Standard Vite DX during development
2. **Production Fidelity** - Ability to test exact production behavior
3. **Single Codebase** - Foundation works in both modes without changes
4. **Minimal Configuration** - Mode selection via single environment variable

## The Challenge

Foundations are React component libraries that need to:

1. **Share React** with the host application (no duplicate React instances)
2. **Load at runtime** in production (independent deployment)
3. **Support HMR** during development (fast iteration)

These requirements create tension:

| Requirement      | Standard Import                | Runtime Loading                              |
| ---------------- | ------------------------------ | -------------------------------------------- |
| Share React      | Automatic (same bundle)        | Requires import maps or transform            |
| Runtime loading  | Bundled at build               | Dynamic import()                             |
| HMR support      | Native Vite HMR                | Requires rebuild + reload                    |

## Solution Architecture

We support both modes, letting developers choose based on their current task.

### Foundation Package Structure

```
foundation-example/
├── src/
│   ├── index.js           # Main entry (no CSS import)
│   ├── entry-runtime.js   # Build entry (includes CSS)
│   ├── styles.css         # Tailwind source
│   └── components/
│       └── *.jsx
├── dist/
│   ├── foundation.js      # Built bundle
│   └── assets/
│       └── style.css      # Built CSS
└── package.json
```

### Package Exports Strategy

```json
{
  "exports": {
    ".": "./src/index.js",
    "./styles": "./src/styles.css",
    "./dist": "./dist/foundation.js",
    "./dist/styles": "./dist/assets/style.css"
  }
}
```

This allows consumers to choose:

- `import 'foundation'` → Source JS
- `import 'foundation/dist'` → Built JS
- `import 'foundation/styles'` → Source CSS
- `import 'foundation/dist/styles'` → Built CSS

### Why Separate Entry Points?

**Problem:** In standard import mode, we want source JS (for HMR) but built CSS (for Tailwind classes).

If `index.js` imports CSS directly:

```js
// This couples JS and CSS
import "./styles.css";
export function getComponent() {
  /* ... */
}
```

Then we can't import JS without also processing CSS through Vite's pipeline, which won't have the right Tailwind config.

**Solution:** Separate entries:

```js
// src/index.js - JS only, no CSS
export function getComponent() {
  /* ... */
}

// src/entry-runtime.js - For bundled builds
import "./styles.css";
export * from "./index.js";
```

Now:

- Standard import: `import('foundation')` + `import('foundation/dist/styles')`
- Runtime loading: Uses `entry-runtime.js` which bundles CSS

### CSS Handling

#### The Tailwind Challenge

Tailwind generates CSS by scanning source files for class names. In standard import mode:

```
Site's Vite                    Foundation
     │                              │
     ├── Processes CSS ◀────────────┤ src/styles.css
     │   with site's Tailwind       │ (@tailwind utilities)
     │   config                     │
     ▼                              │
  Missing classes!                  │ components/*.jsx
  (didn't scan foundation)          │ (contain class names)
```

#### Solution: Pre-built CSS for Standard Mode

```
┌─────────────────────────────────────────────────────────┐
│  Build step (one-time):                                 │
│                                                         │
│  Foundation's Vite ──▶ Scans src/ ──▶ dist/style.css   │
│  with foundation's       for Tailwind    (all classes)  │
│  Tailwind config         classes                        │
└─────────────────────────────────────────────────────────┘

┌─────────────────────────────────────────────────────────┐
│  Standard import mode:                                  │
│                                                         │
│  import('foundation')           ──▶ src/index.js (HMR) │
│  import('foundation/dist/styles') ──▶ Pre-built CSS    │
└─────────────────────────────────────────────────────────┘
```

### Mode Selection

Environment variable controls mode at build time:

```js
// vite.config.js
const useRuntimeLoading = process.env.VITE_FOUNDATION_MODE === "runtime";

export default defineConfig({
  plugins: [
    react(),
    svgr(),
    siteContentPlugin({ /* ... */ }),
    useRuntimeLoading && foundationPlugin({ /* ... */ }),
  ].filter(Boolean),
});
```

```js
// main.jsx
const useRuntimeLoading = import.meta.env.VITE_FOUNDATION_MODE === "runtime";

if (useRuntimeLoading) {
  initRuntime({ url: "/foundation/foundation.js" /* ... */ });
} else {
  const foundation = await import("foundation-example");
  await import("foundation-example/dist/styles");
  initRuntime(foundation);
}
```

## Trade-offs

### Standard Import Mode

**Pros:**

- Instant HMR for component changes
- Full IDE integration
- Easier debugging
- Faster startup (no build step)

**Cons:**

- CSS changes require rebuild
- Doesn't test production loading path
- Requires svgr plugin in site config

### Runtime Loading Mode

**Pros:**

- Tests exact production behavior
- Auto-rebuilds on any source change
- No additional site config needed

**Cons:**

- Slower iteration (rebuild + reload)
- More complex transform pipeline
- Full page reload on changes

## Future Improvements

1. **CSS HMR in Standard Mode** - Watch foundation source and auto-rebuild CSS
2. **Shared Tailwind Config** - Site inherits foundation's Tailwind setup
3. **Hot Module Replacement in Runtime Mode** - Partial updates instead of full reload
