# AI prompts for Vaadin
Collection of AI prompts for creating Vaadin UI's.

## Figma MCP to Vaadin
Figma MCP allows passing Figma designs to AI coding assistant.

The prompts define a workflow where:
1. Details of active selection in Figma is extracted using **Figma MCP**
2. Relevant parts of the Vaadin documentation is fetched with **Vaadin MCP**
3. Code is generated only after documentation is reviewed
4. Any tests or validations are skipped

### How to use
I'm using the prompts with VS Code and GitHub Copilot along with the desktop version of Figma.
Project contains a sample configuration file for VS Code.

Once connected to MCP servers, AI assistant can be prompted with
```
Implement the selection in Figma. Follow the provided guidelines.
```

### Useful links
- Vaadin MCP: https://mcp.vaadin.com/docs/
- Figma MCP developer documentation: https://developers.figma.com/docs/figma-mcp-server
