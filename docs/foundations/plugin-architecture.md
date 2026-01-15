# Foundation Plugin Architecture

This document explains how the `foundationPlugin` serves pre-built foundation modules within Vite's dev server, and the challenges we solved to make it work.

## The Challenge

In the Uniweb architecture, **foundations** are React component libraries that are:

1. Built separately from the site
2. Loaded at runtime via dynamic `import()`
3. Must share the same React instance as the host application

In production, this is straightforward: use browser import maps to map `react` and `react-dom` to CDN URLs, and the foundation's bare imports resolve correctly.

In development with Vite, this becomes complex because:

- Vite pre-bundles dependencies into `.vite/deps/`
- These pre-bundled modules use CJS-style default exports
- The foundation is pre-built with ESM named imports
- Dynamic imports from URLs don't go through Vite's module resolution

## Key Problems & Solutions

### Problem 1: React Bundled Into Foundation

**Symptom:** Foundation bundle was ~50KB instead of ~7KB.

**Cause:** The foundation's `vite.config.js` externalized `react` and `react/jsx-runtime`, but not `react/jsx-dev-runtime` which Vite's React plugin uses in development.

**Solution:** Add all React-related modules to externals:

```js
// foundation's vite.config.js
rollupOptions: {
  external: [
    'react',
    'react-dom',
    'react/jsx-runtime',
    'react/jsx-dev-runtime'  // <-- This was missing
  ],
}
```

### Problem 2: Bare Specifiers Don't Resolve in Browser

**Symptom:** Browser error "Failed to resolve module specifier 'react'"

**Cause:** The foundation's built output contains:

```js
import { jsxDEV } from "react/jsx-dev-runtime";
import { useState } from "react";
```

Browsers can't resolve bare specifiers like `"react"` without an import map.

**Solution:** Use Vite's `transformRequest` API to properly transform the foundation through Vite's pipeline:

```js
// In the middleware
if (filePath.endsWith(".js")) {
  const result = await devServer.transformRequest(`/@fs${fullPath}`, {
    html: false,
  });
  if (result) {
    content = result.code;
  }
}
```

This transforms the foundation's imports into Vite's interop format:

```js
// Before (foundation's output)
import { jsxDEV } from "react/jsx-dev-runtime";
import { useState } from "react";

// After (Vite's transform)
import __vite__cjsImport from "/node_modules/.vite/deps/react_jsx-dev-runtime.js";
const jsxDEV = __vite__cjsImport["jsxDEV"];

import __vite__cjsImport2 from "/node_modules/.vite/deps/react.js";
const useState = __vite__cjsImport2["useState"];
```

### Problem 3: Tailwind Classes Not Generated

**Symptom:** CSS loaded but styles not applied (no margins, padding, etc.)

**Cause:** Tailwind's `content` config used relative paths:

```js
content: ["./src/**/*.{js,jsx}"];
```

When the foundation is built by the parent plugin (from a different working directory), these paths don't resolve.

**Solution:** Use absolute paths in Tailwind config:

```js
import { fileURLToPath } from "url";
import { dirname, join } from "path";

const __dirname = dirname(fileURLToPath(import.meta.url));

export default {
  content: [join(__dirname, "./src/**/*.{js,jsx,ts,tsx}")],
  // ...
};
```

## Architecture Overview

```
┌──────────────────────────────────────────────────────────────┐
│                     Site Dev Server                          │
│                     (Vite @ :3001)                           │
├──────────────────────────────────────────────────────────────┤
│                                                              │
│  ┌─────────────┐    ┌──────────────┐    ┌───────────────┐    │
│  │   Site App  │    │  Foundation  │    │  Vite's React │    │
│  │  (main.jsx) │───▶│   Plugin     │───▶│  Pre-bundled  │    │
│  └─────────────┘    └──────────────┘    │    Deps       │    │
│         │                  │            └───────────────┘    │
│         │                  │                    ▲            │
│         │                  ▼                    │            │
│         │           ┌──────────────┐            │            │
│         │           │  Foundation  │────────────┘            │
│         │           │   dist/      │  (imports rewritten     │
│         │           │              │   via transformRequest) │
│         │           └──────────────┘                         │
│         │                                                    │
│         ▼                                                    │
│  import('/foundation/foundation.js')                         │
│         │                                                    │
│         ▼                                                    │
│  ┌──────────────────────────────────┐                        │
│  │         Middleware               │                        │
│  │  1. Read foundation.js from dist │                        │
│  │  2. Transform via Vite pipeline  │                        │
│  │  3. Serve with proper imports    │                        │
│  └──────────────────────────────────┘                        │
│                                                              │
└──────────────────────────────────────────────────────────────┘
```

## Key Insight

The fundamental insight is that **Vite's dev server is not just a static file server**—it's a transformation pipeline. Pre-built modules with bare specifiers must go through this pipeline to work correctly in the browser.

The `/@fs/` prefix tells Vite to load a file from the filesystem and transform it through its normal pipeline, which includes:

- Resolving bare specifiers to pre-bundled deps
- Adding ESM/CJS interop wrappers
- Injecting HMR code
- Adding React Fast Refresh

## Production Considerations

In production, the approach is different:

- Use browser import maps to resolve React
- Serve foundation from CDN
- No transformation needed (pure ESM)

```html
<script type="importmap">
  {
    "imports": {
      "react": "https://esm.sh/react@18",
      "react-dom": "https://esm.sh/react-dom@18",
      "react/jsx-runtime": "https://esm.sh/react@18/jsx-runtime"
    }
  }
</script>
```

## Dual-Mode Development

The system supports two foundation loading modes:

### 1. Standard Import Mode (Default)

```bash
pnpm dev
```

- Foundation imported as workspace dependency
- Full Vite HMR for foundation components
- IDE autocomplete and go-to-definition work
- Faster iteration, standard DX

### 2. Runtime Loading Mode

```bash
pnpm dev:runtime
# or
VITE_FOUNDATION_MODE=runtime pnpm dev
```

- Foundation built and served separately
- Simulates production behavior
- Tests the full runtime loading path
- Useful for debugging production issues

### When to Use Each Mode

| Scenario                           | Recommended Mode   |
| ---------------------------------- | ------------------ |
| Day-to-day development             | Standard Import    |
| Testing production behavior        | Runtime Loading    |
| Debugging runtime loading issues   | Runtime Loading    |
| Developing foundation components   | Standard Import    |
| CI/CD testing                      | Both               |

### Package Exports

Foundations should export both modes:

```json
// package.json
{
  "exports": {
    ".": "./src/index.js",
    "./styles": "./src/styles.css",
    "./dist": "./dist/foundation.js",
    "./dist/styles": "./dist/assets/style.css"
  }
}
```
