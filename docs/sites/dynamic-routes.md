# Dynamic Routes

This document describes dynamic routes in Uniweb — pages generated from data sources like blog articles, product catalogs, or documentation entries.

## Overview

Dynamic routes allow a single page template to generate multiple pages based on a data source:

```
pages/blog/
├── page.yml              # Page config with data source
├── 1-header.md           # Listing page content
└── [slug]/               # Dynamic route template
    └── 1-article.md      # Article template
```

This would generate:
```
dist/blog/index.html              # Listing page
dist/blog/my-first-post/index.html
dist/blog/another-article/index.html
dist/blog/hello-world/index.html
```

## Proposed Configuration

### Data Source in page.yml

```yaml
# pages/blog/page.yml
title: Blog
description: Our latest articles

dataSource:
  type: json                    # json | yaml | api
  path: ./data/articles.json    # Local file path
  # OR
  url: https://api.example.com/articles  # Remote API

  # Field mappings
  slugField: slug               # Field to use as URL slug
  titleField: title             # Field for page title
  descriptionField: excerpt     # Field for meta description
```

### Data Source Formats

**JSON file:**
```json
[
  {
    "slug": "my-first-post",
    "title": "My First Post",
    "excerpt": "Introduction to our blog",
    "date": "2024-01-15",
    "content": "Full article content here..."
  },
  {
    "slug": "another-article",
    "title": "Another Article",
    "excerpt": "More interesting content",
    "date": "2024-01-20",
    "content": "..."
  }
]
```

**YAML file:**
```yaml
- slug: my-first-post
  title: My First Post
  excerpt: Introduction to our blog
  content: |
    Full article content here...

- slug: another-article
  title: Another Article
  excerpt: More interesting content
```

**API endpoint:**
```json
{
  "articles": [
    { "slug": "...", "title": "...", ... }
  ]
}
```

## Dynamic Route Template

The `[slug]/` directory contains the template for each generated page:

```markdown
# pages/blog/[slug]/1-article.md

---
type: Article
---

{{title}}

{{content}}
```

Template variables are populated from the data source entry.

## Build Process

### Standard Build (SPA)

1. Dynamic routes work client-side
2. JavaScript fetches data and renders appropriate content
3. Single `index.html` handles all routes

### SSG Build (--prerender)

1. Read `page.yml` and detect `dataSource`
2. Fetch/load data source
3. For each entry:
   - Create page context with entry data
   - Render template with entry values
   - Output static HTML file
4. Generate listing page with all entries

```bash
uniweb build --prerender
```

Output:
```
dist/blog/index.html           # Lists all articles
dist/blog/my-first-post/index.html
dist/blog/another-article/index.html
dist/blog/hello-world/index.html
```

## Listing Page

The parent page (e.g., `/blog`) can access all entries for listing:

```markdown
# pages/blog/1-header.md

---
type: ArticleList
---

# Blog

Browse our latest articles.
```

The `ArticleList` component receives all entries from the data source as props.

## Use Cases

| Use Case | Data Source | Example |
|----------|-------------|---------|
| Blog | JSON/YAML file | `./data/posts.json` |
| Documentation | Markdown files | `./docs/**/*.md` |
| Product catalog | API | `https://api.store.com/products` |
| Team members | YAML | `./data/team.yml` |
| Portfolio | JSON | `./data/projects.json` |

## Implementation Phases

### Phase 1: Local Data Sources
- JSON file support
- YAML file support
- Basic template variable substitution
- SSG pre-rendering of all entries

### Phase 2: Enhanced Templates
- Nested data access (`{{author.name}}`)
- Conditional rendering
- Date formatting helpers

### Phase 3: Remote Data Sources
- API endpoint support
- Authentication headers
- Caching during build

## Limitations

- **Build-time only** - Data is fetched at build time, not request time
- **Rebuild for updates** - Content changes require a new build
- **No pagination** - All entries are pre-rendered (suitable for hundreds, not millions)

For real-time dynamic content, draft previews, or massive datasets, use [uniweb.app](https://uniweb.app) which provides server-side rendering and live content management.

## Example: Blog Implementation

```
my-project/
├── site/
│   ├── data/
│   │   └── articles.json       # Blog content
│   └── pages/
│       └── blog/
│           ├── page.yml        # Data source config
│           ├── 1-hero.md       # Listing page hero
│           └── [slug]/
│               ├── page.yml    # Dynamic page config
│               └── 1-article.md # Article template
```

**articles.json:**
```json
[
  {
    "slug": "getting-started",
    "title": "Getting Started with Uniweb",
    "date": "2024-01-15",
    "author": "Team",
    "content": "Welcome to Uniweb..."
  }
]
```

**blog/page.yml:**
```yaml
title: Blog
dataSource:
  type: json
  path: ../data/articles.json
  slugField: slug
```

**blog/[slug]/1-article.md:**
```markdown
---
type: BlogPost
---

# {{title}}

By {{author}} on {{date}}

{{content}}
```

**Build output:**
```
dist/blog/index.html
dist/blog/getting-started/index.html
```

## Related

- [Static Site Generation (SSG)](./ssr-ssg.md) - Pre-rendering basics
- [CLI Reference](../reference/cli.md) - Build commands
