# Uniweb for Content Creators: A Practical Guide

This guide helps content creators build websites without writing code. You'll learn how to create, organize, and configure content effectively within the Uniweb framework.

**Related documentation:**

- [Understanding Uniweb](getting-started/understanding-uniweb.md) - Conceptual overview
- [Uniweb for Developers](developers-guide.md) - Technical guide for building components
- [Terminology Reference](reference/terminology.md) - Key terms and definitions

## Introduction

Uniweb separates content from code. This means:

- You can focus entirely on content without worrying about technical implementation
- You can update websites without developer involvement
- You can reuse the same content structure across different designs
- Your work won't be blocked by technical changes happening behind the scenes

As a content creator, you'll work with markdown files, configure components through simple YAML settings, and organize pages—all without touching code.

## Getting Started

### Your Workspace at a Glance

```
my-project/
└── site/
    ├── pages/                 # Where your content lives
    │   ├── home/              # Home page (→ /)
    │   │   ├── 1-hero.md      # First section
    │   │   ├── 2-features.md  # Second section
    │   │   └── page.yml       # Page metadata
    │   └── about/             # About page (→ /about)
    │       ├── 1-intro.md
    │       └── page.yml
    ├── locales/               # Translations
    │   └── es/
    │       └── pages/
    │           └── home/
    │               └── 1-hero.md
    └── public/                # Images and files
        └── images/
```

As a content creator, you'll primarily work with:

- **Markdown files (.md)** - Individual sections of content
- **Page files (page.yml)** - Page metadata like title and description
- **Images and assets** - Stored in the public folder

### Pages Are Folders

In Uniweb, **a page is a folder**. The folder name becomes the URL:

| Folder | URL |
|--------|-----|
| `pages/home/` | `/` (home page) |
| `pages/about/` | `/about` |
| `pages/products/` | `/products` |
| `pages/blog/my-post/` | `/blog/my-post` |

To create a new page, create a folder and add a markdown file:

```bash
# Create an about page
mkdir -p site/pages/about
```

Then add your content:

```markdown
# site/pages/about/1-intro.md

---
type: Hero
---

# About Us

We build great things.
```

That's it. Visit `/about` and your page is there.

### Basic Content Creation

Each markdown file is a **section** on the page. Here's a typical section:

```markdown
---
type: Hero
theme: dark
---

# Welcome to Our Platform

Discover how our solutions can transform your business.

![Hero Image](/images/hero.jpg)

[Get Started](#getting-started)
[Learn More](#about)
```

Let's break this down:

1. **Frontmatter** (between `---` lines) - Specifies the component and configuration options
2. **Content** - Everything after the frontmatter:
   - `#` creates headings (H1 becomes the title)
   - Regular text creates paragraphs
   - `![Text](/path)` adds images
   - `[Text](link)` creates links

You don't need to understand how components work internally—you just need to know which ones are available and what options they accept.

## Organizing Your Content

### Pages and Sections

- **Pages** are folders that map to URLs
- **Sections** are markdown files within those folders

```
pages/
├── home/                 # Home page
│   ├── 1-hero.md         # First section
│   ├── 2-features.md     # Second section
│   ├── 3-testimonials.md # Third section
│   └── page.yml          # Page metadata
└── about/                # About page
    ├── 1-team.md
    ├── 2-mission.md
    └── page.yml
```

Section files are numbered (`1-`, `2-`, etc.) to control their order on the page.

### Page Metadata (page.yml)

Each page folder can have a `page.yml` file for metadata:

```yaml
# pages/about/page.yml
title: About Us
description: Learn about our company and team
```

This metadata is used for SEO, browser tabs, and social sharing.

## Working with Components

Components are the building blocks that determine how your content is presented. Think of them as templates that give your content structure and style.

### Using Components

To use a component, specify it in the frontmatter:

```yaml
---
type: Features
columns: 3
---
```

### Configuring Components

Most components have options you can configure:

```yaml
---
type: TeamGrid
layout: cards
columns: 3
showRole: true
---
```

### Component Presets

Many components offer presets—pre-configured sets of options:

```yaml
---
type: Hero
preset: centered-dark
---
```

Presets are a great way to use professionally designed configurations without specifying all settings.

## Creating Effective Content

### Content Structure

Components expect content to be structured semantically:

```markdown
# Main Title

Introduction paragraph goes here.

## First Feature

Description of the first feature.

## Second Feature

Description of the second feature.
```

This creates:
- A main title (H1) with an introduction
- Multiple content groups (H2) that components display as cards, tabs, accordion items, etc.

The same content can look different depending on which component you choose:
- **Features** - Displays each H2 as a feature card
- **Tabs** - Turns each H2 into a separate tab
- **Accordion** - Makes each H2 a collapsible item

### Working with Images

Images use standard markdown syntax:

```markdown
![Team photo](/images/team.jpg)
```

The path references files in your `public` folder. `/images/team.jpg` refers to `site/public/images/team.jpg`.

### Creating Links

Standard links:

```markdown
[Learn more](/about)
```

External links:

```markdown
[Visit GitHub](https://github.com)
```

## Translating Your Content

Uniweb makes multilingual websites easy.

### How Translations Work

Translations are stored in the `locales` folder, mirroring your content structure:

```
site/
├── pages/
│   └── about/
│       └── 1-team.md      # English (default)
└── locales/
    └── es/
        └── pages/
            └── about/
                └── 1-team.md  # Spanish translation
```

### Creating Translations

When translating, you only need to include the content—not the frontmatter:

**Original (pages/about/1-team.md):**

```markdown
---
type: TeamGrid
columns: 3
---

# Our Team

Meet our dedicated professionals.

## Alice Smith

Chief Executive Officer
```

**Translation (locales/es/pages/about/1-team.md):**

```markdown
# Nuestro Equipo

Conozca a nuestros profesionales dedicados.

## Alice Smith

Directora General
```

Notice that the translation:
- Doesn't include the frontmatter (component settings stay the same)
- Only contains the translated content
- Keeps the same structure (headings, paragraphs)

## Best Practices

1. **Structure content logically** - Use headings to create clear hierarchy
2. **Be consistent** - Use similar patterns for similar content
3. **Think content, not presentation** - Focus on what you're saying, not exactly how it looks
4. **Use the right component** - Different components are designed for different content types
5. **Preview frequently** - Use the dev server to see how your content renders
6. **Use presets** - Start with presets for professionally designed layouts

## Quick Reference

### Adding a New Page

```bash
# 1. Create the folder
mkdir -p site/pages/services

# 2. Add a section
# Create site/pages/services/1-intro.md with your content

# 3. Visit /services
```

### Adding a Section to Existing Page

Create a new numbered markdown file in the page folder:

```bash
# Add a third section to the home page
# Create site/pages/home/3-cta.md
```

### Adding a Translation

Mirror the file in the locales folder:

```bash
# Spanish translation of home hero
# Create site/locales/es/pages/home/1-hero.md
```

## Next Steps

- [Understanding Uniweb](getting-started/understanding-uniweb.md) - Deeper dive into the architecture
- [Terminology Reference](reference/terminology.md) - Key terms and definitions

**For developers:**

- [Uniweb for Developers](developers-guide.md) - Learn how components are created
