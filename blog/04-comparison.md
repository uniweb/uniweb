# Component Web Platform vs. Headless vs. Webflow vs. WordPress

## Building the same "Pricing page" four ways

In the [previous post], we introduced a new category: **Component Web Platform**—the content–component bridge for domain-adaptable publishing.

But categories are abstract. Problems are concrete.

To see why this category is necessary, let's build the same thing four ways:

- **The Website Builder way** (Webflow, Wix)
- **The Traditional CMS way** (WordPress)
- **The Headless CMS way** (Contentful, Sanity)
- **The Component Web Platform way** (Uniweb)

We'll use one concrete example: a typical **Pricing Page**.

---

## The spec: it's never "just a page"

A pricing page is a living organism. It typically includes:

1. **Hero section**: Headline + positioning text
2. **Pricing toggle**: Monthly vs. Annual logic
3. **Plan grid**: Cards with complex comparison data
4. **Feature groups**: Categorized benefits, often collapsible
5. **Social proof**: Logo strips or testimonial carousels
6. **FAQ**: Accordions with expandable answers
7. **Final CTA**: Conversion-focused closer

And here's the key reality: pricing pages evolve constantly. Marketing runs tests, adds plans, updates copy, rotates proof, and tweaks the comparison table.

So the real requirement isn't "build it once." It's:

> **Build it so the team can evolve it without breaking the system.**

---

## 1. Website Builder: Webflow-style

### How you build it

You assemble the page visually. Draw containers, set padding, choose typography, configure responsive breakpoints, arrange the grid by hand. You are designing pixels.

If your team is disciplined, you'll create reusable symbols and follow a design system. But the tool doesn't enforce this—it just makes it possible.

### What's great

- Extremely fast for skilled creators
- WYSIWYG feeling—what you see is what ships
- Perfect for small teams and short-lived pages
- Beautiful results when done well

### The failure mode: Design Drift

Builders optimize for freedom, which is their fatal flaw at scale.

As different team members edit the page over months, entropy sets in:

- Spacing becomes inconsistent (40px here, 45px there)
- Typography gets overridden manually ("just this once")
- Button styles diverge slightly across pages
- Mobile layouts break in edge cases nobody tested
- "Temporary" overrides become permanent fixtures

Even with a design system, builders make it easy to bypass the system. That's why brand coherence erodes: the tool optimizes for freedom over integrity.

**The verdict:** Builders excel at _creation_. They struggle at _evolution_. You gain speed but lose coherence over time.

---

## 2. Traditional CMS: WordPress-style

### How you build it

The reality varies widely:

- Theme + page builder plugin (Elementor, Divi)
- Gutenberg blocks with custom styling
- Custom templates built by a developer
- Some combination of all three

Content lives in a mix of post bodies, custom fields, shortcodes, and theme options. The pricing table might be a plugin. The FAQ might be another plugin. The hero is probably part of the theme.

### What's great

- Enormous ecosystem—a plugin exists for almost anything
- Widely understood by agencies and freelancers
- Can power almost any use case with enough effort
- Mature hosting and maintenance infrastructure

### The failure mode: Sediment and Coupling

Two patterns emerge over time:

**Sediment**: The theme + plugin stack becomes a layer cake of assumptions. Each update risks compatibility issues. Each new feature adds another dependency. The system accumulates technical debt invisibly.

**Coupling**: Content and presentation intertwine. Your pricing data lives inside the pricing table plugin's format. Your FAQ answers are stored in the FAQ plugin's schema. Want to redesign the site? You're migrating content out of half a dozen plugin formats.

**The verdict:** WordPress offers infinite flexibility, but maintaining coherence becomes a "discipline tax" you pay every week. The system works until it doesn't, and then fixing it is expensive.

---

## 3. Headless CMS: Contentful-style

### How you build it

You stop thinking about pages and start modeling data. You create content types:

- Pricing Page
- Plan (with fields for name, price, features, CTA)
- Feature Group
- FAQ Item
- Testimonial

Then developers build a separate frontend application—React, Next.js, whatever—that fetches this data via API and renders it. The CMS is purely a content repository; the website is a separate codebase entirely.

### What's great

- Clean content architecture (no coupling to presentation)
- Powerful reuse and relationships (content serves multiple channels)
- Developer-friendly (API-first, modern tooling)
- Excellent for multi-channel ecosystems (web, mobile, kiosks)

### The failure mode: The Preview Gap

The architecture is clean, but the workflow has friction.

Creators work in a UI that is:

- Abstract (fields, dropdowns, reference pickers)
- Disconnected from the real page experience
- Dependent on preview quality and build times

To see their changes, creators hit "Preview," wait for a build, check the result, discover a problem, return to the form, adjust, and repeat. The editorial experience is a loop:

> Edit → Preview → Adjust → Preview → Adjust → Publish

The issue isn't "headless can't preview." Modern headless tools have invested heavily in preview. The issue is that the publishing surface is inherently not the website's component language. It's a content system with a separate rendering world.

**The verdict:** Headless solves the _developer's_ problem (clean architecture) but creates a _creator's_ problem (blind editing). It's half a solution—the content half, without the publishing half.

---

## 4. Component Web Platform: Uniweb-style

A Component Web Platform changes the unit of work.

You aren't managing pixels (Builder), wrestling plugins (WordPress), or filling out abstract forms (Headless). **You are composing components.**

### How you build the Pricing Page

**Step 1: The Foundation**

Developers define the component vocabulary once. For a pricing page, this might include:

- Pricing Hero (headline, subhead, optional background)
- Plan Switcher (toggle logic, plan cards, comparison mode)
- Feature Groups (collapsible categories with benefit lists)
- Proof Strip (logo grid or testimonial slider)
- FAQ (accordion with schema.org markup)
- Final CTA (headline, button, urgency options)

Each component includes:

- Content slots (what creators fill in)
- Configuration options (what creators can adjust)
- Constraints (what combinations are valid)
- Responsive behavior (how it adapts—not negotiable)
- Visual design (typography, spacing, color—not negotiable)

**Step 2: The Composition**

Creators open the page editor. They don't see a blank canvas or an abstract form. They see a page composed of components they can arrange and configure.

To build the pricing page, they:

1. Add a Pricing Hero section → fill in headline and subhead
2. Add a Plan Switcher section → enter plan data, choose default billing period
3. Add Feature Groups → organize benefits by category
4. Add a Proof Strip → select which logos to show
5. Add FAQ → write questions and answers
6. Add Final CTA → set the conversion message

The page is live-rendered as they work. No preview loop. No build wait. What they see is what ships.

### Why it works: The Capability Boundary

The platform enforces a hard line between creator territory and system territory:

**Creators control:**

- Content (every piece of text, every image)
- Composition (which sections, in what order)
- Exposed options (toggle defaults, emphasis levels, visible/hidden states)

**The system protects:**

- Visual design (spacing, typography, color relationships)
- Component behavior (toggle logic, accordion interactions, responsive rules)
- Structural integrity (valid combinations, required fields)

This is the **Capability Boundary**. Creators have genuine freedom inside it. The system stays coherent outside it.

**The verdict:** This is the only model that solves the evolution problem directly. It manages **how content becomes pages through components**—so the site stays coherent while the team moves fast.

---

## The same page, summarized

| Approach               | Unit of Work            | Strength             | Long-run Failure Mode         |
| ---------------------- | ----------------------- | -------------------- | ----------------------------- |
| Website Builder        | Pixels & layout         | Fast visual creation | Design Drift                  |
| Traditional CMS        | Themes + plugins        | Broad flexibility    | Coupling & fragility          |
| Headless CMS           | Content models & fields | Clean architecture   | Publishing surface disconnect |
| Component Web Platform | Components              | Coherent evolution   | Requires a real Foundation    |

That last column matters: a Component Web Platform expects the system to be designed. It's not "anything goes." If you don't invest in the Foundation, you don't get the benefits.

---

## How to choose

**If you're a solo creator building a small site:** A builder is often perfect. The drift problem doesn't matter if only one person touches the site and it has a limited lifespan.

**If you need extreme backend extensibility and accept maintenance cost:** WordPress can be right. The ecosystem is vast, and many agencies know how to manage the complexity.

**If you're building a multi-channel content architecture with strong dev resources:** Headless can be right. The API-first model shines when content serves many frontends.

**If your biggest problem is publishing at scale without drift:** A Component Web Platform is the missing model. It's for teams who need non-developers to publish safely, frequently, and without eroding the brand.

---

## The category difference in one sentence

A website builder manages layout.
A headless CMS manages structured content.
A traditional CMS manages templates and plugins.

A **Component Web Platform** manages how content becomes pages through components.
