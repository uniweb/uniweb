# Core Architecture

This document is for developers maintaining the `@uniweb/core` package and the runtime. It explains the internal architecture, design patterns, and how the core classes work together.

**Audience:** Framework maintainers, not Foundation developers.

## Overview

The core package (`@uniweb/core`) provides pure JavaScript classes that represent the runtime data model. These classes have no React dependencies—React integration happens through a "bridge pattern" that accepts hooks as parameters.

```
@uniweb/core (pure JS)     @uniweb/runtime (React)
─────────────────────      ────────────────────────
Uniweb                     createUniweb()
Website                    Layout, ChildBlocks
Page                       Router integration
Block                      Component rendering
```

## Object Graph

When a site loads, the runtime creates this object hierarchy:

```
globalThis.uniweb (Uniweb singleton)
└── activeWebsite: Website
    ├── pages: Page[]
    │   └── Page
    │       ├── website (back-reference)
    │       ├── route, title, description
    │       └── pageBlocks: { header, body, footer, left, right }
    │           └── Block[]
    │               ├── page (back-reference)
    │               ├── website (back-reference)
    │               ├── component (name)
    │               ├── state (persistent)
    │               └── childBlocks: Block[]
    │
    ├── activePage: Page
    ├── locales: [{code, label, isDefault}]
    └── config, theme
```

### Construction Order

The hierarchy is constructed top-down, with parent references passed to children:

```javascript
// Website constructor
constructor(websiteData) {
  // 1. Website initializes its own properties
  this.config = config

  // 2. Website creates Pages, passing itself
  this.pages = pages.map((pageData, index) =>
    new Page(pageData, index, this, ...)  // 'this' is the Website
  )
}

// Page constructor
constructor(pageData, id, website, ...) {
  // 1. Page stores the website reference immediately
  this.website = website

  // 2. Page creates Blocks, passing references
  this.pageBlocks = this.buildPageBlocks(...)
}

// Inside buildPageBlocks
initBlockReferences(block) {
  block.page = this
  block.website = this.website

  // Recursively for child blocks
  for (const child of block.childBlocks) {
    this.initBlockReferences(child)
  }
}
```

This ensures every Block has valid `page` and `website` references from the moment it's created.

## The Bridge Pattern

The core classes are pure JavaScript with no React imports. To integrate with React, they use a "bridge pattern" that accepts React hooks as function parameters.

### Why This Pattern?

1. **No framework coupling** — Core classes don't import React
2. **Version independence** — Works with any React version
3. **Testable** — Can test core logic without React
4. **SSR compatible** — No React globals needed at construction time

### How It Works

```javascript
// In Block class (pure JS)
class Block {
  useBlockState(useState, initState) {
    // Accept useState as a parameter, not an import
    if (initState !== undefined && this.startState === null) {
      this.startState = initState
      this.state = initState
    } else {
      initState = this.startState
    }

    const [state, setState] = useState(initState)

    // Store the reset hook for later use
    this.resetStateHook = () => setState(initState)

    // Return tuple that updates both React state and block.state
    return [state, (newState) => setState((this.state = newState))]
  }
}
```

```jsx
// In a React component (Foundation)
function MyComponent({ block }) {
  // Pass useState to the block
  const [state, setState] = block.useBlockState(useState)

  // Now state is synchronized between React and block.state
}
```

### Key Insight

The handler (Block) takes control of the React integration:
- It decides when to call `useState`
- It manages the subscription lifecycle
- It can trigger re-renders by calling the stored `setState`
- It keeps `block.state` synchronized with React state

From React's perspective, it's just calling a hook. From the Block's perspective, it's managing state imperatively. Both sides get what they need.

## Cross-Block Communication

Blocks can read information from neighboring blocks, enabling adaptive behavior. The canonical example is a NavBar adapting to the first content section.

### Implementation

```javascript
// Block class
class Block {
  getIndex() {
    if (!this.page) return -1
    return this.page.getBlockIndex(this)
  }

  getBlockInfo() {
    return {
      type: this.type,           // Component type ('Hero', 'Features', etc.)
      theme: this.themeName,      // Theme setting ('light', 'dark')
      state: this.state,          // Dynamic state (can change at runtime)
      context: this.context       // Static context (per component type)
    }
  }

  getNextBlockInfo() {
    const index = this.getIndex()
    if (index < 0 || !this.page) return null
    return this.page.getBlockInfo(index + 1)
  }
}

// Page class
class Page {
  getBlockIndex(block) {
    const allBlocks = this.getPageBlocks()
    return allBlocks.indexOf(block)
  }

  getBlockInfo(index) {
    const allBlocks = this.getPageBlocks()
    return allBlocks[index]?.getBlockInfo() || null
  }

  getFirstBodyBlockInfo() {
    return this.pageBlocks.body?.[0]?.getBlockInfo() || null
  }
}
```

### Usage in Components

```jsx
// NavBar checks if first section supports overlay
function NavBar({ block }) {
  const firstBodyInfo = block.page.getFirstBodyBlockInfo()
  // Use context for static component-type properties
  const allowTranslucentTop = firstBodyInfo?.context?.allowTranslucentTop || false

  return <nav className={allowTranslucentTop ? 'transparent' : 'solid'}>...</nav>
}

// Hero declares its static context
Hero.block = {
  context: { allowTranslucentTop: true }  // Static: all Heroes support translucent navbar
}
```

### Block Info Structure

The `getBlockInfo()` method returns:

```javascript
{
  type: string,        // Component type ('Hero', 'Features', etc.)
  theme: string,       // 'light', 'dark', etc.
  state: object|null,  // Dynamic state (can change at runtime)
  context: object|null // Static context (per component type)
}
```

**Key distinction:**
- **`context`** — Static properties defined per component type. Never changes.
- **`state`** — Dynamic properties per block instance. Can change via `useBlockState`.

## State and Context Management

### Block State Lifecycle

1. **Declaration** — Component declares `Component.block = { state: {...}, context: {...} }`
2. **Initialization** — When Block loads the Component, it copies defaults
3. **Connection** — Component calls `useBlockState(useState)`, connecting React
4. **Updates** — `setState` updates both React and `block.state`
5. **Reset** — On page navigation, `initState()` resets state to `startState`

```javascript
// Block class
initComponent() {
  this.Component = globalThis.uniweb?.getComponent(this.type)

  // Get component-level block configuration
  const blockConfig = this.Component.block || {}

  // Initialize state (dynamic)
  this.startState = blockConfig.state ? { ...blockConfig.state } : null
  this.initState()

  // Initialize context (static)
  this.context = blockConfig.context ? { ...blockConfig.context } : null
}

initState() {
  this.state = this.startState
  if (this.resetStateHook) this.resetStateHook()  // Reset React state too
}
```

### Why Persistent State?

Block state persists across:
- Re-renders (standard React behavior)
- Page navigations (block instances are reused)

This enables features like:
- Remembering accordion open/closed state
- Preserving form input when navigating away and back
- Cross-page state for wizards/multi-step flows

## Child Blocks

Child blocks are created from the `subsections` property in content data:

```javascript
// Block constructor
this.childBlocks = blockData.subsections
  ? blockData.subsections.map((block, i) => new Block(block, `${id}_${i}`))
  : []
```

Child blocks get their `page` and `website` references from `initBlockReferences()`, which recurses into children.

### Rendering Child Blocks

The runtime provides a `ChildBlocks` component that's registered globally:

```javascript
// Runtime registers this
globalThis.uniweb.childBlockRenderer = ChildBlocks
```

Components access it via:

```javascript
const ChildBlocks = block.getChildBlockRenderer()
```

## Page Layout Areas

Pages have five block groups:

```javascript
pageBlocks: {
  header: Block[] | null,  // From @header special page
  body: Block[],           // Main content sections
  footer: Block[] | null,  // From @footer special page
  left: Block[] | null,    // Left panel
  right: Block[] | null    // Right panel
}
```

The `getPageBlocks()` method returns a flat array respecting layout preferences:

```javascript
getPageBlocks() {
  const blocks = []
  if (this.hasHeader() && this.pageBlocks.header) {
    blocks.push(...this.pageBlocks.header)
  }
  blocks.push(...this.pageBlocks.body)
  if (this.hasFooter() && this.pageBlocks.footer) {
    blocks.push(...this.pageBlocks.footer)
  }
  return blocks
}
```

## Adding New Features

### Adding a Block Method

1. Add the method to `packages/core/src/block.js`
2. If it needs Page/Website access, use `this.page` or `this.website`
3. If it bridges to React, accept hooks as parameters
4. Update documentation in `docs/developers-guide.md`

### Adding a Page Method

1. Add the method to `packages/core/src/page.js`
2. If it iterates blocks, consider which layout areas to include
3. Update documentation

### Adding Website Functionality

1. Add to `packages/core/src/website.js`
2. Consider if it needs to propagate to pages/blocks
3. Update documentation

## Testing

Core classes can be tested without React:

```javascript
import { Website, Page, Block } from '@uniweb/core'

test('cross-block communication', () => {
  const websiteData = {
    pages: [{
      route: '/',
      sections: [
        { component: 'NavBar' },
        { component: 'Hero', params: { theme: 'dark' } }
      ]
    }]
  }

  const website = new Website(websiteData)
  const page = website.pages[0]
  const [navbar, hero] = page.pageBlocks.body

  // NavBar can get Hero's info
  const nextInfo = navbar.getNextBlockInfo()
  expect(nextInfo.component).toBe('Hero')
  expect(nextInfo.theme).toBe('dark')
})
```

## Performance Considerations

1. **Block lookups** — `getBlockIndex()` uses `indexOf()` which is O(n). For pages with many blocks, consider caching.

2. **State synchronization** — `useBlockState` creates a closure each render. This is unavoidable with the bridge pattern.

3. **Child block recursion** — `initBlockReferences` recurses through all children. Deep nesting could be slow, but typical sites have shallow hierarchies.

## Related Documentation

- [Architecture Decisions](./architecture.md) — Build system decisions
- [Developers Guide](../developers-guide.md) — Foundation developer documentation
- [Terminology](./terminology.md) — Key terms and definitions
