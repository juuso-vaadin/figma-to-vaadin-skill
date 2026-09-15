<img width="900" height="180" alt="code_gen_banner" src="https://github.com/user-attachments/assets/5256577f-4651-4e8f-8f69-4d0c5eee922f" />


# Figma to Vaadin agent skills
Collection of AI skills for translating Figma designs to Vaadin applications.

This repository contains four complementary skills that work with Figma MCP, Vaadin MCP, and Playwright MCP to streamline the design-to-code workflow — from implementing a design, through theming, to verifying the result actually matches.

### 1. Figma to Vaadin (UI implementation)
**Purpose:** Translate Figma designs to well-structured Vaadin Flow code

**Use this skill when you need to:**
- Implement UI components from Figma designs
- Convert design layouts to Vaadin views
- Generate semantic and accessible Vaadin code

**Workflow:**
1. Learn the project first — its Vaadin version, theme, existing views and conventions — so the generated code fits in rather than reinventing patterns the project already has
2. Fetch the design from Figma, breaking large frames into smaller regions so nothing gets silently truncated
3. Measure the design carefully (spacing, borders, colors, sizing) using both the design data and a screenshot, since small details like a 1px border are easy to miss
4. Build the layout with Vaadin's own layout components rather than hand-rolled CSS
5. Choose and style the right Vaadin components, matching the variants and behavior the design calls for
6. Check the generated code back against the original measurements, then compile it
7. Hand off to the visual verification skill (below) to confirm the running view matches the design

**Example prompt:**
```
Implement the selection in Figma. Follow the provided guidelines.
```

### 2. Figma to Lumo Theme
**Purpose:** Map Figma design tokens to Lumo CSS variables

**Use this skill when you need to:**
- Apply custom theme from Figma to your Vaadin app
- Generate CSS variable declarations for Lumo
- Customize colors, typography, spacing from design system

**Workflow:**
1. Extract design tokens from Figma using `get_variable_defs`
2. Categorize tokens by type (colors, typography, spacing, etc.)
3. Map to corresponding Lumo CSS variables  
4. Generate `styles.css` with only non-default values

**Example prompt:**
```
Apply the Figma design tokens to Lumo theme. Follow the provided guidelines.
```

### 3. Figma to Aura Theme
**Purpose:** Map a Figma Aura design system to the Vaadin Aura theme

**Use this skill when you need to:**
- Configure the Aura theme (Vaadin's default theme from 25.0 onwards) to match a Figma design
- Generate Aura CSS custom properties (accent color, background, density, radius, etc.) from Figma variables
- Support light/dark mode variants defined in Figma

**Workflow:**
1. Extract Figma variables from all available modes (light/dark) using `use_figma` and `get_variable_defs`
2. Map variables to Aura's higher-level properties — not a 1:1 copy, since Aura derives many values from a small set of inputs
3. Infer visual properties (density, radius, surface level) from `get_design_context`
4. Generate the Aura theme CSS file with only non-default values

**Example prompt:**
```
Set up the Aura theme to match this Figma design. Follow the provided guidelines.
```

### 4. Vaadin Visual Verification
**Purpose:** Check a freshly implemented Vaadin view against the Figma design it came from

**Use this skill when you need to:**
- Confirm an implementation actually matches the design, rather than just eyeballing it
- Catch layout, spacing, color, typography and component-fidelity issues after `figma-to-vaadin` finishes a view
- Get a prioritized list of concrete visual differences instead of a vague "looks good"

**Workflow:**
1. Run the app and open the implemented view in a browser
2. Take a screenshot of the Figma design as the reference, and of the running view as the implementation
3. Compare them side by side, region by region — layout, viewport/scroll, typography, color, and component fidelity — while also checking the browser console for errors
4. Report findings as a prioritized list (blocker / high / low) with what's expected vs. actual and a suggested fix

This skill only observes and reports; it doesn't edit code itself. It's normally invoked automatically as the last step of `figma-to-vaadin`, but can also be run standalone against any already-implemented view.

**Example prompt:**
```
Verify that the Orders view matches the Figma design it was implemented from.
```

## How to Use

### Prerequisites
- AI coding assistant with MCP support (GitHub Copilot, Claude Code, or similar)
- Connection to the Figma MCP and Vaadin MCP servers, and the Playwright MCP server for visual verification

### Setup
The agent skill definitions are found in `skills/`. The skill structure is generic, but different tools may load the skills slightly differently.

The project's `.mcp.json` at the repository root configures all three MCP servers. Figma MCP can be used **remotely** (link to Figma file) or **locally** (Figma Desktop app).

### Usage
1. Make a selection in Figma.
2. Right click to see context menu and select "Copy link to selection". Alternatively you can copy the URL with node ID from browser's address bar.
3. In your code editor, use the appropriate slash command in the AI chat and paste the URL pointing to a Figma node:
   - `/figma-to-vaadin` for UI code implementation
   - `/figma-to-lumo-theme` for configuring Lumo theme
   - `/figma-to-aura-theme` for configuring Aura theme
   - `/vaadin-visual-verification` to check an existing implementation against its design
4. The AI agent will follow the skill guidelines to generate accurate code or styles.

#### Example prompt
```
/figma-to-vaadin Create new view based on https://www.figma.com/design/ExAMpLeID/My-Figma-File?node-id=1-234&t=DABCINP9G1B1d8Wi-1
```

## Useful Links
- [Vaadin MCP Documentation](https://mcp.vaadin.com/docs/)
- [Figma MCP Developer Documentation](https://developers.figma.com/docs/figma-mcp-server)
