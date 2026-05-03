---
name: ai-component-metadata-yaml
version: 1.0.0
author: Robin Di Capua
based_on: "ai-component-metadata-json by Robin Di Capua, itself adapted from ai-component-metadata by Cristian Morales — https://github.com/cris-achiardi/claude-skills/tree/main/skills/ai-component-metadata"
changes: "Converted from JSON metadata format to YAML for ~25% token reduction on deeply nested component data. Schema validation preserved via yaml-language-server comment."
license: MIT
description: Generate AI-ready metadata for design system components as YAML files with JSON Schema validation. Produces structured, tool-agnostic metadata that helps AI understand when and how to use components correctly. Lower token cost than JSON equivalents, human-readable, and IDE-validated via the yaml-language-server comment.
---

**Version:** 1.0.0  
**Last Updated:** 2026-05-03

# AI Component Metadata Generator (YAML)

Generate structured, AI-consumable metadata as `.metadata.yaml` files for design system components. Uses JSON Schema for validation via the `yaml-language-server` comment — no TypeScript build step required.

## Why YAML over JSON?

- **~25% fewer tokens** — No quotes on keys, no brackets, less structural punctuation. Savings compound on deeply nested data.
- **Multi-line strings** — JSX composition examples are readable with YAML block scalars instead of escaped `\n`.
- **Schema-validated** — The `yaml-language-server` comment gives IDE autocomplete + validation (requires the Red Hat YAML VS Code extension).
- **Universal** — Readable by any language, tool, or pipeline.

## Quick Start

1. Copy the template below into `[component-name].metadata.yaml`
2. The `yaml-language-server` comment activates IDE validation automatically
3. Fill in component data

```yaml
# yaml-language-server: $schema=../../../../../.agent/skills/ai-component-metadata-yaml/schemas/component-metadata.schema.json
component:
  name: ComponentName
  category: atoms
  description: Brief description
  type: interactive
  path: packages/design-system/src/components/component-name/component-name.jsx
usage:
  useCases:
    - primary-use
  requiredProps: []
  optionalProps: []
  commonPatterns: []
  antiPatterns: []
composition:
  slots: {}
  nestedComponents: []
  commonPartners: []
  parentConstraints: []
  dependencies: []
behavior:
  states: []
  interactions: {}
  responsive: {}
accessibility:
  role: button
  keyboardSupport: ""
  screenReader: ""
  focusManagement: ""
  wcag: AA
variants: {}
aiHints:
  priority: medium
  keywords: []
  context: ""
```

## Core Workflow

### 1. Analyze Component Structure
Identify:
- Component composition (slots, children)
- Available variants and states
- Props and their types
- Accessibility attributes

### 2. Generate Metadata
Create a `.metadata.yaml` file. The `yaml-language-server` comment on line 1 enables:
- **Autocomplete** — Press Ctrl+Space in your IDE for property suggestions
- **Validation** — Red squiggles for invalid values
- **Hover docs** — Schema descriptions appear on hover

Requires the **Red Hat YAML** VS Code extension (`redhat.vscode-yaml`).

### 3. Multi-line JSX patterns
Use YAML block scalars for readable composition examples:

```yaml
commonPatterns:
  - name: full-header
    description: Logo, nav items, and a CTA button
    composition: |
      <Header
        logo={<img src="/logo.svg" alt="Brand" />}
        navItems={[
          { label: "Work", href: "/work" },
          { label: "About", href: "/about", active: true }
        ]}
        cta={{ label: "Get started", href: "/signup" }}
      />
```

### 4. Validate Metadata
- IDE validates automatically via the `yaml-language-server` comment
- Schema is also mapped in `.vscode/settings.json` as a fallback
- CLI validation (requires `js-yaml` + `ajv`):
  ```bash
  npx js-yaml path/to/file.metadata.yaml | npx ajv validate \
    -s .agent/skills/ai-component-metadata-yaml/schemas/component-metadata.schema.json \
    -d /dev/stdin
  ```

## Folder Structure (Required)

Co-locate the metadata file alongside the component code:

```text
packages/design-system/src/components/[component-name]/
├── index.js                         (exports the component)
├── [component-name].jsx             (the component code)
└── [component-name].metadata.yaml  (the AI metadata)
```

## Schema Reference

The JSON Schema is located at:
```
.agent/skills/ai-component-metadata-yaml/schemas/component-metadata.schema.json
```

### Required sections
| Section | Purpose |
|---|---|
| `component` | Name, category (atoms/molecules/organisms), type, description |
| `usage` | Use cases, patterns, anti-patterns |
| `aiHints` | Priority, keywords, natural language context |

### Optional sections
| Section | Purpose |
|---|---|
| `composition` | Slots, nested components, dependencies |
| `behavior` | States, interactions, responsive rules |
| `accessibility` | ARIA roles, keyboard support, WCAG level |
| `variants` | Variant dimensions and their options |

## Component Categories

- **atoms**: Basic building blocks (Button, Text, Input)
- **molecules**: Simple combinations (Card, Chip, FormField)
- **organisms**: Complex components (Header, Table, Form)

## YAML quoting rules

Quote strings that contain: colons (`:`), `#`, `>`, `|`, `{`, `}`, `[`, `]`, `&`, `*`, `!`, or that start with `@`, `` ` ``.

Simple strings like prop names and short labels do not need quotes.

## Best Practices

1. **Keep examples real** — Use actual, runnable JSX in `commonPatterns`
2. **Use block scalars for multi-line JSX** — `|` for literal, `>` for folded
3. **Focus on patterns** — Document common usage patterns
4. **Include anti-patterns** — Help AI avoid mistakes
5. **Be consistent** — All components should use the same structure

## Success Metrics

Your metadata is effective when:
- AI uses existing components instead of recreating
- Correct variants are selected based on context
- Accessibility is maintained in generated code
- Patterns are consistent across AI outputs
- Token usage is lower than JSON equivalents
