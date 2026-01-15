# Runtime Linking

This document explains how foundations are loaded at runtime using native ES modules and import maps.

## Overview

When a site uses runtime linking:

1. The host provides shared dependencies (React) via an import map
2. The foundation is loaded dynamically via `import()`
3. The foundation's React imports resolve to the host's React
4. The runtime renders components with content

```
┌─────────────────────────────────────────────────────────────┐
│                         Browser                             │
├─────────────────────────────────────────────────────────────┤
│  Import Map                                                 │
│  ┌─────────────────────────────────────────────────────┐    │
│  │ "react" → https://cdn.example.com/react.mjs         │    │
│  │ "react-dom" → https://cdn.example.com/react-dom.mjs │    │
│  └─────────────────────────────────────────────────────┘    │
│                              ↑                              │
│                              │ resolves                     │
│  Foundation Module           │                              │
│  ┌─────────────────────────────────────────────────────┐    │
│  │ import React from 'react'  ←─────────────────────────    │
│  │ export function Hero({ content }) { ... }           │    │
│  └─────────────────────────────────────────────────────┘    │
│                              ↑                              │
│                              │ loads                        │
│  Runtime                     │                              │
│  ┌─────────────────────────────────────────────────────┐    │
│  │ const foundation = await import(foundationUrl)      │    │
│  │ const Hero = foundation.getComponent('Hero')        │    │
│  │ render(<Hero content={pageContent} />)              │    │
│  └─────────────────────────────────────────────────────┘    │
└─────────────────────────────────────────────────────────────┘
```

## Import Maps Explained

Import maps tell the browser how to resolve bare module specifiers. Without an import map, `import React from 'react'` would fail because the browser doesn't know where to find `'react'`.

```html
<script type="importmap">
  {
    "imports": {
      "react": "https://esm.sh/react@18.2.0",
      "react-dom": "https://esm.sh/react-dom@18.2.0",
      "react-dom/client": "https://esm.sh/react-dom@18.2.0/client",
      "react/jsx-runtime": "https://esm.sh/react@18.2.0/jsx-runtime"
    }
  }
</script>
```

### Key Points

- Import maps must appear before any `type="module"` scripts
- They apply to all modules loaded afterward
- The browser caches resolved modules, so React is loaded once
- All foundations using `import 'react'` get the same instance

### Browser Support

Import maps are supported in all modern browsers (Chrome 89+, Firefox 108+, Safari 16.4+). For older browsers, use the [es-module-shims](https://github.com/guybedford/es-module-shims) polyfill.

## Loading a Foundation

```javascript
async function loadFoundation(url) {
  const foundation = await import(url);
  console.log("Available components:", foundation.listComponents());
  return foundation;
}

// Usage
const foundation = await loadFoundation(
  "https://cdn.example.com/my-foundation/foundation.js"
);
const Hero = foundation.getComponent("Hero");
```

### Error Handling

```javascript
async function loadFoundation(url) {
  try {
    const foundation = await import(url);

    if (typeof foundation.getComponent !== "function") {
      throw new Error("Invalid foundation: missing getComponent");
    }

    return foundation;
  } catch (error) {
    if (error instanceof TypeError) {
      console.error("Network error loading foundation:", url);
    } else if (error instanceof SyntaxError) {
      console.error("Foundation has syntax errors:", url);
    }
    throw error;
  }
}
```

## Rendering with Content

The runtime loads content (from JSON, markdown, or API) and passes it to components:

```javascript
// content.json
{
  "sections": [
    {
      "component": "Hero",
      "content": {
        "title": "Welcome",
        "subtitle": "Hello world"
      }
    }
  ]
}
```

```javascript
function renderPage(foundation, content) {
  return content.sections.map((section, i) => {
    const Component = foundation.getComponent(section.component);

    if (!Component) {
      console.warn(`Unknown component: ${section.component}`);
      return null;
    }

    return <Component key={i} content={section.content} />;
  });
}
```

## React Version Compatibility

Since the host controls the React version via the import map, foundations must be compatible with that version.

### Recommended Approach

1. Document the supported React version range in foundation metadata
2. Test foundations against the host's React version
3. Use only stable React APIs, avoid experimental features

### If Versions Conflict

If a foundation requires React 19 but the host has React 18:

- The foundation may work if it only uses APIs present in both
- Features missing in React 18 will throw errors
- There's no automatic fallback (unlike Module Federation)

## CDN Options for Shared Deps

### esm.sh (Recommended)

```javascript
"react": "https://esm.sh/react@18.2.0"
```

Pros: No build step, version pinning, TypeScript types
Cons: External dependency, cold start latency

### Self-Hosted

Build and host ESM versions yourself:

```javascript
"react": "https://cdn.yoursite.com/shared/react@18.2.0.mjs"
```

Pros: Full control, no external dependency
Cons: Build and hosting overhead

## Multiple Foundations

A page can load multiple foundations:

```javascript
const [marketing, docs] = await Promise.all([
  import("https://cdn.example.com/marketing-foundation/foundation.js"),
  import("https://cdn.example.com/docs-foundation/foundation.js"),
]);

// Both foundations share the same React instance
const Hero = marketing.getComponent("Hero");
const CodeBlock = docs.getComponent("CodeBlock");
```

## Caching Considerations

### Browser Caching

Foundations should be served with appropriate cache headers:

```
Cache-Control: public, max-age=31536000, immutable
```

Use content hashing in filenames for cache busting:

```
foundation.abc123.js
```

### Import Map Updates

To update a foundation without changing the HTML:

1. Deploy new foundation with new hash
2. Update a manifest file that the runtime fetches
3. Runtime constructs import map dynamically

```javascript
const manifest = await fetch("/manifest.json").then((r) => r.json());

const importMap = {
  imports: {
    react: manifest.shared.react,
    "react-dom": manifest.shared.reactDom,
  },
};

const script = document.createElement("script");
script.type = "importmap";
script.textContent = JSON.stringify(importMap);
document.head.appendChild(script);
```

## Debugging

### Common Issues

**"Failed to resolve module specifier 'react'"**

- Import map is missing or malformed
- Import map appears after module scripts

**"Cannot read properties of undefined"**

- React version mismatch
- Foundation using API not in host's React

**"Module not found"**

- Foundation URL is incorrect
- CORS blocking the request

### Debugging Tools

- Browser DevTools → Network tab shows module loading
- Browser DevTools → Console shows import map errors
- Browser DevTools → Sources shows loaded modules

## Security Considerations

- Only load foundations from trusted sources
- Use Subresource Integrity (SRI) for critical dependencies
- Consider CSP headers for module sources

```html
<script type="importmap">
  {
    "imports": {
      "react": "https://esm.sh/react@18.2.0"
    },
    "integrity": {
      "https://esm.sh/react@18.2.0": "sha384-..."
    }
  }
</script>
```
