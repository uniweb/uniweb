# Uniweb Framework for Developers: Building Foundations

This guide explains Uniweb's component architecture and provides practical guidance for building effective, reusable Foundations. As a developer, you'll create the React components that content creators use to build websites without writing code.

**Related documentation:**

- [Understanding Uniweb](getting-started/understanding-uniweb.md) - Conceptual overview of the architecture
- [Terminology Reference](reference/terminology.md) - Key terms and definitions

## Introduction

When building Foundations for websites, you're creating more than just React components—you're building a content-facing interface that non-technical users will interact with through markdown and visual editors.

**Key insight:** Your components have two audiences:

1. **Content creators** who reference them declaratively in markdown
2. **The runtime engine (RTE)** that instantiates them with structured content

This dual interface is what makes Foundations different from traditional component libraries. See [Understanding Uniweb](getting-started/understanding-uniweb.md) for the conceptual foundation behind this architecture.

### Section Types vs Components

A foundation has two kinds of things:

1. **Section types** — the small number of components that content creators can select by name in markdown files. These have a `meta.js` file, follow the `{ content, params, block }` interface, and act as the bridge between content and code.

2. **Components** — everything else. Regular React components used within section types for rendering. Standard React patterns, any props structure, any npm packages. No special files or interfaces needed. Most of the actual UI rendering happens here.

This distinction matters: Uniweb fully supports standard React development patterns. Most of your rendering logic should live in regular components, with section types primarily handling the content/code bridge.

## Building Components

### Section Type Interface

Every section type follows this interface:

```jsx
function HeroSection({ content, params, block }) {
  // Extract content from the structured content object
  const { title, paragraphs, images } = content.main;

  // Extract parameters - defaults come from meta.js
  const { layout, theme } = params;

  // Render the content according to the parameters
  return (
    <div className={`hero-section layout-${layout} theme-${theme}`}>
      <h2>{title}</h2>
      {paragraphs.map((paragraph, index) => (
        <p key={index} dangerouslySetInnerHTML={{ __html: paragraph }} />
      ))}
      {images.length > 0 && (
        <img
          src={images[0].src}
          alt={images[0].alt || ""}
          className="hero-image"
        />
      )}
    </div>
  );
}

export default HeroSection;
```

Block configuration (context and initialState) is defined in meta.js alongside other component metadata:

```javascript
// meta.js
export default {
  title: 'Hero Section',
  description: 'Bold hero section with headline and CTA',

  // Static capabilities for cross-block coordination
  context: {
    allowTranslucentTop: true,  // Header can overlay this section
  },

  // Initial values for mutable block state (optional)
  initialState: {
    // expanded: false,
  },

  params: {
    layout: { type: 'select', options: ['standard', 'split'], default: 'standard' },
    theme: { type: 'select', options: ['light', 'dark'], default: 'light' },
  },
};
```

### Understanding the Content Object

The `content` object contains structured data parsed from markdown:

```javascript
{
  // Main content (from H1 and following paragraphs)
  main: {
    title: "Welcome to Our Platform",
    paragraphs: ["Discover how our innovative solutions..."],
    images: [{
      src: "/images/hero.jpg",
      alt: "Hero Image",
      attributes: { background: true }
    }],
    links: [{
      text: "Get Started",
      url: "#getting-started",
      attributes: { "button-primary": true }
    }],
    list: []
  },

  // Content groups from H2 headings
  items: [
    {
      title: "Feature One",
      paragraphs: ["Feature one description..."],
      images: [],
      links: [],
      list: []
    }
  ]
}
```

### Working with Parameters

Parameters come from the front matter in markdown files:

```markdown
---
type: HeroSection
layout: centered
theme: dark
showButton: true
---
```

In your component, extract and use these parameters:

```jsx
function HeroSection({ content, params, block }) {
  const { layout = "standard", theme = "light", showButton = false } = params;

  return (
    <div className={`hero layout-${layout} theme-${theme}`}>
      <h1>{content.main.title}</h1>
      {content.main.paragraphs.map((p, i) => (
        <p key={i} dangerouslySetInnerHTML={{ __html: p }} />
      ))}
      {showButton && content.main.links.length > 0 && (
        <a href={content.main.links[0].url} className="button">
          {content.main.links[0].text}
        </a>
      )}
    </div>
  );
}
```

### Working with the Block Object

The `block` object provides runtime context, state management, and cross-block communication:

```jsx
import { useState } from 'react';

function InteractiveComponent({ content, params, block }) {
  // Connect block state to React state using the bridge pattern
  const [state, setState] = block.useBlockState(useState);

  const increment = () => {
    setState({
      ...state,
      count: (state.count || 0) + 1
    });
  };

  return (
    <div>
      <h2>{content.main.title}</h2>
      <p>Count: {state.count || 0}</p>
      <button onClick={increment}>Increment</button>
    </div>
  );
}

export default InteractiveComponent;
```

Initial state is declared in meta.js using the `initialState` field:

```javascript
// meta.js
export default {
  title: 'Interactive Component',
  initialState: {
    count: 0
  },
  params: { ... },
};
```

**Block Properties:**

- `block.id` - Unique identifier for this block instance
- `block.type` - Component type (matches frontmatter `type:`)
- `block.themeName` - Theme setting (e.g., 'light', 'dark')
- `block.state` - Dynamic state (from `meta.js initialState`)
- `block.context` - Static context (from `meta.js context`)
- `block.childBlocks` - Array of child Block instances
- `block.page` - Reference to the parent Page
- `block.website` - Reference to the Website

**Block Methods:**

- `block.useBlockState(useState)` - Bridge pattern for React state sync (see below)
- `block.getChildBlockRenderer()` - Get the ChildBlocks component for rendering children
- `block.getIndex()` - Get this block's position in the page
- `block.getBlockInfo()` - Get `{ type, theme, state, context }` for cross-block communication
- `block.getNextBlockInfo()` - Get the next block's info
- `block.getPrevBlockInfo()` - Get the previous block's info

### Block State and the Bridge Pattern

Block state persists across renders and even page navigations, making it more powerful than standard React state for certain use cases.

The `useBlockState` method uses a "bridge pattern" that connects vanilla JavaScript state to React:

```jsx
function Accordion({ content, params, block }) {
  // Pass useState to the block - it manages the subscription
  const [state, setState] = block.useBlockState(useState);

  const toggleItem = (index) => {
    setState({
      ...state,
      openItem: state.openItem === index ? null : index
    });
  };

  return (
    <div className="accordion">
      {content.items.map((item, index) => (
        <div key={index} className={state.openItem === index ? 'open' : ''}>
          <button onClick={() => toggleItem(index)}>{item.title}</button>
          {state.openItem === index && <div>{item.paragraphs[0]}</div>}
        </div>
      ))}
    </div>
  );
}

// Initial state declaration
Accordion.block = {
  state: { openItem: null }
};
```

### Cross-Block Communication

Components can read information from neighboring blocks, enabling adaptive behavior without content author involvement.

Block info includes:
- **`type`** - Component type (e.g., 'Hero', 'Features')
- **`theme`** - Theme setting ('light', 'dark')
- **`state`** - Dynamic state (can change at runtime)
- **`context`** - Static context (properties inherent to the component type)

**Common use case:** A NavBar that adapts based on the first content section:

```jsx
function NavBar({ content, params, block }) {
  // Check what the first body section looks like
  const firstBodyInfo = block.page.getFirstBodyBlockInfo();

  // Or check the next block after this one
  const nextBlockInfo = block.getNextBlockInfo();

  // Use static context for component-type properties
  const allowTranslucentTop = nextBlockInfo?.context?.allowTranslucentTop || false;

  // Use theme for color adaptation
  const isDarkTheme = nextBlockInfo?.theme === 'dark';

  return (
    <nav className={`
      ${allowTranslucentTop ? 'absolute top-0 bg-transparent' : 'relative bg-white'}
      ${isDarkTheme ? 'text-white' : 'text-gray-900'}
    `}>
      {/* Navigation content */}
    </nav>
  );
}
```

**Hero component declaring its context in meta.js:**

```javascript
// Hero/meta.js
export default {
  title: 'Hero Banner',
  category: 'impact',

  // Static context: capabilities that are always true for Hero components
  context: {
    allowTranslucentTop: true  // Heroes always support translucent navbar overlay
  },

  params: { ... },
};
```

```jsx
// Hero/index.jsx
function Hero({ content, params, block }) {
  return (
    <section className="relative h-screen">
      {/* Full-screen hero with background image */}
    </section>
  );
}

export default Hero;
```

**Key distinction:**
- **`context`** — Static properties per component type. "All Hero components allow translucent top."
- **`state`** — Dynamic properties per block instance. "This specific accordion has item 2 open."

This pattern allows sophisticated visual coordination between sections without requiring content authors to manually configure each page.

### Styling Components

You can style components using any approach:

**Tailwind CSS:**

```jsx
function HeroSection({ content, params }) {
  return (
    <div className="relative h-screen flex items-center justify-center">
      <h1 className="text-5xl font-bold text-white">{content.main.title}</h1>
    </div>
  );
}
```

**CSS Modules:**

```jsx
import styles from "./HeroSection.module.css";

function HeroSection({ content, params }) {
  return (
    <div className={styles.hero}>
      <h1 className={styles.title}>{content.main.title}</h1>
    </div>
  );
}
```

### Composing with Regular Components

Most rendering logic should live in regular React components:

```jsx
// Section type — content creators select this via type: TeamSection
function TeamSection({ content, params, block }) {
  const { columns = 3, cardStyle = "standard" } = params;

  const CardComponent =
    cardStyle === "minimal" ? MinimalTeamCard : StandardTeamCard;

  return (
    <div className="team-section">
      <h2>{content.main.title}</h2>
      <div className={`team-grid cols-${columns}`}>
        {content.items.map((item, index) => (
          <CardComponent
            key={index}
            name={item.title}
            role={item.subtitle}
            bio={item.paragraphs[0]}
            image={item.images[0]}
          />
        ))}
      </div>
    </div>
  );
}

// Regular component — standard React, no meta.js
function StandardTeamCard({ name, role, bio, image }) {
  return (
    <div className="team-member-card">
      {image && <img src={image.src} alt={image.alt || name} />}
      <h3>{name}</h3>
      <p className="role">{role}</p>
      <p className="bio">{bio}</p>
    </div>
  );
}
```

## Component Configuration

Each section type has a `meta.js` file:

```js
// components/HeroSection/meta.js
export default {
  label: "Hero Section",
  description: "A full-width hero section with optional background image",
  category: "Layout",
};

export const parameters = [
  {
    name: "layout",
    label: "Layout",
    options: ["grid", "list", "carousel"],
    default: "grid",
  },
  {
    name: "columns",
    label: "Columns",
    type: "number",
    min: 1,
    max: 4,
    default: 3,
  },
];

export const presets = [
  {
    name: "standard",
    label: "Standard Grid",
    preview: "previews/standard.jpg",
    settings: { layout: "grid", columns: 3 },
  },
];
```

### Reserved Parameter Names

Do not use these names in your parameter definitions:

- **`component`** - Specifies which component to render
- **`preset`** - Applies predefined parameter configurations
- **`input`** - Defines data sources for dynamic content
- **`background`** - Controls background visuals
- **`theme`** - Controls color mode (light, medium, dark)
- **`title`** - Internal name of the section

## Advanced Patterns

### Parent-Child Relationships (Child Blocks)

Content creators can nest sections in `page.yml`, and parent components can render these child blocks:

```yaml
# pages/about/page.yml
sections:
  - tabs           # Parent section
    - overview     # Child of tabs
    - features     # Child of tabs
    - pricing      # Child of tabs
```

The parent component accesses children via `block.childBlocks` and renders them with `ChildBlocks`:

```jsx
import { useState } from 'react';

function Tabs({ content, params, block }) {
  const [activeTab, setActiveTab] = useState(0);

  // Get the renderer component for child blocks
  const ChildBlocks = block.getChildBlockRenderer();

  // No children? Render fallback
  if (!block.childBlocks?.length) {
    return <div>{content.main.title}</div>;
  }

  return (
    <div className="tabs">
      {/* Tab navigation - read child block titles */}
      <div className="tab-buttons">
        {block.childBlocks.map((childBlock, index) => (
          <button
            key={index}
            onClick={() => setActiveTab(index)}
            className={activeTab === index ? 'active' : ''}
          >
            {childBlock.main?.header?.title || `Tab ${index + 1}`}
          </button>
        ))}
      </div>

      {/* Render only the active child block */}
      <div className="tab-content">
        <ChildBlocks
          block={block}
          childBlocks={[block.childBlocks[activeTab]]}
        />
      </div>
    </div>
  );
}
```

**Other composition patterns:**

```jsx
// Two-column layout
function TwoColumn({ content, params, block }) {
  const ChildBlocks = block.getChildBlockRenderer();
  const [left, right] = block.childBlocks || [];

  return (
    <div className="grid grid-cols-2 gap-8">
      <div>{left && <ChildBlocks block={block} childBlocks={[left]} />}</div>
      <div>{right && <ChildBlocks block={block} childBlocks={[right]} />}</div>
    </div>
  );
}

// Accordion - render all children, control visibility
function Accordion({ content, params, block }) {
  const [state, setState] = block.useBlockState(useState);
  const ChildBlocks = block.getChildBlockRenderer();

  return (
    <div className="accordion">
      {block.childBlocks?.map((childBlock, index) => (
        <div key={index}>
          <button onClick={() => setState({ openIndex: index })}>
            {childBlock.main?.header?.title}
          </button>
          {state.openIndex === index && (
            <ChildBlocks block={block} childBlocks={[childBlock]} />
          )}
        </div>
      ))}
    </div>
  );
}
```

### Dynamic Data Integration

Fetched data (from `data:` or `fetch:` in frontmatter) is available via `content.data`:

```jsx
function ProductList({ content, params, block }) {
  const products = content.data?.products || [];
  const { layout = "grid", columns = 3 } = params;

  return (
    <div className="product-list">
      <h2>{content.title}</h2>
      {products.length === 0 ? (
        <p>No products available</p>
      ) : (
        <div className={`product-grid cols-${columns}`}>
          {products.map((product) => (
            <div key={product.slug} className="product-card">
              <h3>{product.title}</h3>
              <p>{product.description}</p>
            </div>
          ))}
        </div>
      )}
    </div>
  );
}
```

## Best Practices

1. **Start with the content structure** - Design components to handle common content patterns
2. **Use semantic parameters** - `theme: "dark"` not `backgroundColor: "#333"`
3. **Provide sensible defaults** - Components should work without configuration
4. **Handle edge cases gracefully** - Missing content, empty arrays, invalid values
5. **Document thoroughly** - Write clear parameter descriptions
6. **Keep section types thin** - Move rendering logic to regular components
7. **Always provide a Section component** - The default for content without explicit component selection

## Getting Started

```bash
# Create a development project with a Foundation and a site
npx @uniweb@latest create my-project
```

This creates:

- A Foundation at `src/marketing`
- A demo site at `sites/demo` for testing

Then add components:

```bash
cd my-project
npx uniweb component add HeroSection
```

Start the development server:

```bash
npx uniweb start
```

Visit `http://localhost:3000/sites/demo/` to see your test site.
