# design-react-api

A spec-driven skill that treats a Figma design as the source of truth and produces a React component API specification before any implementation begins.

Given a Figma URL, the skill analyzes the component's variants, properties, and structure, then outputs a `proposed-api.md` — a TypeScript props interface with a full mapping table from Figma properties to React props. No code is generated. The spec comes first.

---

## What It Does

1. **Fetches** the Figma design context and variable definitions via the Figma MCP server
2. **Analyzes** all variant properties, boolean toggles, text layers, instance swaps, and structural differences across variants
3. **Proposes** a React component API: props interface, type definitions, variant-to-prop mappings, and usage examples
4. **Documents** any deviation from the Figma structure with explicit rationale

The output is a human-readable contract that a developer (or a follow-up build skill) implements against — not production code.

---

## Why Spec-First?

Jumping straight from Figma to code often leads to prop APIs that don't reflect design intent, missed variants, or components that need breaking changes after the first review. This skill separates the **what** (the spec) from the **how** (the implementation), so the API is agreed on before a line of component code is written.

---

## Prerequisites

- A Figma file with a component or frame you want to implement
- The **Figma MCP server** connected to your AI agent (see below)
- A Figma personal access token — get one from **Figma → Account Settings → Personal Access Tokens**

> This skill uses the Figma MCP server, not a Figma plugin. The AI agent reads directly from your Figma file through the MCP connection — no plugin panel or Figma Community install required.

---

## Connecting the Figma MCP Server

### Using Claude.ai (browser)

1. Go to **Claude.ai → Settings → Integrations**
2. Find Figma and click **Connect**
3. Authorize with your Figma account

### Using Claude Code (terminal)

Add to your Claude config file (`~/.claude.json` or `claude_desktop_config.json`):

```json
{
  "mcpServers": {
    "figma": {
      "url": "https://mcp.figma.com/mcp",
      "headers": {
        "Authorization": "Bearer YOUR_FIGMA_ACCESS_TOKEN"
      }
    }
  }
}
```

### Using VS Code + GitHub Copilot

Add to `.vscode/mcp.json`:

```json
{
  "servers": {
    "figma": {
      "type": "sse",
      "url": "https://mcp.figma.com/sse",
      "headers": {
        "Authorization": "Bearer YOUR_FIGMA_ACCESS_TOKEN"
      }
    }
  }
}
```

---

## Installation

1. Clone the repo

```bash
git clone https://github.com/bitovi/design-react-api.git
cd design-react-api
```

2. Add `SKILL.md` to your AI agent's skills directory. For Claude Code, place it under `.github/skills/` in your project.

---

## How to Trigger It

Paste a Figma URL and ask the agent to design the React component API:

> "Design a React component from this Figma: https://figma.com/design/abc123/DesignSystem?node-id=100-200"

> "Analyze this Figma component and propose a props API before we build it."

> "What should the React API look like for this Figma component?"

---

## Output

```
.temp/design-components/{component-name}/
├── design-context.md    # Raw Figma data + source URL
└── proposed-api.md      # Proposed props interface, variant mappings, usage examples
```

**Example `proposed-api.md` content:**

```typescript
interface ButtonProps {
  // Mapped from Figma variant "Size"
  size?: 'sm' | 'md' | 'lg';

  // Mapped from Figma variant "Type"
  variant?: 'primary' | 'secondary' | 'outline';

  // Mapped from Figma boolean "Disabled"
  disabled?: boolean;

  // Mapped from Figma text layer "Label"
  children: React.ReactNode;

  // Mapped from Figma instance "Icon"
  icon?: React.ReactNode;
}
```

Pseudo-states like `hover`, `pressed`, and `focused` are excluded from props and documented as handled by CSS/Tailwind.

---

## MCP Tools Used

- `get_design_context` — Fetch component structure, variants, and properties
- `get_variable_defs` — Resolve design tokens to semantic names across modes (light/dark/brand)

---

## Contributing

Contributions are welcome.

1. Fork the repo
2. Create a branch: `git checkout -b feature/your-improvement`
3. Make your changes to `SKILL.md`
4. Test against at least one multi-variant Figma component
5. Open a pull request with a description of what changed and why

---

## License

[MIT](LICENSE)
