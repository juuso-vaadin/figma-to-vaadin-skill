# Figma to Vaadin agent skills
Collection of AI skills for translating Figma designs to Vaadin applications.

This repository contains two complementary skills that work with Figma MCP and Vaadin MCP to streamline the design-to-code workflow:

### 1. Figma to Vaadin (UI implementation)
**Purpose:** Translate Figma designs to well-structured Vaadin Flow code

**Use this skill when you need to:**
- Implement UI components from Figma designs
- Convert design layouts to Vaadin views
- Generate semantic and accessible Vaadin code

**Workflow:**
1. Extract design context from Figma selection using Figma MCP
2. Check component annotations for implementation guidance
3. Fetch relevant Vaadin documentation via Vaadin MCP
4. Generate proper Vaadin components with correct themes

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

## How to Use

### Prerequisites
- AI coding assistant with MCP support (GitHub Copilot, Claude Code, or similar)
- Desktop version of Figma
- Connection to Figma MCP and Vaadin MCP servers

### Setup
The skill definitions in `.github/skills/` are designed for GitHub Copilot in VS Code, but can easily be adapted for other AI coding assistants like Claude Code or other tooling that supports custom instructions and MCP.

Sample configuration files for VS Code are provided in the `.vscode` directory.

### Usage
1. Make a selection in Figma. (Figma MCP only supports a single frame selection).
2. In your code editor, use the appropriate slash command in the AI chat:
   - `/figma-to-vaadin` for UI code implementation
   - `/figma-to-lumo-theme` for configuring Lumo theme
3. The AI will follow the skill guidelines to generate accurate code or styles

## Technical Details

Both skills follow a consistent workflow:
- Always extract design information from Figma MCP first
- Fetch relevant Vaadin documentation before generating code
- Focus on accuracy over speed
- Skip tests and validations (manual verification is expected)

## Useful Links
- [Vaadin MCP Documentation](https://mcp.vaadin.com/docs/)
- [Figma MCP Developer Documentation](https://developers.figma.com/docs/figma-mcp-server)
