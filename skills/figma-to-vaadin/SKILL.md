---
name: figma-to-vaadin
description: >
  Translate Figma designs into Vaadin Flow (Java) UI code. Use this skill whenever the user wants
  to implement a Figma frame, screen, or component as Vaadin Java code — even if they just say
  "implement this design", "generate Vaadin code from Figma", "convert this frame to Java", or
  paste a Figma URL. Does NOT apply to React, HTML, web components, or other frontend frameworks
  — only Vaadin Flow (Java). Does NOT apply to design-only tasks such as editing Figma files or
  generating Figma components. Does NOT configure themes or visual design tokens — that is a
  separate skill.
compatibility: Requires a Figma MCP server and the Vaadin MCP server
---

# Figma to Vaadin: the Vaadin-specific half

## The process — follow it in order

**This skill is four documents.** This file is the spine; the three references carry the detail
that decides whether the output matches the design. Work the steps in order, and **open the
document a step names before doing that step's work.** The one-line summaries below are pointers
to those documents, not replacements for them.

**1. Learn the project.** Nothing about it should be assumed — read it out of the project each
time:

- **Vaadin version** — from the build file (`pom.xml` / `build.gradle`). Pass it to **every**
  Vaadin MCP call, so you get the API surface this project actually compiles against.
- **The app's theme** — Lumo, Aura, or custom. Decides which variant constants apply, the
  default component styling, and which CSS custom properties are available.
- **An existing view** — shows the base class views extend, the shared header/footer wrappers,
  how CSS classes are named and where rules live.
- **What the app shell already provides** — a design screenshot shows the whole application, but
  navigation and chrome usually belong to the shell. Build only the content region;
  re-implementing the navigation renders it twice.
- **The icon set and the data source** — projects often add their own icon set, and existing
  records beat a parallel data model invented to fit the design.

Code style, architecture and conventions come from the project's own guidelines — a `CLAUDE.md`
or equivalent. This skill does not restate them: how to structure a view, when to split out
reusable components, how to name things and how to shape sample data are decisions the project
already makes. Read them there and follow them.

**2. Load `figma-design-to-code`, then decompose the frame up front.** Full-screen frames
truncate, and `get_design_context` can return an incomplete answer without saying so. Get the
region tree first, then request context **per region**. Don't discover truncation late and fall
back to metadata alone — that carries geometry with no styling, so the implementation degrades to
boxes in roughly the right places.

**3. Measure the design — read `references/fidelity.md` first.** It sets out which properties to
take from the returned code and which from the screenshot, and which details are easiest to lose
on the way to Java. Record the view's own background and foreground first, then per region:
nesting, padding, gap, border, size, component type.

**4. Build the layout — read `references/layout.md` first.** It has the layout API surface in
full, so the design's flexbox maps onto Vaadin's layout components rather than hand-written CSS,
together with what genuinely belongs in CSS and the sizing defaults worth setting explicitly.

**5. Choose and style components — read `references/components.md` first.** It covers how the
components behave out of the box — Grid column sizing, card semantics, theme variants, icons — so
what you write complements a component's own styling rather than duplicating or overriding it.

**6. Close the loop — read the last section of `references/fidelity.md` again.** Check the
emitted code back against your measurements in all three directions it describes, then compile.
Compilation is the cheapest objective check available.

**7. Hand off verification.** Writing the code is this skill's job; confirming it against the
design is not. If the project has a visual-verification skill, invoke it with the Figma reference
and the route, and present its findings rather than acting on them unprompted. Agree with the
user first whether you should also apply a round of fixes.

## What this skill does and does not own

The Figma side already has a skill: `figma-design-to-code`, which the Figma MCP requires you to
load before calling `get_design_context`. It owns fetching design context, treating the returned
code as a reference rather than final, the hint priority order (Code Connect → component docs →
annotations → design tokens → raw values), reusing what the project has, and asset fidelity.
**Follow it, and don't restate it here.**

This skill adds only what that skill cannot know: how Vaadin behaves, and how to find out what
*this* project does. It assumes nothing about the design either — a Figma file may be built from
a Vaadin library, may reference Lumo or another theme, or may have no relationship to Vaadin at
all. All are in scope; the difference is only how much you can take directly and how much you
must translate.

## The source is authoritative; the docs are for usage

Never answer a Vaadin question from memory — APIs, variants, custom properties and feature flags
evolve between versions.

**For what exists, the project's own Vaadin jars are ground truth.** They are the version the
project compiles against. Settle any question of the form *does this method/constant/overload
exist* against them: `javap` on the classpath for a signature or an enum's constants, or a
three-line `javac` probe for anything involving generics or overload resolution.

**Never repair a compile error by guessing a nearby method name.** A probe settles in seconds
what reasoning gets wrong confidently — for instance whether a setter takes a boolean or a CSS
string, or whether a sizing method lives on the component or on a grid column.

**For what things mean, use the Vaadin MCP.** Source tells you a variant constant exists; the
docs tell you what it does to the rendering, which is usually the question you have.

- `get_component_styling` — **before writing any CSS for a component.** What the component
  already does is the input to half the rules in `references/components.md`.
- `get_theme_css_properties` — before using a custom property. **Never invent a property name**:
  a `var(--made-up, fallback)` silently becomes a permanent hard-coded value that never tracks
  the theme.
- `search_vaadin_docs` → `get_full_document` — to find a component when you don't know which
  fits, for intended usage, and for worked examples. Search results are previews; read the
  document before relying on one.
- `get_component_java_api` — a faster read than `javap` when you want the shape of a component's
  API rather than a yes/no on one signature.

**Feature-flag status changes between versions too.** Some components sit behind a flag in one
version and ship enabled in the next — check rather than recalling, and if a component needs a
flag the project hasn't set, say so instead of silently choosing something else.

## The rules that decide the outcome

**A floor, not a summary — this list does not replace the three documents.** These are the rules
that most often decide whether the result is right, kept here so they survive even if everything
else is forgotten:

- **The layout API comes before CSS.** `HorizontalLayout` and `VerticalLayout` *are* flexbox.
  Translating Figma's flexbox into `Div`s with `display: flex` feels faithful and is the most
  common structural defect.
- **Absence of a declaration is not neutral — it inherits the component default.** Restate no
  default; override every one the design contradicts.
- **Borders are the most-missed property in the design.** A 1px line is invisible in a screenshot,
  so nothing downstream will catch a missing one.
- **Implement what the design contains and nothing else.** An empty or underspecified region is a
  question for the user, not a blank to fill. Writing the guess down does not license shipping it.
- **The design names a specific icon.** If it isn't available in the project, ask — never
  substitute a different one, and never silently omit it.
- **Don't trust layer names.** Figma names drift from content. Take component type from
  `data-name` and annotations, and the view's identity from its visible heading text. If a
  component still doesn't map to one Vaadin component, ask rather than guess.
