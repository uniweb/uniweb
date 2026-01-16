# Introducing Uniweb Framework

We're excited to share something we've been working on for a while: the Uniweb Framework. It's a different way to think about building websites.

The framework's runtime handles the infrastructure that's the same in every project—routing, content processing, data fetching, localization. You focus on building components. No setup. No boilerplate. Just your design system and the components that make it unique.

And when you need content creators to work independently, the architecture already supports that. You build component systems—what we call Foundations—that others use to build sites. When you improve a component, every site using that Foundation gets the update on next page load if it contains non-breaking changes. No coordination, no redeployment.

It's component development as infrastructure. And it's open source.

## What We Made

The simplest way to explain it: you build components with full React capabilities while the runtime handles the infrastructure. Routing, content processing, data fetching—handled. You focus on your design system.

Sites are pure content and Foundations are pure code, hosted independently and interacting through a structured interface at runtime. Foundations can be hosted on any static CDN, or use Uniweb's hosting infrastructure.

You build component collections—what we call Foundations. A Foundation is like component library, but with a schema that describes the components that you choose to expose to content creators. When content creators need to work with your components, they can compose pages with their own content using content files or visual tools like the Uniweb App.

A Foundation is React components. Build them however you want. A Site is content—markdown, structured data, dynamic data, configuration, assets. When someone visits a page, the framework's runtime loads the Foundation dynamically, coordinates routing between pages, and renders each content block through the component it specifies.

Update a Foundation with a non-breaking improvement, and every site using that Foundation gets it on their next page load. Foundations use semantic versioning so sites can control their update strategy for stability.

## Why This Approach

We wanted something that works well for both developers and content creators—without compromise.

Developers should build real, dynamic sites. Full React capabilities. Any patterns. Complex interactions, API calls, whatever the project needs. The rich, evolving web—not toy sites or constrained templates.

Content creators should compose pages, add content, and publish—without waiting for developer time or touching code.

Most frameworks force a choice or create friction. We built the separation into the architecture itself. Foundations and Sites are independent. Developers maintain components. Content creators compose with them. The schema defines what's possible—you control how much flexibility to expose.

The framework's runtime handles the coordination—routing, component lifecycle, data fetching, content preprocessing—so Foundations stay focused on what makes them unique: rendering your components with your design system.

## What the Runtime Handles

Most web development involves writing the same infrastructure code repeatedly. Content processing. Data fetching. Routing. Localization. Form handling. State management. Every project needs it. Few projects need it to be different.

The framework's runtime takes care of this undifferentiated work. It's a lightweight coordination layer between your components and the content—so you can focus on what actually makes your Foundation valuable.

**Content preprocessing.** The runtime processes content before your components see it. You don't write parsing logic or handle edge cases. You receive structured content objects—headings organized into sections, paragraphs as arrays, images with metadata, lists with structure. Your components just render.

**SPA coordination.** The runtime manages routing with React Router, handles page transitions, coordinates component lifecycle. Sites are full single-page applications. Your Foundation components never interact with routing directly—the runtime handles it.

**Data fetching.** Need API data or database content? The runtime fetches it before rendering. Your component receives data ready to use. You don't write loading states or error handling for data sources the runtime manages.

**Framework components.** The `@uniweb/kit` package provides Section, Image, Link, Form, and more—all wrappers around runtime functionality. These handle accessibility, performance, edge cases. More importantly, they protect your Foundations from runtime implementation changes. The runtime evolves, but your components work with a stable interface.

This isn't about limiting what you can build. It's about not writing the same infrastructure code in every Foundation. Handle what's unique to your design system. Let the runtime handle what's common to every site.

## What You're Actually Working With

Here's what development looks like:

**Mostly standard React.** Only components exposed to content creators need special interfaces and schemas. Everything else is regular React—internal components, utilities, whatever you need. Import any packages. Use any patterns. The Framework defines the boundary where components meet content. Inside that boundary, it's your code.

**Zero-config tooling.** Webpack, Babel, TypeScript support, Tailwind CSS, PostCSS—pre-configured. Hot module replacement, source maps, production optimization. Just build.

**Structured and dynamic data.** Content can be markdown, JSON blocks within markdown, or structured data. Need explicit structure like team members or products? Use JSON blocks with schema validation. Need dynamic content? The framework's runtime handles data fetching from APIs and databases. Your component receives data ready to render.

**Semantic parameters.** The components you expose to content creators use semantic options—`theme: "dark"` not `backgroundColor: "#333"`. Content creators understand their options. You can change implementation without breaking content.

## The Workflow

You build a Foundation for whatever you're building—marketing site, documentation, product catalog. Your components. Your design system. Write schemas defining each component's interface (at minimum just the name, add parameters for configurability). Publish it.

Someone creates a site and selects your Foundation. They compose pages with your components—either working with content files and Git, or through the Uniweb App, a visual editor that reads your schemas and presents your components as native building blocks. The file-based approach is free and works with any Git workflow. The App is a commercial service.

They compose pages: Hero at the top, feature grid below, FAQ at the bottom. Configure components through frontmatter or visual controls. Write content. Add pages. Publish.

You improve the Hero component—better responsive behavior, new feature, bug fix. You publish.

Their site gets it on the next page load. No redeployment from them. No coordination with you. The update propagated.

They add three pages and reorganize navigation. You never see those changes. Don't need to.

That's the independence.

## What Works Well

**Visual editor integration.** When a site uses the Uniweb App, your schemas tell the editor how to present your components—what controls to show, what presets to offer, what's valid. Your design system becomes the interface.

**Natural scaling.** Start with one site and one Foundation—better organized React development. Need to serve multiple sites later? The architecture already supports it. Don't need it? You've got cleaner separation regardless.

**True content independence.** Content creators work in their space, you work in yours. Neither blocks the other. Content changes don't require developer review. Component improvements don't require content coordination.

**Full dynamic capabilities.** Not limited to static content. API calls, databases, complex state, forms, authentication—build what your site needs.

**No lock-in.** Use any packages, styling, or tools. The Framework is unopinionated about implementation. Standard React development with better content organization.

## Who This Helps

| **Developers**                                                                          | **Content Creators**                                            |
| --------------------------------------------------------------------------------------- | --------------------------------------------------------------- |
| Preprocessed content—no content parsing, receive structured objects                     | Compose pages with smart components, not generic blocks         |
| Component improvements propagate automatically—no coordination                          | Updates flow automatically—no maintenance requests              |
| Undifferentiated work handled by runtime—data fetching, localization, structure parsing | Visual editor (with App)—your components become their framework |
| Professional utilities included—forms, images, navigation, React hooks                  | Publish instantly—no builds, no deployment pipeline             |
| Foundations get visual editing (with App)—no extra code or configuration needed         | Autonomy—add pages, reorganize content, no developer dependency |

## Try It

```bash
npx @uniweb@latest create my-project
cd my-project
pnpm install
pnpm dev
```

You'll get a Foundation with example components and a demo site. See how it works. Modify components. Check the docs at [docs.uniweb.io](https://docs.uniweb.io).

## Publishing Your Foundation

Once you've built a Foundation, you can deploy it to any CDN for sites to use. To make your Foundation available in the Uniweb App, publish it to the App registry:

```bash
npx uniweb login
npx uniweb module publish
```

This registers your Foundation under your account.

## For Client Work

The framework makes client handoffs straightforward. Build a Foundation for your clients' needs—your design system as a framework they build on:

1. Go to [uniweb.app](https://uniweb.app)
2. Click **Create → Website**
3. Select your Foundation (published in the previous step)
4. Build initial content with the visual editor
5. Click **Publish**
6. Transfer ownership to your client

The Foundation license automatically transfers with the site. Your client owns their site and can continue building content independently. You maintain the Foundation—improving components, adding features, fixing bugs. Your improvements propagate to their site automatically.

Terms and compensation are between you and your client. You can offer Foundation maintenance as an ongoing service, or structure it however works for your business. The architecture just makes the technical workflow clean.

## What's Next

This is the beginning of community engagement. The Framework is open source (GPL-3.0)—Foundations and sites you build are yours to license however you want. Build and deploy everything yourself, or use the Uniweb App and hosting as optional services.

We're building out the broader ecosystem too: better tooling, more examples, and the Uniweb App—a visual editor that integrates with Foundations to give content creators a complete UI for building and managing sites.

Right now, we're just happy to share what we made and see what people build with it.

If you've wanted to build component frameworks that others can build sites with, or clean separation between component code and content, or updates that propagate instantly across multiple sites—try it.

We'd love to hear what you think. [Docs](https://docs.uniweb.io) at the framework's site, [code](https://github.com/uniweb/cli) on GitHub. We're early, so you're early. Build something. Tell us what's confusing or doesn't work. Help us figure out what this becomes.
