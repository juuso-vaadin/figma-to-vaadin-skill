<img width="900" height="180" alt="code_gen_banner" src="https://github.com/user-attachments/assets/5256577f-4651-4e8f-8f69-4d0c5eee922f" />


# Figma to Vaadin agent skills
Collection of AI skills for translating Figma designs into Vaadin applications, built on the Figma MCP, Vaadin MCP and Playwright MCP servers.

**Start with `figma-to-vaadin-orchestrator`.** It is the entry point for all Figma-to-Vaadin work: it runs the three phases in order and delegates each to the skill that owns it.

```
figma-to-vaadin-orchestrator
├── Phase 1 — Theme configuration → figma-to-aura-theme  (Vaadin 25+, Aura)
│                                   figma-to-lumo-theme  (Vaadin 24, or Lumo on 25+)
├── Phase 2 — UI implementation   → figma-to-vaadin
└── Phase 3 — Verification        → vaadin-visual-verification
                                    findings routed back to phase 1 or 2
```

The orchestrator picks the right theme skill from the project's Vaadin version, configures the theme once per Figma file, restarts the app between phases that change Java code, routes verification findings back to the phase that owns the fix, and tracks progress in `.figma-to-vaadin/state.json` at the project root.

## The skills

| Skill | Does |
|---|---|
| `figma-to-vaadin-orchestrator` | Entry point — runs the phases below in order and keeps the shared state |
| `figma-to-aura-theme` | Maps Figma variables to Aura theme properties, including light/dark modes |
| `figma-to-lumo-theme` | Maps Figma design tokens to Lumo CSS variables in `styles.css` |
| `figma-to-vaadin` | Translates one Figma frame into Vaadin Flow (Java) view code |
| `vaadin-visual-verification` | Renders the running view and reports how it differs from the design |

## How to Use

### Prerequisites
- AI coding assistant with MCP and skill support (Claude Code, GitHub Copilot, or similar)
- Connection to the Figma MCP, Vaadin MCP and Playwright MCP servers
- A buildable, runnable Vaadin Flow (Java) project

### Setup
The skill definitions are in `skills/`. The structure is generic, but different tools may load skills slightly differently.

The `.mcp.json` at the repository root configures all three MCP servers. Figma MCP can be used **remotely** (link to Figma file) or **locally** (Figma Desktop app).

### Usage
1. Make a selection in Figma, right click and select "Copy link to selection".
2. In your code editor, paste the URL into the AI chat:

```
/figma-to-vaadin-orchestrator Implement https://www.figma.com/design/ExAMpLeID/My-Figma-File?node-id=1-234
```

3. Run it again for each further screen or region — the theme phase is skipped once it's configured for that Figma file.

## Useful Links
- [Vaadin MCP Documentation](https://mcp.vaadin.com/docs/)
- [Figma MCP Developer Documentation](https://developers.figma.com/docs/figma-mcp-server)
