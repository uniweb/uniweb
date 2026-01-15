# Choosing Uniweb Framework for Your Next Project

You're starting a new project. Next.js? Vite? Something else? There's another option worth considering: Uniweb Framework.

But not necessarily for the reason you might think.

## What You're Actually Choosing

Most framework decisions come down to: "What features do I need?" SSR, routing, build optimization, whatever.

Uniweb Framework is different. The choice is really: "Do I want to write infrastructure code, or focus on components?"

The framework's runtime handles routing with React Router, preprocesses content, fetches dynamic data before rendering, manages localization. Not "provides tools for these"—actually does them. You build components that render. The runtime handles everything else.

This isn't about whether you need content/code separation. That's a benefit, but the productivity gain exists even if you're building solo. You're not setting up routing. Not writing markdown parsers. Not handling data fetching boilerplate. You build components and schemas. That's it.

## What the Runtime Actually Does

**Content preprocessing.** Your components receive structured content, parameters, and full website context. Your write the code that renders the content.

**Routing coordination.** React Router is set up and managed. Use a natural folder hierarchy to define page routes. Navigation works. You don't configure anything.

**Data fetching.** The runtime fetches data before rendering. Your component receives data ready to use. No loading states for data sources the runtime manages.

**Component communication.** Event bus for intra-component coordination. Components can communicate without coupling. React hooks let you react to state changes external to your component.

**Site context access.** Components can query page hierarchy, active page, available pages, available languages, current language, search functionality—anything about the site structure. You don't build navigation systems from scratch. You ask the runtime for context and use it.

**Analytics tracking.** Track user interactions and events from your components. The infrastructure is there. You just use it.

**Localization infrastructure.** Built in. Content in multiple languages? Handled.

**Content structure management.** Pages are folders with page sections as individual markdown files. Markdown with frontmatter and schema-based data blocks. The runtime understands the structure and gives you page hierarchies, navigation data, section organization. You define components and compose with them.

This is real work you're not writing. Every project needs it. Few projects need it to be different.

## When It Makes Sense

**You're building content-heavy sites.** Documentation, marketing, portfolios, learning—sites where content structure matters and changes frequently. The markdown-first approach with structured content objects is designed for this.

**You separate content from code anyway.** If you naturally put content in json/markdown files or external data sources, the framework's architecture matches your workflow. It just gives you infrastructure for free.

**You want to build components, not setup.** Zero-config tooling. Webpack, Babel, TypeScript support, Tailwind CSS, hot module replacement—pre-configured. You focus on component code.

**You're building for clients who'll manage content.** The Foundation model (components you build, content they manage) makes handoffs clean. They work independently. You maintain components with automatic update propagation.

**You're building similar sites repeatedly.** Build a Foundation once, use it for multiple client sites. Each site has independent content, shared components. Updates to components propagate to all sites automatically, based on their own versioning policy.

## When It Doesn't Make Sense

**You need custom routing logic.** The runtime manages routing with React Router. If your project needs routing to work fundamentally differently, that's fighting the framework's architecture. Use something where routing is yours to control.

**Infrastructure IS the product.** Building a SaaS dashboard? Custom application with complex state management as the core feature? The framework is designed for content-driven sites, not applications where infrastructure logic is the primary complexity. Use tools built for that.

**Content and code are tightly coupled by nature.** If your project doesn't naturally separate content from code—like an interactive tool or data visualization where the "content" is really application state—the framework's separation model doesn't help.

## The Real Question

It's not "what features does Uniweb Framework have?" It's "does the architecture match how I work?"

If you naturally separate content from code, if you build content-heavy sites, if you want infrastructure handled so you can focus on components—try it.

If you need custom infrastructure, if you're building applications rather than sites, if you need total control over how everything works—use tools built for that.

## What You Get

Full React capabilities. Import any packages. Use any patterns. Only components exposed to content creators need schemas. Everything else is standard React.

The framework provides professional utilities—forms, images, navigation, content rendering—that wrap runtime functionality. You don't start from scratch.

When you build a Foundation, you can use it for multiple sites. Improve a component, the update propagates to every site using that Foundation automatically. Sites use semantic versioning to control updates—minor and patch flow automatically, major changes require opt-in.

Deploy Foundations to any CDN. Content creators can work in markdown/Git (free) or use the Uniweb App (commercial service) for visual editing.

The framework is open source (GPL-3.0). Foundations you build are yours to license however you want.

## Try It

```bash
npx @uniweb@latest create my-project
cd my-project
pnpm install
pnpm dev
```

You'll get a Foundation with example components and a demo site. See what the runtime handles. See what you actually build. Check if it matches your workflow.

Documentation at [docs.framework.uniweb.app](https://docs.framework.uniweb.app). Code on [GitHub](https://github.com/uniwebcms/framework).

## The Decision

You probably already know if this fits your project. Content-heavy, naturally separated, want infrastructure handled? Worth trying.

Custom application, need low-level infrastructure control, content tightly coupled? Use Next.js or Vite.

The framework isn't better or worse. It's optimized for different work. Does that work match yours?
