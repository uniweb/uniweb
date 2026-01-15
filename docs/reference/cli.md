# Uniweb CLI Reference

The Uniweb CLI helps you create and build Uniweb projects.

## Installation

```bash
# Use directly with npx
npx uniweb <command>

# Or install globally
npm install -g @uniweb/cli
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
| `--template <type>` | Project template: `workspace`, `site`, or `foundation` |

**Examples:**

```bash
# Interactive prompts
uniweb create

# Create with specific name
uniweb create my-project

# Create specific template
uniweb create my-site --template site
uniweb create my-foundation --template foundation
uniweb create my-workspace --template workspace
```

**Templates:**

| Template | Description |
|----------|-------------|
| `workspace` | Monorepo with site + foundation for co-development |
| `site` | Standalone site using an existing foundation |
| `foundation` | Standalone foundation (component library) |

### `build`

Build the current project.

```bash
uniweb build [options]
```

**Options:**

| Option | Description |
|--------|-------------|
| `--target <type>` | Build target: `foundation` or `site` (auto-detected if not specified) |

**Examples:**

```bash
# Auto-detect and build
uniweb build

# Explicitly build as foundation
uniweb build --target foundation

# Explicitly build as site
uniweb build --target site
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
├── schema.json         # Component metadata for editor
└── assets/
    ├── style.css       # Compiled CSS
    └── [Component]/    # Preview images
        └── [preset].webp
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
