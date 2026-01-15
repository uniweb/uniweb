# Foundation Development Guide

This guide covers how to create and structure foundations (component libraries) for the Uniweb framework.

## Overview

A **foundation** is a collection of React components that can be used to render content-driven websites. Foundations are:

- Built with Vite and distributed as ES modules
- Discovered and loaded by the Uniweb runtime
- Configured through metadata files (`meta.js`)
- Published with schema information for the Uniweb visual editor

## Project Structure

```
my-foundation/
├── package.json
├── vite.config.js
├── tailwind.config.js
├── postcss.config.js
├── src/
│   ├── meta.js                    # Foundation-level metadata
│   ├── index.js                   # Manual entry (for dev)
│   ├── entry-runtime.js           # Runtime entry (includes CSS)
│   ├── styles.css                 # Global styles (Tailwind)
│   ├── icons/                     # Shared icons/assets
│   │   └── arrow-right.svg
│   └── components/
│       ├── Hero/
│       │   ├── index.jsx          # Component implementation
│       │   ├── meta.js            # Component metadata
│       │   └── previews/          # Preset preview images
│       │       ├── default.png
│       │       └── dark.png
│       ├── Features/
│       │   ├── index.jsx
│       │   └── meta.js
│       └── FAQ/
│           ├── index.jsx
│           └── meta.js
└── dist/                          # Build output
    ├── foundation.js
    ├── schema.json
    └── assets/
        ├── style.css
        └── Hero/
            ├── default.webp
            └── dark.webp
```

## Component Structure

Each component lives in its own folder under `src/components/`:

```
components/
└── Hero/
    ├── index.jsx      # Required: Component implementation
    ├── meta.js        # Required: Component metadata
    ├── styles.css     # Optional: Component-specific styles
    └── previews/      # Optional: Preset preview images
```

### Component Implementation (`index.jsx`)

Components receive a `content` prop with data from the CMS:

```jsx
import React from "react";

export function Hero({ content }) {
  const { title, subtitle, ctaText, ctaUrl } = content;

  return (
    <section className="py-20 px-6 bg-gradient-to-br from-primary to-blue-700 text-white">
      <div className="max-w-4xl mx-auto text-center">
        <h1 className="text-4xl font-bold mb-6">{title}</h1>
        {subtitle && <p className="text-lg mb-8">{subtitle}</p>}
        {ctaText && ctaUrl && (
          <a
            href={ctaUrl}
            className="px-6 py-3 bg-white text-primary rounded-lg"
          >
            {ctaText}
          </a>
        )}
      </div>
    </section>
  );
}

export default Hero;
```

### Component Metadata (`meta.js`)

The `meta.js` file defines how the component appears in the Uniweb editor:

```js
export default {
  // Display information
  title: "Hero Banner",
  description: "A prominent header section with headline and call-to-action",
  category: "Headers",

  // Content elements (semantic content structure)
  elements: {
    title: {
      label: "Headline",
      description: "The main headline text",
      required: true,
    },
    subtitle: {
      label: "Subtitle",
      description: "Supporting text below the headline",
    },
    links: {
      label: "Call to Action",
      description: "Button linking to another page",
    },
  },

  // Configurable properties
  properties: {
    alignment: {
      type: "select",
      label: "Text Alignment",
      options: [
        { value: "center", label: "Center" },
        { value: "left", label: "Left" },
      ],
      default: "center",
    },
    backgroundImage: {
      type: "image",
      label: "Background Image",
    },
  },

  // Named presets (pre-configured styles)
  presets: {
    default: {
      label: "Default",
      description: "Standard gradient background",
      properties: {},
    },
    dark: {
      label: "Dark Theme",
      description: "Dark background with light text",
      properties: { theme: "dark" },
    },
  },

  // Runtime configuration (extracted into foundation.js)
  input: {
    type: "reference",
    limit: 3,
  },

  // Set to false to hide from editor (internal component)
  // exposed: false,
};
```

## Metadata Reference

### Component `meta.js` Properties

| Property      | Type    | Description                                      |
| ------------- | ------- | ------------------------------------------------ |
| `title`       | string  | Display name in editor                           |
| `description` | string  | Brief description of the component               |
| `category`    | string  | Grouping category (e.g., "Headers", "Content")   |
| `elements`    | object  | Semantic content structure                       |
| `items`       | object  | For repeatable content (arrays)                  |
| `properties`  | object  | Configurable options                             |
| `presets`     | object  | Named preset configurations                      |
| `input`       | object  | Runtime input schema (for live data)             |
| `exposed`     | boolean | Whether component appears in editor (default: true) |

### Property Types

```js
properties: {
  // Select dropdown
  alignment: {
    type: 'select',
    label: 'Alignment',
    options: [
      { value: 'left', label: 'Left' },
      { value: 'center', label: 'Center' },
    ],
    default: 'center',
  },

  // Boolean toggle
  showBorder: {
    type: 'boolean',
    label: 'Show Border',
    default: false,
  },

  // Color picker
  accentColor: {
    type: 'color',
    label: 'Accent Color',
    default: '#3b82f6',
  },

  // Image upload
  backgroundImage: {
    type: 'image',
    label: 'Background',
  },

  // Number input
  columns: {
    type: 'number',
    label: 'Columns',
    min: 1,
    max: 6,
    default: 3,
  },
}
```

### Items (Repeatable Content)

For components with repeatable items (like Features or FAQ):

```js
export default {
  title: "Features Grid",
  category: "Content",

  elements: {
    title: { label: "Section Title" },
    subtitle: { label: "Section Subtitle" },
  },

  items: {
    label: "Features",
    description: "Individual feature cards",
    elements: {
      title: { label: "Feature Title", required: true },
      description: { label: "Feature Description", required: true },
      icon: { label: "Icon", type: "icon" },
    },
  },
};
```

## Foundation-Level Metadata

The `src/meta.js` file defines foundation-wide configuration:

```js
export default {
  name: "My Foundation",
  description: "A collection of components for marketing sites",

  // Default language for plain strings (default: 'en')
  defaultLanguage: "en",

  // Runtime props (available at render time)
  props: {
    themeToggleEnabled: true,
    markdownCopyEnabled: true,
  },

  // Foundation-wide style fields
  styleFields: [
    {
      id: "primary-color",
      type: "color",
      label: "Primary Color",
      description: "Main brand color",
      default: "#3b82f6",
    },
    {
      id: "border-radius",
      type: "select",
      label: "Corner Style",
      options: [
        { value: "0px", label: "Sharp" },
        { value: "0.5rem", label: "Rounded" },
      ],
      default: "0.5rem",
    },
  ],
};
```

## Preview Images

Preview images help users visualize presets in the editor.

### Convention

Place images in `components/[Name]/previews/`:

```
components/
└── Hero/
    └── previews/
        ├── default.png    # Preview for "default" preset
        ├── dark.png       # Preview for "dark" preset
        └── minimal.png    # Preview for "minimal" preset
```

### Requirements

- **Format**: PNG or JPG (converted to WebP during build)
- **Naming**: Must match preset name (e.g., `dark.png` for `presets.dark`)
- **Size**: Recommended 400x300px (or similar aspect ratio)
- **Content**: Should show realistic component appearance

## Build Process

### Commands

```bash
# Build foundation (auto-detects project type)
uniweb build

# Explicitly build as foundation
uniweb build --target foundation
```

### What Happens During Build

1. **Component Discovery**: Scans `src/components/*/meta.js`
2. **Entry Generation**: Creates `_entry.generated.js` with all components
3. **Vite Build**: Bundles components to `dist/foundation.js`
4. **Image Processing**: Converts preview images to WebP
5. **Schema Generation**: Outputs `dist/schema.json`

### Build Output

```
dist/
├── foundation.js           # Bundled components (~6KB typical)
├── foundation.js.map       # Source map
├── schema.json             # Full metadata for Uniweb app
└── assets/
    ├── style.css           # Compiled Tailwind CSS
    └── [Component]/        # Preview images per component
        └── [preset].webp
```

## Runtime vs Editor Metadata

Not all metadata is bundled in the JavaScript:

| Data                       | In `foundation.js` | In `schema.json` |
| -------------------------- | ------------------ | ---------------- |
| Component code             | Yes                | No               |
| `input` schema             | Yes                | Yes              |
| `props`                    | Yes                | Yes              |
| `title`, `description`     | No                 | Yes              |
| `elements`, `properties`   | No                 | Yes              |
| `presets`                  | No                 | Yes              |
| Preview images             | No                 | Yes (paths)      |

This keeps the JavaScript bundle small (~6KB) while providing full metadata for the editor.

## Best Practices

### Component Design

1. **Accept `content` prop**: All data comes through the `content` prop
2. **Handle missing data**: Use defaults and conditional rendering
3. **Use Tailwind**: Consistent styling with the design system
4. **Keep components focused**: One component = one purpose

### Metadata

1. **Write clear descriptions**: Help users understand each component
2. **Use meaningful categories**: Group related components together
3. **Provide sensible defaults**: Properties should have good defaults
4. **Create useful presets**: Common configurations users will want

### File Organization

1. **One component per folder**: Keep related files together
2. **Use index.jsx**: Default export from `index.jsx` or `index.js`
3. **Colocate assets**: Component-specific assets in the component folder
4. **Share global assets**: Put shared icons/images in `src/icons/` or `src/assets/`
