---
title: "Design Tokens: UI Aesthetics Codified"
description: "Bridging the gap between design and front-end development"
date: 2026-02-16
layout: layouts/post.njk
tags: blog
---

There's been excitement in the office recently about UI Design, Front-End Development, and bridging the gap between them. I wanted to put together this article to articulate some ideas in this area, mainly around Design Tokens.

Whenever asked "what are Design Tokens", I tend to tailor my answer to whoever is listening (usually developers!). But a purely technical viewpoint can miss the point: Design Tokens are the codified aesthetics of an interface, and using them is one of the most effective ways to bridge the gap between UI design and front-end development.

Consider a well-designed room. The colours, proportions, and materials are carefully chosen to work together, creating a sense of balance and harmony. Each decision is made in relation to the others, within a shared visual framework. The result is a coherent aesthetic experience.

Digital interfaces work the same way. The ones that feel _well designed_ are the product of deliberate, related decisions. The typography, spacing, colour, and motion all work together to create a sense of cohesion.

## Design decisions as data

<a href="https://spectrum.adobe.com/page/design-tokens/" target="_blank">Adobe Spectrum</a> (Adobe's Design System) refers to design tokens as "design decisions, translated into data." A token can be a colour, spacing unit, font size, drop shadow, or any visual unit in the design system. Tokens should have a name (`color-primary`), a value (`#1d4ed8`), and sometimes a type or description. <a href="https://zeroheight.com/blog/a-basic-introduction-to-design-tokens/" target="_blank">Zeroheight</a> describes them as variables with added metadata.

The term Design Token was coined by <a href="https://www.jina.me/" target="_blank">Jina Anne</a> while working on Salesforce's <a href="https://www.lightningdesignsystem.com/2e1ef8501/p/85bd85-lightning-design-system-2" target="_blank">Lightning Design System</a>, which refers to tokens as "the visual design atoms of the design system." <a href="https://css-tricks.com/what-are-design-tokens/" target="_blank">Robin Rendle's piece on CSS-Tricks</a> highlights why the idea spread: tokens did not just solve a problem for Salesforce, many other teams used them to solve the same issue. <a href="https://piccalil.li/blog/what-are-design-tokens/" target="_blank">Andy Bell also explains why tokens are useful in Front-End Design</a>: flat-file style guides delivered as PDFs just do not work. Developers eyeball the colours, approximate the spacing, and things drift out of sync. The gap between what the designer intended and what a developer creates widens. Design Tokens narrow this gap by eliminating the design handoff. 

When you choose the spacing scale, colour palette, type sizes, and *name* them, you articulate the visual logic of the entire product. You make the decisions that give the product its character and personality explicit. 

## Encoding relationships

A single colour hex like `#1d4ed8` doesn't mean anything on its own. What makes a design work is how that blue relates to everything else: how it contrasts with the background, how it sits alongside greys and accent colours, and whether it belongs in the same visual world as the typography, spacing, and other visual elements.

Defining a set of tokens, means defining a set of relationships. Take this spacing scale as an example:

```yaml
space-1: 0.25rem
space-2: 0.5rem
space-3: 0.75rem
space-4: 1rem
space-6: 1.5rem
space-8: 2rem
space-12: 3rem
```

This statement is about visual rhythm, and about how different elements should breathe in relation to each other. A card with `space-4` padding and `space-2` gaps between elements has a specific density and character. Changing the scale alters the character of the entire interface.

The same applies to colour. A well-constructed palette is a set of relationships between hues, contrast ratios and luminance curves. When <a href="https://github.com/adobe/spectrum-tokens/" target="_blank">Adobe Spectrum</a> maps each colour token to different values across five themes, they maintain those relationships under different conditions, ensuring the system *coheres* in each setting.

Typography works the same way. A type scale based on a consistent ratio produces sizes that feel related. The <a href="https://utopia.fyi" target="_blank">Utopia</a> fluid scales (I used to create this blog) are design tokens in this sense: proportional, considered values that produce rhythmic results because their relationships are designed.

## Constraints as a material

There is a theory in design and art more broadly that constraints enable creativity rather restrict it. A haiku has seventeen syllables, yet these limitations can produce better work than a blank page and infinite freedom.

Design tokens work on the same principle. A finite colour palette, a limited type scale, and a fixed set of spacing units make *coherence* possible. Without these constraints, every decision is made in isolation. Every component is a fresh canvas. The result is an interface that _feels off_, assembled from parts that were never introduced to each other.

Good designers already work this way intuitively. They establish constraints early and then design within that system. Whether they use the word or not, they are defining Design Tokens. The formalism makes the practice explicit, shareable, and enforceable.

The simplest way to start with Design Tokens on the Web is to use CSS Custom Properties:

```css
:root {
  --color-primary: #1d4ed8;
  --color-background-surface: #f3f4f6;

  --space-s: 0.75rem;
  --space-m: 1rem;
  --space-l: 1.5rem;

  --font-size-1: 1rem;
  --font-size-2: 1.25rem;
}
```

A collection of well-named tokens defines the visual language of the product.

## The act of naming

When you call something `color-background-surface` you make a commitment. You are saying: "this is the colour of a surface, and if we change what surfaces look like, this is what changes."

Naming forces you to articulate *intent*. Why is this element this colour, and what role does it play? What happens when the theme changes? These are design questions, and naming tokens answers them.

Consider the difference:

```css
/* A description — what it looks like */
background: var(--gray-100);

/* A decision — what it means */
background: var(--color-background-surface);
```

The first example gives facts about the UI: the background is a certain shade of grey. The second example describes the function: the background is a surface.

This matters because it communicates design intent beyond the original designer. <a href="https://atlassian.design/foundations/tokens/design-tokens" target="_blank">Atlassian</a> uses names like `color.icon.success` so anyone reading it knows what the colour is *for*. Anyone can decide when to use this token, even after the original designer is gone. Design intent encoded this way becomes design infrastructure.

Most design systems structure tokens into layers:

**Option tokens**: the raw values. `blue-500`, `gray-100`, `space-4`. These are the *materials* and represent the full set of colours, sizes, and values.

**Decision tokens**: the semantic layer. `color-primary`, `color-background-surface`, `space-element-gap`. These reference the options and add meaning. This is where design intent lives. Change `color-primary` from `blue-500` to `purple-600` and everything downstream updates. This also makes theming work. Dark mode becomes a remapping: `color-background-surface` points to `gray-100` in light mode and `gray-900` in dark mode.

**Component tokens**: scoped to specific elements. `button-background-primary`, `card-padding`. Not every system needs this layer, but it can be useful if it provides more context about intent.

For more information about structuring tokens this way, <a href="https://martinfowler.com/articles/design-token-based-ui-architecture.html" target="_blank">Martin Fowler has a good write-up on token-based UI architecture</a> that suggests starting with two layers and adding component tokens only when you need them.

## Coherence over consistency

There's a risk in talking about tokens only in terms of consistency. Consistency sounds mechanical, as if the goal is to stamp out variation and enforce sameness. That's not the point.

The deeper goal is *coherence*: the sense that every part of an interface belongs to the same world, is shaped by the same values, and responds to the same logic. That's much closer to what's meant by aesthetics or craft in digital design. A beautiful interface is one where the blues, greys, spacing, typography, shadows, and motion all feel like they were *considered together*. 

Think of it like a musical key. A piece in C major can use all sorts of chords and melodies, but they cohere because they share a tonal centre. Playing a random note will jar, because *unmotivated* tonal variation falls outside what the listener expects. Deviation is still possible, but it is _felt_. It is up to us where (or even, if) that deviation happens and which results we aim for. 

With five engineers building features independently, visual drift and deviation are a certainty. Tokens resist that drift by making the system's logic visible and deviation a conscious choice instead of an accident. 

## Preserving intent over time

Design entropy is real. One-off overrides creep in and exceptions accumulate.

Tokens provide structural resistance. When the system is encoded as named data with semantic meaning, introducing a new hardcoded value feels *wrong*. The token system creates a gravitational pull towards the defined values. That pull is not absolute, and people still override tokens. But it makes erosion a conscious act rather than an unconscious one.

A token system that defines every possible value for every property is as useless as no system at all. The art is choosing what to codify and what to leave open.

## Design as data

In a traditional workflow, a designer creates a mockup, produces a specification, and hands it to a developer to translate into code. The translation is lossy, and things get misread, approximated, and simplified.

If the token file is the source of truth, such as the JSON file that generates both Figma variables and CSS custom properties, then both disciplines work from a shared artefact.

The <a href="https://www.mozillafoundation.org/en/docs/design/websites/design-tokens/" target="_blank">Mozilla Foundation</a> keeps their tokens in a GitHub repo synced with Figma, and both designers and developers work from that file. When a designer changes a value in <a href="https://www.figma.com/community/plugin/888356646278934516/design-tokens" target="_blank">Tokens Studio</a>, it syncs to the repo and is consumed by the build. The developer uses the same named values in their code. The design *is* the code.

This shifts the relationship between disciplines from a sequential relay to a collaborative and simultaneous process. When a developer sees `color-background-surface-raised` and understands not just a hex value but a concept of visual hierarchy, the system teaches its own logic.

The important thing about tokens is that they're *data*, usually JSON. That file can be transformed into CSS custom properties, Swift constants, Sass variables, or any format the platform needs. <a href="https://www.contentful.com/blog/design-token-system/" target="_blank">Contentful</a> describes a hub-and-spoke model: JSON at the centre, platform-specific outputs radiating out. Amazon's <a href="https://github.com/style-dictionary/style-dictionary" target="_blank">Style Dictionary</a> is one of the tools available for this transformation. One source, many outputs.

## Where to start

If you are working on the Web, the easiest way to get started is with CSS Custom Properties. <a href="https://tailwindcss.com/" target="_blank">Tailwind CSS</a> now lets you use its CSS custom properties without utility classes. Tools like <a href="https://open-props.style/" target="_blank">Open Props</a> also provide a framework using Design Tokens on the Web.

The real point is the design work. Sit down and decide: what colours define this product? What is the spacing scale? What are the type sizes, and what is their relationship? Give those decisions names that express *why*, not just *what*. Write them down in one place.

Then you can decide the technical details and which platforms to support. 

## The landscape

Don't just take my word for it. Here is a section companies and teams using Design Tokens which I surveyed when putting this article together: 

**<a href="https://www.lightningdesignsystem.com/2e1ef8501/p/85bd85-lightning-design-system-2" target="_blank">Lightning Design System</a> (Salesforce)** — Where it started. Tokens as consumable packages for multiple platforms.

**<a href="https://m3.material.io/foundations/design-tokens/overview" target="_blank">Material Design 3</a> (Google)** — Dynamic colour: tokens derived algorithmically from a user's wallpaper or brand colour. The relationship system adapts to context.

**<a href="https://spectrum.adobe.com/page/design-tokens/" target="_blank">Spectrum</a> (Adobe)** — Three-part naming (`checkbox-control-size-small`). Five themes, all maintained through the same token structure.

**<a href="https://fluent2.microsoft.design/design-tokens" target="_blank">Fluent 2</a> (Microsoft)** — Global + alias layers across Windows, Web, iOS, Android, and macOS. Dark mode and high-contrast baked in.

**<a href="https://atlassian.design/foundations/tokens/design-tokens" target="_blank">Atlassian Design</a>** — `color.icon.success` naming. Covers elevation, opacity, space, and typography alongside colour. Uses tokens for density preferences and reduced motion too.

**<a href="https://designsystem.digital.gov/design-tokens/" target="_blank">USWDS</a> (US Government)** — Tokens as "curated palettes." Function-based access: `color('red')`, `u-padding-x(2)`.

**<a href="https://design.gitlab.com/product-foundations/design-tokens/" target="_blank">Pajamas</a> (GitLab)** — Open source. CSS custom properties in Vue. Design rationale is public.

**<a href="https://cloudscape.design/foundation/visual-foundation/design-tokens/" target="_blank">Cloudscape</a> (AWS)** — Enterprise scale. Names encode category, property, element, variant, and state.

**<a href="https://base.uber.com/6d2425e9f/p/33fa5e-design-tokens" target="_blank">Base</a> (Uber)** — Cross-brand theming for rider, driver, and delivery apps plus regional variants.

**<a href="https://www.mozillafoundation.org/en/docs/design/websites/design-tokens/" target="_blank">Mozilla Foundation</a>** — Three tiers in JSON, synced between Figma and GitHub.

## The tools

The **<a href="https://www.designtokens.org/" target="_blank">W3C Design Tokens Community Group</a>** is working on a standard JSON format so tokens can move between tools without bespoke translation layers.

**<a href="https://github.com/style-dictionary/style-dictionary" target="_blank">Style Dictionary</a>** (Amazon) is the workhorse for transformation — feed it token definitions, get platform-specific outputs.

**<a href="https://www.figma.com/community/plugin/888356646278934516/design-tokens" target="_blank">Design Tokens</a>** (formerly the Figma Tokens plugin) lets designers manage tokens inside Figma and sync to GitHub or GitLab. It turned token management from an engineering-only concern into something designers own too.

**<a href="https://penpot.app/blog/what-are-design-tokens-a-complete-guide/" target="_blank">Penpot</a>** supports W3C-standard tokens natively — import, export, and a REST API on the way.

## References and further reading

- <a href="https://zeroheight.com/blog/a-basic-introduction-to-design-tokens/" target="_blank">A Basic Introduction to Design Tokens</a> — Zeroheight
- <a href="https://piccalil.li/blog/what-are-design-tokens/" target="_blank">What Are Design Tokens?</a> — Andy Bell, Piccalilli
- <a href="https://css-tricks.com/what-are-design-tokens/" target="_blank">What Are Design Tokens?</a> — Robin Rendle, CSS-Tricks
- <a href="https://www.uxpin.com/studio/blog/what-are-design-tokens/" target="_blank">What Are Design Tokens?</a> — UXPin
- <a href="https://penpot.app/blog/what-are-design-tokens-a-complete-guide/" target="_blank">What Are Design Tokens: A Complete Guide</a> — Penpot
- <a href="https://uxdesign.cc/design-tokens-for-dummies-8acebf010d71" target="_blank">Design Tokens for Dummies</a> — UX Collective
- <a href="https://www.lightningdesignsystem.com/2e1ef8501/p/85bd85-lightning-design-system-2" target="_blank">Lightning Design System</a> — Salesforce
- <a href="https://m3.material.io/foundations/design-tokens/overview" target="_blank">Material Design 3</a> — Google
- <a href="https://spectrum.adobe.com/page/design-tokens/" target="_blank">Spectrum</a> — Adobe
- <a href="https://fluent2.microsoft.design/design-tokens" target="_blank">Fluent 2</a> — Microsoft
- <a href="https://atlassian.design/foundations/tokens/design-tokens" target="_blank">Atlassian Design</a> — Atlassian
- <a href="https://designsystem.digital.gov/design-tokens/" target="_blank">USWDS</a> — US Government
- <a href="https://design.gitlab.com/product-foundations/design-tokens/" target="_blank">Pajamas</a> — GitLab
- <a href="https://cloudscape.design/foundation/visual-foundation/design-tokens/" target="_blank">Cloudscape</a> — AWS
- <a href="https://base.uber.com/6d2425e9f/p/33fa5e-design-tokens" target="_blank">Base</a> — Uber
- <a href="https://www.mozillafoundation.org/en/docs/design/websites/design-tokens/" target="_blank">Mozilla Foundation</a>
- <a href="https://martinfowler.com/articles/design-token-based-ui-architecture.html" target="_blank">Design Token-Based UI Architecture</a> — Martin Fowler
- <a href="https://www.contentful.com/blog/design-token-system/" target="_blank">Design Token System</a> — Contentful
- <a href="https://www.designsystemscollective.com/design-tokens-demystified-a1d3dfa212c2" target="_blank">Design Tokens Demystified</a> — Design Systems Collective
- <a href="https://thedesignsystem.guide/design-tokens" target="_blank">The Design System Guide: Design Tokens</a>
- <a href="https://www.figma.com/community/plugin/888356646278934516/design-tokens" target="_blank">Tokens Studio for Figma</a>