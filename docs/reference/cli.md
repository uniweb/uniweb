# Uniweb CLI Reference

The Uniweb CLI helps you create and build Uniweb projects.

## Installation

```bash
# Use directly with npx
npx uniweb <command>

# Or install globally
npm install -g uniweb
uniweb <command>
```

## Commands

### `create`

Create a new Uniweb project.

```bash
uniweb create [project-name] [options]
```

**Options:**

| Option | Description |
|--------|-------------|
| `--template <type>` | Project template (see Templates below) |

**Examples:**

```bash
# Interactive prompts
uniweb create

# Create with specific name
uniweb create my-project

# Create with specific template
uniweb create my-project --template single
uniweb create my-project --template marketing
uniweb create my-project --template github:myorg/template
```

**Templates:**

| Template | Description |
|----------|-------------|
| `single` | One site + one foundation in `site/` and `foundation/` (default) |
| `multi` | Multiple sites and foundations in `sites/*` and `foundations/*` |
| `marketing` | Official marketing template with Hero, Features, Pricing, etc. |
| `@scope/name` | Install template from npm package |
| `github:user/repo` | Install template from GitHub repository |

### `build`

Build the current project.

```bash
uniweb build [options]
```

**Options:**

| Option | Description |
|--------|-------------|
| `--target <type>` | Build target: `foundation` or `site` (auto-detected if not specified) |
| `--prerender` | Pre-render pages to static HTML (SSG) - site builds only |
| `--foundation-dir <path>` | Path to foundation directory (for prerendering) |

**Examples:**

```bash
# Auto-detect and build
uniweb build

# Explicitly build as foundation
uniweb build --target foundation

# Explicitly build as site
uniweb build --target site

# Build site with SSG pre-rendering
uniweb build --prerender

# SSG with custom foundation path
uniweb build --prerender --foundation-dir ../my-foundation
```

**Foundation Build Process:**

1. Discovers components from `src/components/*/meta.js`
2. Generates entry point (`_entry.generated.js`)
3. Runs Vite build
4. Processes preview images (converts to WebP)
5. Generates `schema.json`

**Foundation Build Output:**

```
dist/
├── foundation.js       # Bundled components
├── foundation.js.map   # Source map
├── schema.json         # Component metadata
└── assets/
    ├── style.css       # Compiled CSS
    └── [Component]/    # Preview images
        └── [preset].webp
```

**Site Build Output (Standard):**

```
dist/
├── index.html          # SPA shell
├── site-content.json   # Page content
└── assets/
    └── *.js, *.css     # Bundled app
```

**Site Build Output (with `--prerender`):**

```
dist/
├── index.html          # Pre-rendered home page
├── about/
│   └── index.html      # Pre-rendered about page
├── contact/
│   └── index.html      # Pre-rendered contact page
├── site-content.json   # Page content (for hydration)
└── assets/
    └── *.js, *.css     # Bundled app (for hydration)
```

### `--help`

Show help information.

```bash
uniweb --help
uniweb -h
```

## Project Detection

The CLI auto-detects project type based on directory structure:

| Directory | Detected As |
|-----------|-------------|
| Has `src/components/` | Foundation |
| Has `pages/` | Site |

## Environment Variables

| Variable | Description |
|----------|-------------|
| `NODE_ENV` | Set to `production` for optimized builds |
| `DEBUG` | Set to show verbose error output |

## Generated Files

### `_entry.generated.js`

Auto-generated foundation entry point. **Do not edit manually.**

Add to `.gitignore`:
```
_entry.generated.js
```

### `schema.json`

Generated metadata file containing:
- Foundation-level configuration
- All component metadata
- Preview image references with dimensions

## Exit Codes

| Code | Meaning |
|------|---------|
| 0 | Success |
| 1 | Error (invalid arguments, build failure, etc.) |
