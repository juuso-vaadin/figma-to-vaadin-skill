---
name: vaadin-to-figma
description: Generate or update Figma designs from Vaadin Flow views using live app capture combined with Vaadin Design System components. Use when the user wants to create Figma designs from Vaadin views, mentions "generate design", "create in Figma", "push to Figma", or "Vaadin to Figma". Always combines Java code analysis + live Figma capture + Vaadin DS component composition. Delegates to figma-generate-design and figma-use skills — this skill provides Vaadin-specific context for those workflows.
---

# Vaadin to Figma

This skill provides Vaadin-specific context when invoking the `figma-generate-design` and `figma-use` skills. Always load and follow those skills for the actual Figma workflow — this file adds the Vaadin layer on top.

**Always use both approaches together:**
1. **Java code analysis** — to identify correct components, variants, and layout structure
2. **Live Figma capture** — as the visual source and reference for the Figma design

---

## Workflow

At the start of every session, create this checklist and check them off as you go:

- [ ] **Step 1 — Analyze Java view**: Read the view file, extract layout, components, variants, text, route
- [ ] **Step 2 — Inject capture script**: Add script to `/src/main/frontend/index.html` before app starts
- [ ] **Step 3 — Confirm app is running**: Ask user; prompt restart if script was just added
- [ ] **Step 4 — Run Figma capture**: Capture the live app via the Figma capture script
- [ ] **Step 5 — Discover Figma components**: Search design system for all needed components
- [ ] **Step 6 — Build Figma design**: Construct layout section by section using DS components
- [ ] **Step 7 — Update text content**: Replace all placeholder text with real values from the Java view
- [ ] **Step 8 — Visual verification**: Screenshot Figma design, compare to capture, identify and fix issues

---

## Step 1 — Analyze the Vaadin View (Java Code)

Read the Vaadin view Java file and extract:

- **Layout structure**: `VerticalLayout`, `HorizontalLayout`, `FlexLayout`, `FormLayout`, `SplitLayout`
- **Components and variants**: e.g. `Button` with `addThemeVariants(ButtonVariant.PRIMARY)`, `TextField` with `addThemeVariants(TextFieldVariant.SMALL)`
- **Text content**: labels, placeholder text, button labels, headings
- **Route**: find `@Route("route-name")` annotation — determines the URL
- **Sections**: header/toolbar, navigation (tabs, drawer), content areas, footer

**Route → URL mapping:**
- `@Route("settings")` → `http://localhost:8080/settings`
- `@Route("")` → `http://localhost:8080/`

---

## Step 2 — Inject the Capture Script

**Before asking if the app is running**, add the Figma capture script to the Vaadin frontend entry point. This must be in place before the app starts — it cannot be injected into a running app.

**File**: `/src/main/frontend/index.html`

Add inside `<head>`, before `</head>`:
```html
<script src="https://mcp.figma.com/mcp/html-to-design/capture.js" async></script>
```

> Do not use Playwright or any other screenshot tool — the Figma capture script is the only capture mechanism to use.

---

## Step 3 — Confirm App is Running

After confirming the script is injected, ask the user:

> "Is the Vaadin application running (with the capture script in place)? Please choose:
> - **Option 1**: Yes, running on the default port (http://localhost:8080)
> - **Option 2**: Yes, running at a different URL — please share it"

If the app was already running before the script was injected, ask the user to restart it first.

---

## Step 4 — Run the Figma Capture

Use `figma-generate-design` capture workflow with the app URL and route derived from the `@Route` annotation. The Figma capture is the **visual source** — it drives layout proportions, spacing, and visual appearance. The DS component rebuild is layered on top.

---

## Step 5 — Discover Components in the User's Figma Library

Use `search_design_system` to find Vaadin DS components. **Never hardcode component keys** — always discover dynamically.

**Search strategy:**
- Search by Vaadin component name (see mapping table below)
- If the user has customized components, prefer their custom library matches over the base Vaadin DS library
- Map Java variant code to Figma component names or properties as described below

### Theme Variants

Vaadin components use `addThemeVariants()` to apply visual styles. In recent Vaadin versions, many variants are theme-agnostic (e.g. `TextFieldVariant.SMALL`). Older or theme-specific variants carry a `LUMO_` or `AURA_` prefix (e.g. `RadioGroupVariant.LUMO_VERTICAL`). Both should be mapped to the correct Figma component or property.

**General rule**: strip the `LUMO_` / `AURA_` prefix to get the semantic variant name, then find the matching Figma component or property. Theme-specific variants (where Lumo and Aura differ visually) may map to different Figma components — use whichever matches the user's active theme.

### Component and Variant Mapping

The list below is illustrative, not exhaustive. Use it as a pattern for discovering other components not listed here.

**Button**

| Java | Figma |
|------|-------|
| `Button` (no variant) | Search `"button"` → default Button component |
| `ButtonVariant.PRIMARY` | Search `"button primary"` — separate component set in Vaadin DS |
| `ButtonVariant.TERTIARY` | Search `"button tertiary"` — separate component set |
| `ButtonVariant.ICON` | Search `"button icon"` — separate component set |

> Note: In Vaadin DS Figma, button theme variants are typically **separate component sets**, not properties within one component set.

**Text Field**

| Java | Figma |
|------|-------|
| `TextField` (default) | Search `"text field"` |
| `addThemeVariants(TextFieldVariant.SMALL)` | Search `"text field"` → set size property to `"Small"` |
| `addThemeVariants(TextFieldVariant.ALIGN_RIGHT)` | Search `"text field"` → set alignment property |

**Tabs**

| Java | Figma |
|------|-------|
| `Tabs` (default / no orientation set) | Search `"horizontal tabs"` |
| `tabs.setOrientation(Orientation.VERTICAL)` | Search `"vertical tabs"` |

**Radio Button Group**

| Java | Figma |
|------|-------|
| `RadioButtonGroup` (default) | Search `"radio button group"` → horizontal variant |
| `addThemeVariants(RadioGroupVariant.AURA_HORIZONTAL)` or `RadioGroupVariant.LUMO_HORIZONTAL` | Search `"radio button group (horizontal)"` |
| `addThemeVariants(RadioGroupVariant.LUMO_VERTICAL)` or `RadioGroupVariant.AURA_VERTICAL` | Search `"radio button group (vertical)"` |

**Badge**

| Java | Figma |
|------|-------|
| `Span` + `addThemeNames("badge")` (default) | Search `"badge"` → default style |
| `addThemeVariants(BadgeVariant.ERROR)` | Search `"badge"` → set style property to `"Error"` |
| `addThemeVariants(BadgeVariant.SUCCESS)` | Search `"badge"` → set style property to `"Success"` |
| `addThemeVariants(BadgeVariant.WARNING)` | Search `"badge"` → set style property to `"Warning"` |
| `BadgeVariant.PILL` | Search `"badge"` → set shape property to `"Pill"` |

**Other common components** — search by name and map variants as properties:

| Vaadin Component | Search Term |
|-----------------|-------------|
| `TextArea` | `"text area"` |
| `ComboBox` | `"combo box"` |
| `Select` | `"select"` |
| `DatePicker` | `"date picker"` |
| `TimePicker` | `"time picker"` |
| `Checkbox` | `"checkbox"` |
| `Grid` | `"grid"` |
| `Dialog` | `"dialog"` |
| `Notification` | `"notification"` |
| `Avatar` | `"avatar"` |

**Layouts** (`VerticalLayout`, `HorizontalLayout`, `FlexLayout`, `FormLayout`) → Figma auto-layout frames, no library component needed.

---

## Step 6 — Build the Figma Design

Follow the `figma-generate-design` and `figma-use` skill workflows. Vaadin-specific settings:

- **Page size**: 1440×1024px (default desktop)
- **Spacing**: Vaadin Lumo uses an 8px base unit — prefer multiples: `8, 16, 24, 32, 48`
- **Colors**: Use existing Figma variables for colors whenever possible — do not hardcode hex values
- **Text styles**: Use existing Figma text styles whenever possible — do not hardcode font sizes or weights
- **Build one section per `use_figma` call** — header, nav, content areas separately
- **Always set `primaryAxisSizingMode` and `counterAxisSizingMode` explicitly** when creating auto-layout frames — the default is `FIXED` and `resize()` alone does not change sizing behavior, so a frame resized to 10px height stays locked at 10px even with auto-layout enabled
- **Set `layoutSizingHorizontal = "FILL"` only after `appendChild`**, never before
- Use the Figma capture as visual reference for proportions, spacing, and content placement

---

## Step 7 — Update Text Content

After placing component instances, update all text to match values from the Java view — labels, button text, placeholder text, headings. Never leave default placeholder text (`"Label"`, `"Value"`, `"Button"`).

---

## Step 8 — Visual Verification and Iteration

This step is **mandatory** — always run it.

1. Take a screenshot of the Figma design
2. Compare side-by-side with the Figma capture result from Step 4
3. Identify issues across these categories:
   - **Text**: wrong labels, placeholder text still showing, wrong font/size/weight
   - **Layout**: wrong spacing, misaligned elements, wrong section proportions
   - **Components**: wrong variant used, component swapped for a primitive, missing components
   - **Visuals**: wrong colors, missing icons, borders/shadows incorrect
   - **Missing content**: entire sections absent, hidden content not shown
   - **Hidden layers**: Vaadin DS components often have layers that are hidden by default — if something appears missing or blank, check for hidden layers and enable them
4. If issues are found, do **one automatic iteration** — fix all identified issues and take a new screenshot
5. If the design is still off after that iteration, **ask the user** whether to do another round or stop:
   > "The design still has some differences from the capture. Would you like me to do another iteration, or is this good enough for now?"
6. Only continue iterating if the user confirms

---

## Handling Custom/Extended Components

If the user has customized their Vaadin DS library:
- Their custom components take priority — always prefer a match from their library
- Look for components with names that extend the base (e.g. "App Button", "My Text Field")
- If a custom component wraps a Vaadin base component, use the custom one

---

## Related Skills

Always load these for the actual execution:
- **`figma-generate-design`** — primary skill for screen building and capture workflow
- **`figma-use`** — Plugin API rules; load when writing Figma scripts
- **`figma-to-vaadin`** — reverse workflow (Figma → Vaadin code)