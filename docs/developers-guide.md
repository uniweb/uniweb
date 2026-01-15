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

### Two Types of Components in a Foundation

The Framework supports two distinct types of components:

1. **Exposed components**:

   - Components that content creators can select in markdown files
   - Follow the special `{ content, params, block }` interface
   - Require schema files (`component.config.js`)
   - Act as the bridge between content and code
   - Can be composed through section hierarchy (parent-child relationships)

2. **Internal components**:
   - Regular React components used within your implementations
   - Follow standard React patterns and can use any props structure
   - Don't require schema files or special interfaces
   - Not directly available to content creators
   - Handle most of the actual rendering and UI logic

This distinction is crucial: Uniweb fully supports standard React development patterns. In fact, most of your actual UI rendering should happen in internal components, with exposed components primarily handling the content/code bridge.

## Building Components

### Component Interface

Every Foundation exposed component follows this interface:

```jsx
function HeroSection({ content, params, block }) {
  // Extract content from the structured content object
  const { title, paragraphs, images } = content.main;

  // Extract parameters with defaults
  const { layout = "standard", theme = "light" } = params;

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

// Initialize block defaults for persistent state
HeroSection.blockDefaults = {
  state: {
    supportsOverlay: true,
  },
};

export default HeroSection;
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
component: HeroSection
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

The `block` object provides runtime context and methods:

```jsx
function InteractiveComponent({ content, params, block }) {
  const [count, setCount] = useState(block.getState("count") || 0);

  const increment = () => {
    const newCount = count + 1;
    setCount(newCount);
    block.setState("count", newCount);
  };

  const parent = block.getParent();
  const children = block.getChildBlocks();

  return (
    <div>
      <h2>{content.main.title}</h2>
      <p>Count: {count}</p>
      <button onClick={increment}>Increment</button>
    </div>
  );
}
```

Common block methods:

- `block.id` - Unique identifier for this block instance
- `block.type` - Component name
- `block.getState(key)` - Get persistent state value
- `block.setState(key, value)` - Set persistent state value
- `block.getParent()` - Get parent block (if child)
- `block.getChildBlocks()` - Get array of child blocks (if parent)
- `block.input` - Dynamic data fetched from APIs

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

### Using Internal Components

Most rendering logic should live in internal (standard React) components:

```jsx
// Exposed component that content creators can select
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

// Internal component - standard React
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

Each exposed component has a `component.config.js` file:

```js
// components/HeroSection/component.config.js
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

### Parent-Child Relationships

```jsx
function Tabs({ content, params, block }) {
  const [activeTab, setActiveTab] = useState(0);
  const childBlocks = block.getChildBlocks();

  return (
    <div className="tabs">
      <div className="tab-buttons">
        {childBlocks.map((child, index) => (
          <button
            key={index}
            onClick={() => setActiveTab(index)}
            className={activeTab === index ? "active" : ""}
          >
            {child.content.main.title}
          </button>
        ))}
      </div>
      <div className="tab-content">{childBlocks[activeTab]?.render()}</div>
    </div>
  );
}
```

### Dynamic Data Integration

```jsx
function ProductList({ content, params, block }) {
  const products = block.input || [];
  const { layout = "grid", columns = 3 } = params;

  return (
    <div className="product-list">
      <h2>{content.main.title}</h2>
      {products.length === 0 ? (
        <p>No products available</p>
      ) : (
        <div className={`product-grid cols-${columns}`}>
          {products.map((product, index) => (
            <div key={index} className="product-card">
              <h3>{product.getTitle()}</h3>
              <p>{product.getDescription()}</p>
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
6. **Keep exposed components focused** - Move rendering logic to internal components
7. **Always provide a Section component** - The default for content without explicit component selection

## Getting Started

```bash
# Create a development project with a Foundation
npx @uniwebcms/framework@latest create my-project --site demo --module marketing
```

This creates:

- A Foundation module at `src/marketing`
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
