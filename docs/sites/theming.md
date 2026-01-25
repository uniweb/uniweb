# Site Theming

Uniweb sites can be customized through a `theme.yml` file that defines colors, typography, and appearance settings. The theming system generates CSS custom properties at build time.

## Quick Start

Create a `theme.yml` file in your site root:

```yaml
# site/theme.yml
colors:
  primary: "#3b82f6"
  neutral: "#64748b"
```

That's it! Uniweb automatically generates a complete color palette with 11 shades for each color.

## Color Palettes

### Basic Usage

Provide a single base color and Uniweb generates all shades:

```yaml
colors:
  primary: "#3b82f6"      # Your brand color
  secondary: "#10b981"    # Accent color
  neutral: "#64748b"      # Grays for text, backgrounds
```

This generates CSS variables like:
```css
:root {
  --primary-50: oklch(97.0% 0.0285 260.0);
  --primary-100: oklch(93.0% 0.0475 260.0);
  /* ... */
  --primary-500: oklch(55.0% 0.1900 260.0);
  /* ... */
  --primary-950: oklch(14.0% 0.0855 260.0);
}
```

### Generation Modes

Control how shades are generated with the `mode` option:

```yaml
colors:
  primary:
    base: "#3b82f6"
    mode: natural     # 'fixed', 'natural', or 'vivid'
```

#### Fixed Mode (Default)

Predictable, consistent results ideal for design systems.

- **Constant hue** across all shades
- **Fixed lightness values** for each shade level
- **Predictable contrast ratios** for accessibility

```yaml
colors:
  primary: "#3b82f6"  # Uses fixed mode by default
```

Best for: Design systems, accessibility-focused projects, consistent branding.

#### Natural Mode

Creates more organic palettes with temperature-aware adjustments.

- **Hue shifts** based on color temperature:
  - Warm colors (reds, oranges): Lighter shades shift cooler, darker shift warmer
  - Cool colors (blues, greens): Opposite behavior
- **Curved chroma** using Bézier interpolation
- **Slight saturation boost** (1.1x) at middle shades

```yaml
colors:
  primary:
    base: "#3b82f6"
    mode: natural
```

Best for: Marketing sites, creative projects, organic brand aesthetics.

#### Vivid Mode

High-saturation palettes for bold, dramatic designs.

- **Higher chroma boost** (1.4x at peak)
- **More saturation preserved** at light and dark extremes
- **Subtle hue shifts** for depth
- **Gamut-mapped** to prevent color clipping

```yaml
colors:
  primary:
    base: "#f97316"
    mode: vivid
```

Best for: Gaming sites, entertainment, bold marketing campaigns.

### Exact Color Matching

Guarantee your exact brand color appears at shade 500:

```yaml
colors:
  brand:
    base: "#E31937"    # Exact brand color
    exactMatch: true   # Shade 500 = exact input
```

Without `exactMatch`, shade 500 uses the algorithm's calculated lightness, which may differ slightly from your input.

### Pre-defined Shades

For complete control, define all shades manually:

```yaml
colors:
  corporate:
    50: "#fef2f2"
    100: "#fee2e2"
    200: "#fecaca"
    300: "#fca5a5"
    400: "#f87171"
    500: "#ef4444"
    600: "#dc2626"
    700: "#b91c1c"
    800: "#991b1b"
    900: "#7f1d1d"
    950: "#450a0a"
```

## Typography

Configure font families and imports:

```yaml
fonts:
  body: "Inter, sans-serif"
  heading: "Poppins, sans-serif"
  mono: "Fira Code, monospace"
  import:
    - url: "https://fonts.googleapis.com/css2?family=Inter:wght@400;500;600;700"
    - url: "https://fonts.googleapis.com/css2?family=Poppins:wght@600;700"
```

Generates:
```css
@import url('https://fonts.googleapis.com/css2?family=Inter...');
@import url('https://fonts.googleapis.com/css2?family=Poppins...');

:root {
  --font-body: Inter, sans-serif;
  --font-heading: Poppins, sans-serif;
  --font-mono: Fira Code, monospace;
}
```

## Context Classes

Define semantic color tokens for different contexts:

```yaml
contexts:
  light:
    bg: white
    text: var(--neutral-900)
    link: var(--primary-600)
  dark:
    bg: var(--neutral-900)
    text: white
    link: var(--primary-400)
```

Use in sections via the `theme` parameter:

```markdown
---
type: Hero
theme: dark
---
```

The section wrapper gets the `context-dark` class with appropriate tokens.

## Appearance (Dark Mode)

Configure light/dark mode behavior:

```yaml
appearance:
  default: light              # 'light', 'dark', or 'system'
  allowToggle: true           # Show dark mode toggle
  respectSystemPreference: true
  schemes: [light, dark]
```

## Foundation Variables

Override foundation-defined variables:

```yaml
vars:
  header-height: 5rem
  max-content-width: 72rem
```

Available variables depend on your foundation—check its `foundation.js` for options.

## Complete Example

```yaml
# site/theme.yml

colors:
  primary:
    base: "#3b82f6"
    mode: natural
  secondary: "#10b981"
  accent:
    base: "#f59e0b"
    mode: vivid
  neutral: "#64748b"

fonts:
  body: "Inter, system-ui, sans-serif"
  heading: "Cal Sans, Inter, sans-serif"
  import:
    - url: "https://fonts.googleapis.com/css2?family=Inter:wght@400;500;600;700"

contexts:
  light:
    bg: white
    text: var(--neutral-950)
    muted: var(--neutral-500)
    link: var(--primary-600)
  dark:
    bg: var(--neutral-950)
    text: var(--neutral-50)
    muted: var(--neutral-400)
    link: var(--primary-400)

appearance:
  default: system
  allowToggle: true

vars:
  header-height: 4rem
```

## Using Theme Values

### In Components (React)

```jsx
import { useThemeData, useThemeColor } from '@uniweb/kit'

function MyComponent() {
  // Get a specific color
  const primaryColor = useThemeColor('primary', 500)

  // Get CSS variable reference
  const colorVar = useThemeColorVar('primary', 600) // 'var(--primary-600)'

  // Access full theme
  const theme = useThemeData()
  const palette = theme?.getPalette('primary')
}
```

### In CSS

```css
.my-element {
  background: var(--primary-500);
  color: var(--text);
  border: 1px solid var(--border);
}

/* Dark mode override */
.context-dark .my-element {
  background: var(--primary-400);
}
```

## How Shade Generation Works

Uniweb uses the **OKLCH color space** for perceptually uniform shade generation.

### Why OKLCH?

Unlike RGB or HSL, OKLCH produces shades where:
- Equal lightness steps look equally different to human eyes
- Colors don't shift hue unexpectedly at extremes
- High saturation colors don't clip or distort

### The Algorithm

1. **Parse** input color to OKLCH (Lightness, Chroma, Hue)
2. **Apply lightness** for each shade level (97% for 50, down to 14% for 950)
3. **Scale chroma** to prevent clipping at extremes
4. **Shift hue** (natural/vivid modes only) based on color temperature
5. **Gamut map** to ensure valid sRGB output

### Shade Lightness Values

| Shade | Lightness | Description |
|-------|-----------|-------------|
| 50 | 97% | Very light |
| 100 | 93% | |
| 200 | 87% | |
| 300 | 78% | |
| 400 | 68% | |
| 500 | 55% | Base (most vibrant) |
| 600 | 48% | |
| 700 | 40% | |
| 800 | 32% | |
| 900 | 24% | |
| 950 | 14% | Very dark |

## Related

- [Content Structure](../content-structure.md) - Page and section organization
- [Component Metadata](../component-metadata.md) - Component params including theme support
