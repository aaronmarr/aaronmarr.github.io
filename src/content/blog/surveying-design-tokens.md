---
title: "Design Tokens"
description: "An overview of the current state of Design Tokens"
date: 2026-02-26
layout: layouts/post.njk
tags: blog
---

In this post I want to survey the current landscape surrounding Design Tokens. Much of what's written here is information that has been put out by some great designers and developers currently working in the Front-End space, so I don't want to try to take any credit for ideas mentioned here. 

Instead, I'd like to get an overview of the current state of things, and to update my knowledge in this area. If it can be of use to anyone reading this, then that's a bonus.

AI was used during the writing of this post, but the research done and ideas presented were done by real humans including me and I will give credit where due!

## What are Design Tokens

The term Design Token was coined by <a href="https://www.jina.me/" target="_blank">Jina Anne</a> while working on Salesforce's <a href="https://www.lightningdesignsystem.com/2e1ef8501/p/85bd85-lightning-design-system-2" target="_blank">Lightning Design System</a>, which refers to tokens as "the visual design atoms of the design system." 

A token can be a colour, spacing unit, font size, drop shadow, or any visual unit in a design system. Tokens have a name (`color-primary`), a value (`#1d4ed8`), and sometimes a type or description. <a href="https://zeroheight.com/blog/a-basic-introduction-to-design-tokens/" target="_blank">Zeroheight</a> describes them as variables with added metadata.

<a href="https://css-tricks.com/what-are-design-tokens/" target="_blank">Robin Rendle's piece on CSS-Tricks</a> highlights why the idea spread: tokens did not just solve a problem for Salesforce, many other teams used them to solve the same issues in integrating design and front-end development.

As <a href="https://piccalil.li/blog/what-are-design-tokens/" target="_blank">Andy Bell explains</a>, flat-file style guides delivered as PDFs do not work. Developers eyeball the colours, approximate the spacing, and things drift out of sync. The gap between what the designer intended and what a developer builds widens over time. Design Tokens narrow this gap by replacing the handoff with a shared artefact.

## What tokens encode

A single colour hex like `#1d4ed8` does not mean much on its own. Tokens become useful when they encode relationships: how a colour contrasts with the background, how spacing values relate to each other, and how type sizes form a scale.

A spacing scale is a good example:

```
space-1:  0.25rem
space-2:  0.5rem
space-3:  0.75rem
space-4:  1rem
space-6:  1.5rem
space-8:  2rem
space-12: 3rem
```

These values define how elements are spaced in relation to each other. A card with `space-4` padding and `space-2` gaps has a specific density. Changing the scale changes the density of the entire interface.

Colour palettes work the same way. <a href="https://spectrum.adobe.com/" target="_blank">Adobe Spectrum</a> maps each colour token to different values across five themes, maintaining contrast ratios and luminance relationships in each setting. Typography follows the same pattern: a type scale based on a consistent ratio (such as the <a href="https://utopia.fyi" target="_blank">Utopia fluid scales</a> used on this blog) produces sizes that are proportionally related.

The simplest implementation on the Web is CSS Custom Properties:

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

## Naming and token layers

When a token is named `color-background-surface`, it carries a commitment: this is the colour of a surface, and if surfaces change, this is what changes. The name communicates intent, not just a value.

Consider the difference:

```css
/* Describes what it looks like */
background: var(--gray-100);

/* Describes what it means */
background: var(--color-background-surface);
```

The first tells you a shade of grey. The second tells you a function. Atlassian uses names like `color.icon.success` so that anyone reading the code knows what the colour is for and when to use it, regardless of who originally chose the value.

Most design systems in production today structure tokens into layers:

**Option tokens** are the raw values: `blue-500`, `gray-100`, `space-4`. These represent the full palette of available colours, sizes, and values.

**Decision tokens** are the semantic layer: `color-primary`, `color-background-surface`, `space-element-gap`. These reference option tokens and add meaning. Changing `color-primary` from `blue-500` to `purple-600` updates everything downstream. This is also how theming works: `color-background-surface` points to `gray-100` in light mode and `gray-900` in dark mode. The same semantic name, different underlying values.

**Component tokens** are scoped to specific elements: `button-background-primary`, `card-padding`. Not every system needs this layer, but it provides additional context about where a value is used.

<a href="https://martinfowler.com/articles/tokens-adaptive-ui.html" target="_blank">Martin Fowler's write-up on token-based UI architecture</a> recommends starting with two layers (options and decisions) and adding component tokens only when the need arises.

## Tokens as constraints

A finite colour palette, a limited type scale, and a fixed set of spacing units constrain what is available. Without these constraints, every decision is made in isolation and each component becomes a one-off. With multiple engineers building features independently, visual drift is inevitable. Tokens make the system's boundaries visible so that deviation becomes a deliberate choice rather than an accident.

This also helps with maintenance. When a system is encoded as named, semantic data, introducing a hardcoded value stands out. The token system creates pressure towards the defined values. That pressure is not absolute — overrides still happen — but it makes erosion visible.

A token system that tries to define every possible value is as unwieldy as no system at all. The practical advice from most design system teams is to codify the values that recur across components and leave the rest open.

## The design-to-code pipeline

In a traditional workflow, a designer produces a specification and hands it to a developer to translate into code. The translation is lossy: values get misread, approximated, and simplified.

Tokens change this by making a shared file the source of truth. The <a href="https://github.com/nickolasmv/mozilla-design-tokens" target="_blank">Mozilla Foundation</a> keeps their tokens in a GitHub repo synced with Figma via Tokens Studio. When a designer changes a value, it syncs to the repo and the build consumes it directly. The developer uses the same named values in code.

Because tokens are data (typically JSON), they can be transformed into whatever format each platform needs: CSS custom properties, Swift constants, Sass variables, Android XML resources. <a href="https://www.contentful.com/blog/design-token-guide/" target="_blank">Contentful describes this as a hub-and-spoke model</a>: JSON at the centre, platform-specific outputs radiating out. <a href="https://amzn.github.io/style-dictionary/" target="_blank">Amazon's Style Dictionary</a> is one of the more established tools for this transformation.

## Tools and starting points

For teams working on the Web, CSS Custom Properties are the most straightforward entry point. Beyond that, several tools and frameworks have emerged:

- <a href="https://sugarcube.sh/" target="_blank">**Sugarcube**</a> turns design tokens into CSS variables, utilities, and component styles.
- <a href="https://tailwindcss.com" target="_blank">**Tailwind CSS**</a> exposes its entire design system as CSS custom properties, which can be used independently of utility classes.
- <a href="https://open-props.style" target="_blank">**Open Props**</a> provides a pre-built set of design tokens as CSS custom properties, covering colour, spacing, typography, easing, and more.
- <a href="https://amzn.github.io/style-dictionary/" target="_blank">**Style Dictionary**</a> (Amazon) transforms a JSON token file into platform-specific outputs.
- <a href="https://tokens.studio" target="_blank">**Tokens Studio**</a> is a Figma plugin that manages tokens and syncs them to code repositories.
- <a href="https://utopia.fyi" target="_blank">**Utopia**</a> generates fluid type and space scales as token-ready CSS clamp values.

The starting point is the design work itself: deciding which colours, spacing values, and type sizes define the product, and giving those decisions names that express their purpose. The tooling follows from there.
