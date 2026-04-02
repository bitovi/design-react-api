---
name: design-react-api
description: Design React components from Figma files. Use when given a Figma URL to analyze a design and propose React component architecture, props API, and variant handling. Outputs design analysis and suggested API, does not build components.
---

# Skill: Design React Components from Figma

This skill analyzes Figma designs and proposes React component architecture and props APIs. It helps bridge the gap between design and implementation by providing clear specifications before coding begins.

## When to Use

- User provides a Figma URL and wants to understand how to implement it as React
- Planning component architecture before implementation
- Determining what props a component should have based on Figma variants
- Deciding if a Figma component should be one or multiple React components

## What This Skill Does NOT Do

- Build or implement the actual React components
- Generate production code

## Required Inputs

1. **Figma URL**: Full URL like `https://figma.com/design/{fileKey}/{fileName}?node-id={nodeId}`

## Design Principle

**Follow FIGMA structure by default; document justified deviations**

When designing component APIs from Figma:

**Do:**
- Use the Figma design as the primary source of truth for component structure
- Map Figma variants and properties directly to props when they represent meaningful structural or behavioral differences
- Preserve the component hierarchy and variant intent shown in Figma
- Document any deviation from Figma with clear rationale in `proposed-api.md`

**Don't:**
- Use other component libraries as the primary reference over Figma
- Introduce extra props not represented in Figma unless needed for implementation requirements
- Mirror Figma literally when it would harm accessibility, semantics, maintainability, or project conventions

Allowed reasons to deviate (must be documented in `proposed-api.md`):
- Accessibility and semantic HTML requirements
- Reuse of existing shared/common project components
- API ergonomics and long-term maintainability
- Responsive/runtime behavior not fully represented in static Figma variants

The component API should mirror how the design is structured in Figma unless a documented exception is required.

**Example:**
- If Figma shows `Type="Mobile"` with vertical button layout and `Type="Desktop"` with horizontal layout
- Prefer a `type` prop that controls layout structure and alignment
- If implementation requires a different API shape, document why and how it still preserves the Figma intent

## Workflow Overview

```
┌─────────────────────────────────────────────────────────────────┐
│ 1. FETCH - Get Figma design context using MCP                  │
├─────────────────────────────────────────────────────────────────┤
│ 2. TOKENS - Get variable definitions to resolve design tokens  │
├─────────────────────────────────────────────────────────────────┤
│ 3. SAVE - Store design context in .temp/design-components/     │
├─────────────────────────────────────────────────────────────────┤
│ 4. ANALYZE - Review variants, properties, and nested components│
├─────────────────────────────────────────────────────────────────┤
│ 5. PROPOSE - Suggest component API(s) with props and types     │
└─────────────────────────────────────────────────────────────────┘
```

## Required Flow

**Follow this sequence exactly:**

1. **Run `get_design_context`** to fetch the structured representation for the exact node(s). If the response is too large or truncated, run `get_metadata` first to get the high-level node map, then re-fetch only the required node(s) with `get_design_context`.

2. **Run `get_variable_defs`** with the same `fileKey` to resolve design token names to their semantic meaning across modes (light, dark, brand). Use this to understand what each token represents so the proposed API notes can reference the correct project CSS variables rather than hardcoded hex fallbacks.

3. **Only after you have both** (`get_design_context` and `get_variable_defs`), proceed to the step-by-step instructions below.

## Step-by-Step Instructions

### Step 1: Parse Figma URL and Fetch Design Context

Extract from the URL:
- `fileKey`: The ID after `/design/` (or after `/branch/` if on a branch)
- `nodeId`: From `node-id=` query param (convert `123-456` → `123:456`)

Call `mcp_figma_get_design_context` with:
- `nodeId`: The extracted node ID
- `fileKey`: The extracted file key

### Step 2: Fetch Variable Definitions

Call `mcp_figma_get_variable_defs` with:
- `fileKey`: The extracted file key

This returns all design tokens and their values across every mode (e.g. light/dark, brand themes). Cross-reference the token names found in the `get_design_context` output against the project's CSS variables in `src/index.css`. Include any relevant token-to-project-variable mappings as notes in `proposed-api.md` rather than as a separate file.

### Step 3: Create Output Directory and Save Design Context

Create the output directory:
```
.temp/design-components/{COMPONENT_NAME}/
```

Where `{COMPONENT_NAME}` is derived from the Figma component name (kebab-case).

Save the design context to `.temp/design-components/{COMPONENT_NAME}/design-context.md`:

```markdown
# Design Context: {ComponentName}

## Figma Source
{original URL}

## Component Overview
{Brief description based on Figma data}

## Raw Design Data
{Full output from mcp_figma_get_design_context}
```

### Step 4: Analyze Figma Design

Extract and analyze:

1. **Variants and Variant Options**
   - List all variant properties (e.g., Size, State, Type)
   - List all options for each variant (e.g., Size: Small, Medium, Large)
   - Note how variants affect component structure, not just styling
     - Does the variant change element ordering?
     - Does it change flex direction or layout?
     - Does it add/remove elements?
     - Does it change text alignment or button positioning?

2. **Component Properties**
   - Boolean properties (e.g., "Has Icon", "Show Label")
   - String properties (e.g., "Label Text")
   - Instance swap properties (e.g., "Icon")

3. **Nested Components**
   - Child component instances
   - Repeated elements

4. **Text Layers**
   - Configurable text content
   - Alignment changes across variants

5. **Structural Differences**
   - Document how the DOM structure changes between variants

### Step 5: Propose Component API

Based on the analysis, create `.temp/design-components/{COMPONENT_NAME}/proposed-api.md`:

```markdown
# Proposed API: {ComponentName}

## Figma Source
{original URL}

## Summary
{One paragraph describing what this component does}

## Recommended Component Structure

{Explain if this should be one component or multiple, and why}

---

## Component: {ComponentName}

### Props Interface

\`\`\`typescript
interface {ComponentName}Props {
  // Mapped from Figma variant "Size"
  size?: 'sm' | 'md' | 'lg';
  
  // Mapped from Figma variant "Variant"
  variant?: 'primary' | 'secondary' | 'outline';
  
  // Mapped from Figma boolean "Disabled"
  disabled?: boolean;
  
  // Mapped from Figma text layer "Label"
  children: React.ReactNode;
  
  // Mapped from Figma instance "Icon"
  icon?: React.ReactNode;
}
\`\`\`

### Prop Details

| Prop | Type | Default | Figma Source | Notes |
|------|------|---------|--------------|-------|
| size | `'sm' \| 'md' \| 'lg'` | `'md'` | Variant: Size | Maps Small→sm, Medium→md, Large→lg |
| variant | `'primary' \| 'secondary'` | `'primary'` | Variant: Type | - |
| disabled | `boolean` | `false` | Boolean: Disabled | - |
| children | `React.ReactNode` | required | Text: Label | - |
| icon | `React.ReactNode` | `undefined` | Instance: Icon | Only shown when Has Icon=true |

### Excluded from Props (Handled by Tailwind/Internal State)

| Figma Property | Reason |
|----------------|--------|
| State: Hover | Tailwind `hover:` modifier |
| State: Pressed | Tailwind `active:` modifier |
| State: Focused | Tailwind `focus-visible:` modifier |

### Example Usage

\`\`\`tsx
<{ComponentName} size="lg" variant="primary">
  Click me
</{ComponentName}>

<{ComponentName} size="sm" icon={<IconPlus />}>
  Add Item
</{ComponentName}>
\`\`\`

---

## Additional Components (if applicable)

{If the Figma component should be split into multiple React components, document each one here with the same structure}
```

## Decision Guidelines

### When to Create Multiple Components

Create separate components when:
- Figma variants represent fundamentally different UI patterns (e.g., "Type: Text Input" vs "Type: Date Picker")
- Variants have completely different props/behavior
- One variant is a specialized version with unique functionality

Keep as one component when:
- Variants only affect styling (colors, sizes)
- All variants share the same props interface
- Behavior is consistent across variants

### Props to Include vs Exclude

**Include as props:**
- Variants that change content, behavior, OR structure
- Boolean toggles for optional elements
- Text content that should be configurable
- Instance swaps for customizable slots

**Exclude from props (handle internally):**
- Pseudo-state variants: `hover`, `pressed`, `focused`, `active`
- Interaction states: `loading` might be a prop, but visual feedback is internal
- Animation states

### Naming Conventions

| Figma | React Prop |
|-------|-----------|
| `Small`, `Medium`, `Large` | `'sm'`, `'md'`, `'lg'` |
| `Primary`, `Secondary` | `'primary'`, `'secondary'` |
| `Has Icon`, `Show Label` | `hasIcon: boolean`, `showLabel: boolean` |
| `Yes`/`No`, `True`/`False` | `boolean` |
| Title Case values | kebab-case or camelCase |

## Output Files

```
.temp/design-components/{component-name}/
├── design-context.md    # Raw Figma data + URL
└── proposed-api.md      # Suggested component API(s) + token notes
```

## Example Session

**User:** Design a React component from this Figma: https://figma.com/design/abc123/DesignSystem?node-id=100-200

**Agent:**
1. Fetches design context via MCP
2. Fetches variable definitions via MCP to understand token semantics across modes
3. Creates `.temp/design-components/button/`
4. Saves `design-context.md` with Figma data
5. Analyzes variants (Size, Type, State) and properties (Has Icon, Label)
6. Creates `proposed-api.md` with:
   - `ButtonProps` interface
   - Mapping table from Figma to props
   - Note that State: Hover/Pressed/Focused are handled by CSS
   - Example usage

**Agent Response:**
```
📋 Design analysis complete for Button!

Created: `.temp/design-components/button/`

## Proposed Structure
Single component: `Button`

## Props Summary
| Prop | Type | From Figma |
|------|------|------------|
| size | 'sm' \| 'md' \| 'lg' | Variant: Size |
| variant | 'primary' \| 'secondary' | Variant: Type |
| disabled | boolean | Boolean: Disabled |
| children | ReactNode | Text: Label |
| icon | ReactNode | Instance: Icon |

Excluded: Hover, Pressed, Focused states (CSS handles these)

See full details: `.temp/design-components/button/proposed-api.md`
```
