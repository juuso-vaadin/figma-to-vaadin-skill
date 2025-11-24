# Figma to Lumo CSS Variables Mapping Template

## Overview
This template provides a systematic approach for mapping Figma design tokens to Lumo CSS variables using the `get_variable_defs` tool. The goal is to customize Lumo Style Properties in `frontend/themes/[app]/styles.css` to match the design system defined in Figma.

## Required Workflow for CSS Variable Mapping
Create TODOs based on these steps.
- Step 1: Extract Figma Design Tokens
- Step 2: Categorize Design Tokens
- Step 3: Extract Component Styles From Figma with `get_design_context`
- Step 4. Review Component Styling Documentation
- Step 5: Generate CSS Variable Declarations (Only Non-Default Values)

This approach ensures:
- ✅ Only custom values are set, preserving Lumo defaults
- ✅ Component-specific variables are included when needed  
- ✅ Dark theme only included if Figma design shows dark variants
- ✅ Systematic conversion from Figma tokens to CSS variables
- ✅ No validation/testing steps - pure value transfer process

### Step 1: Extract Figma Design Tokens
**Always start with `get_variable_defs`**

This returns design tokens like:
```javascript
{
  "lumo-primary-color": "#006af5",
  "lumo-header-text-color": "#192434",
  "Normal body text": "Font(family: \"Inter\", style: Regular, size: 16, weight: 400, lineHeight: 1.61)",
  "lumo-border-radius-m": "8",
  "vaadin-input-field-height": "36",
  "vaadin-input-field-border-color": "#1c304a85"
}
```

### Step 2: Categorize Design Tokens
Organize Figma tokens by Lumo categories:

#### **Color Tokens** → Map to Lumo Color Properties
- `lumo-primary-color` → `--lumo-primary-color`
- `lumo-header-text-color` → `--lumo-header-text-color`
- `lumo-body-text-color` → `--lumo-body-text-color`
- `lumo-base` → `--lumo-base-color`
- `lumo-contrast-XXpct` → `--lumo-contrast-XXpct`
- `lumo-error-color`, `lumo-warning-color`, `lumo-success-color` → Error/warning/success colors

#### **Typography Tokens** → Map to Lumo Typography Properties
- `lumo-font-family` → `--lumo-font-family`
- `lumo-font-size-xxxl`, `lumo-font-size-xxl`, `lumo-font-size-xl`, etc. → `--lumo-font-size-{xxxl|xxl|xl|l|m|s|xs|xxs}`
- Font() definitions from `Heading 1`, `Heading 2`, `Normal body text` → Extract size and line-height values

#### **Sizing Tokens** → Map to Lumo Size & Space Properties
- Component sizes → `--lumo-size-{xs|s|m|l|xl}`
- Spacing values → `--lumo-space-{xs|s|m|l|xl}`
- Icon sizes → `--lumo-icon-size-{s|m|l}`
- `Component sizes/Fields/Field - Label gap` → Custom spacing values

#### **Shape Tokens** → Map to Lumo Shape Properties
- `lumo-border-radius-s`, `lumo-border-radius-m`, `lumo-border-radius-l` → `--lumo-border-radius-{s|m|l}`

#### **Elevation Tokens** → Map to Lumo Elevation Properties
- Shadow definitions → `--lumo-box-shadow-{xs|s|m|l|xl}`

#### **Component-Specific Tokens** → Direct Variable Mapping
Many component variables are directly available in Figma:
- `vaadin-input-field-height` → `--vaadin-input-field-height`
- `vaadin-input-field-background` → `--vaadin-input-field-background`
- `vaadin-input-field-border-color` → `--vaadin-input-field-border-color`
- `vaadin-input-field-border-width` → `--vaadin-input-field-border-width`
- Use Vaadin MCP for additional component variables if needed

### Step 3: Extract Component Styles From Figma with `get_design_context`
- Contains the most detailed component information
- Check `className` attributes for any Tailwind classnames
- Identify theme/styling variable names and values

### Step 4: Review Component Styling Documentation
Use Vaadin MCP to identify component specific CSS variables:
```javascript
// Search for additional component variables if needed
search_vaadin_docs("button styling CSS variables")
get_full_document("components/button/styling-flow.md")
```

Use Vaadin MCP to identify Lumo theme defaults
```javascript
// Search for Lumo style properties
search_vaadin_docs("lumo style properties")
get_full_document("styling/lumo/lumo-style-properties.md")
```


### Step 5: Generate CSS Variable Declarations (Only Non-Default Values)
**IMPORTANT**: Only set CSS variables that differ from Lumo defaults. Do not override variables with their default values.

Create CSS declarations in `styles.css`:

```css
/* Imported from Figma */

html {
  /* Primary colors */
  --lumo-primary-color: hsla(218, 100%, 48%, 1);
  --lumo-primary-text-color: hsla(218, 100%, 44%, 1);

  /* Text colors */
  --lumo-header-text-color: hsla(218, 31%, 20%, 1);
  --lumo-body-text-color: hsla(218, 31%, 20%, 0.94);
  --lumo-secondary-text-color: hsla(218, 31%, 26%, 0.69);

  /* Typography */
  --lumo-font-family: "Inter", "Segoe UI", Roboto, "Helvetica Neue", Arial, sans-serif;
  --lumo-font-size-xl: 1.375rem;

  /* Component styles */
  --vaadin-input-field-height: 2.25rem;
  --vaadin-input-field-background: hsla(218, 31%, 35%, 0.1);
  --vaadin-input-field-border-color: hsla(218, 31%, 20%, 0.52);
}

/* Dark theme */
[theme~="dark"] {
  --lumo-primary-color: hsla(177, 35%, 55%, 1); /* Darker variant of primary */
}
```

## Value Conversion Guidelines

### Size Conversion (Only Non-Default Values)
- **Pixel to rem**: Divide by 16 (`36px` → `2.25rem`) - but only if different from Lumo default
- **Unitless numbers**: Treat as pixels for border-radius, spacing
- **Font sizes**: Convert to rem, but skip if matches Lumo default (16px = 1rem for font-size-m)
- **Line heights**: Use unitless values (`1.6`, `1.25`)

### Typography Parsing
Parse Figma Font() definitions:
```javascript
"Normal body text": "Font(family: \"Inter\", style: Regular, size: 16, weight: 400, lineHeight: 1.61)"
```
Extract:
- family: "Inter" → font-family (only if different from system stack)
- size: 16 → Skip --lumo-font-size-m if it equals default 1rem
- weight: 400 → font-weight (handled by components, not CSS variables)
- lineHeight: 1.61 → --lumo-line-height-m: 1.6 (only if different from default)

### Color values conversions
Always avoid changing the color syntax. There will be a separate step outside of this task that takes care of color format conversions.
Focus on mapping the colors to CSS variables.

### Color configurations
When assigning Lumo Primary, Error, Warning, Success or Contrast colors. Ensure all related color values are also assigned.
```css
/* Proper way to set colors is to take all related values into account */
  --lumo-primary-color: hsl(214, 100%, 48%);
  --lumo-primary-color-50pct: hsla(214, 100%, 49%, 0.76);
  --lumo-primary-color-10pct: hsla(214, 100%, 60%, 0.13);
  --lumo-primary-text-color: hsl(214, 100%, 43%);

/* Incorrect to only set some color values */
  --lumo-primary-color: hsl(214, 100%, 48%);
  --lumo-primary-color-10pct: hsla(214, 100%, 60%, 0.13);
```

### Suitable values for contrast color variables
- `--lumo-contrast-XXpct` color values can only have color values that have lightness channel value below 25%.
- Only exeption is in the scope of `theme="dark"` where the lightness channel has to be above 75%.

### Input field styling
- If `--vaadin-input-field-border-color` is set, ensure `--vaadin-input-field-border-width` is also set at least 1px.
- Use CSS variables for styling whenever possible. If writing custom styles of input fields do not target specific fields. Target `::part(input-field)`, `::part(label)` and `::part(value)` to keep styling of all inputs consistent.

### Component-Specific Variables (Direct Mapping + Vaadin MCP)
Many component variables are directly available in Figma with exact CSS variable names:

```javascript
// Direct variable mapping from Figma
"vaadin-input-field-height" → --vaadin-input-field-height
"vaadin-input-field-background" → --vaadin-input-field-background  
"vaadin-input-field-border-color" → --vaadin-input-field-border-color
"vaadin-input-field-border-width" → --vaadin-input-field-border-width
"vaadin-user-color-0" → --vaadin-user-color-0
"vaadin-user-color-1" → --vaadin-user-color-1
// ... etc
```

Map Figma component tokens to variables:
```css
/* Input Field Component */
--vaadin-input-field-height: 2.25rem;
--vaadin-input-field-background: hsla(218, 31%, 35%, 0.1);
--vaadin-input-field-border-color: hsla(218, 31%, 20%, 0.52);
--vaadin-input-field-border-width: 1px;

/* User Colors */
--vaadin-user-color-0: hsla(320, 87%, 46%, 1);
--vaadin-user-color-1: hsla(258, 85%, 42%, 1);
--vaadin-user-color-2: hsla(193, 86%, 34%, 1);
--vaadin-user-color-3: hsla(35, 100%, 34%, 1);
```
