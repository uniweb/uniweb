# Dual-Mode Foundation Development

One of Uniweb's key advantages is the ability to develop foundations with the full power of modern tooling while maintaining production-ready runtime loading.

## The Best of Both Worlds

During development, you want:

- **Instant feedback** - Changes appear immediately
- **Full HMR** - No page reloads, state preserved
- **IDE integration** - Autocomplete, go-to-definition, refactoring
- **Easy debugging** - Direct source maps, clear stack traces

In production, you need:

- **Independent deployment** - Update foundations without rebuilding sites
- **Shared foundations** - One foundation serves multiple sites
- **Runtime flexibility** - A/B testing, gradual rollouts

Uniweb gives you both.

## Quick Start

```bash
# Standard Import Mode (recommended for development)
pnpm dev

# Runtime Loading Mode (simulates production)
pnpm dev:runtime
```

## Standard Import Mode

**When to use:** Day-to-day development, building new components, debugging.

In this mode, the foundation is imported as a regular workspace dependency. Vite handles everything natively:

- Edit a component → See changes instantly (HMR)
- Cmd/Ctrl+Click on component → Jump to source
- TypeScript errors → Shown inline in your editor
- Breakpoints → Work directly in browser DevTools

```js
// What happens under the hood
const foundation = await import("foundation-example"); // Source code
await import("foundation-example/dist/styles"); // Pre-built CSS
initRuntime(foundation);
```

### One-Time Setup

Before using standard import mode, build the foundation CSS once:

```bash
pnpm build:foundation
```

This generates the Tailwind CSS with all utility classes. You only need to rebuild if you add new Tailwind classes.

## Runtime Loading Mode

**When to use:** Testing production behavior, debugging loading issues, CI/CD.

In this mode, the foundation is built and served as a separate module, exactly like production:

- Foundation rebuilt automatically on source changes
- Tests the actual runtime loading path
- Validates import map / module resolution
- Catches issues that only appear in production

```bash
# Start in runtime loading mode
pnpm dev:runtime

# Or set the environment variable directly
VITE_FOUNDATION_MODE=runtime pnpm dev
```

## Choosing the Right Mode

| Scenario                                   | Recommended Mode |
| ------------------------------------------ | ---------------- |
| Building new components                    | Standard Import  |
| Styling and layout work                    | Standard Import  |
| Debugging component logic                  | Standard Import  |
| Testing production behavior                | Runtime Loading  |
| Debugging "works locally, breaks in prod"  | Runtime Loading  |
| CI/CD integration tests                    | Both             |
| Performance profiling                      | Runtime Loading  |

## How It Works

### Standard Import Mode

```
┌─────────────────────────────────────────┐
│           Site Dev Server               │
│                                         │
│  main.jsx                               │
│     │                                   │
│     ├──▶ import('foundation-example')   │
│     │         │                         │
│     │         ▼                         │
│     │    foundation/src/index.js  ◀── HMR
│     │                                   │
│     └──▶ import('foundation/dist/styles')
│                   │                     │
│                   ▼                     │
│          Pre-built Tailwind CSS        │
└─────────────────────────────────────────┘
```

### Runtime Loading Mode

```
┌─────────────────────────────────────────┐
│           Site Dev Server               │
│                                         │
│  main.jsx                               │
│     │                                   │
│     └──▶ import('/foundation/foundation.js')
│                   │                     │
│                   ▼                     │
│  ┌─────────────────────────────────┐   │
│  │     Foundation Plugin           │   │
│  │  • Builds foundation on start   │   │
│  │  • Watches for changes          │   │
│  │  • Transforms imports via Vite  │   │
│  │  • Serves from /foundation/     │   │
│  └─────────────────────────────────┘   │
└─────────────────────────────────────────┘
```

## FAQ

### Why do I need to build CSS for standard import mode?

The foundation's CSS uses Tailwind, which scans source files to generate only the classes you use. When Vite processes the CSS in the site context, it doesn't know about the foundation's source files. Pre-building ensures all classes are included.

### Can I have HMR for CSS too?

In standard import mode, CSS changes require rebuilding (`pnpm build:foundation`). For rapid CSS iteration, use runtime loading mode which rebuilds automatically.

### What about TypeScript?

Both modes work with TypeScript. Standard import mode gives you better IDE integration since you're importing source directly.

### How do I add a new Tailwind class?

1. Add the class to your component
2. Run `pnpm build:foundation` to regenerate CSS
3. Refresh the page

Or use runtime loading mode where CSS rebuilds automatically.
