---
name: ai-component-metadata-yaml
version: 2.0.0
author: Robin Di Capua
based_on: "ai-component-metadata-json by Robin Di Capua, itself adapted from ai-component-metadata by Cristian Morales — https://github.com/cris-achiardi/claude-skills/tree/main/skills/ai-component-metadata"
changes: "v2.0.0: split the single .metadata.yaml into two files — a descriptive .spec.yaml (API, variants, a11y, examples) and a normative .governance.yaml (citable rules: anti-patterns, parent constraints). Previously: converted from JSON to YAML for ~25% token reduction."
license: MIT
description: Generate AI-ready component context as two YAML files with JSON Schema validation — a descriptive spec (<component>.spec.yaml) and, when the component has citable rules, a governance file (<component>.governance.yaml). A spec describes, governance prescribes. Lower token cost than JSON equivalents, human-readable, and IDE-validated via the yaml-language-server comment.
---

**Version:** 2.0.0  
**Last Updated:** 2026-07-15

# AI Component Spec + Governance Generator (YAML)

Generate structured, AI-consumable component context as **two co-located YAML files**, validated via the `yaml-language-server` comment — no TypeScript build step required:

- `<component>.spec.yaml` — **descriptive**: what the component *is* (API, variants, accessibility, usage examples, AI hints). A spec cannot be violated.
- `<component>.governance.yaml` — **normative**: citable rules about what you *must or must not do* (anti-patterns, parent constraints). Only create it when the component has at least one rule.

The split follows one test: **a spec describes, governance prescribes.**

## Why YAML over JSON?

- **~25% fewer tokens** — No quotes on keys, no brackets, less structural punctuation. Savings compound on deeply nested data.
- **Multi-line strings** — JSX composition examples are readable with YAML block scalars instead of escaped `\n`.
- **Schema-validated** — The `yaml-language-server` comment gives IDE autocomplete + validation (requires the Red Hat YAML VS Code extension).
- **Universal** — Readable by any language, tool, or pipeline.

## Quick Start

1. Copy the spec template below into `[component-name].spec.yaml`
2. The `yaml-language-server` comment activates IDE validation automatically
3. Fill in component data
4. If the component has citable rules, also create `[component-name].governance.yaml` from the governance template

```yaml
# yaml-language-server: $schema=../../../../../.agent/skills/ai-component-metadata-yaml/schemas/component-spec.schema.json
component:
  name: ComponentName
  category: atoms
  description: Brief description
  type: interactive
  path: packages/design-system/src/components/component-name/component-name.jsx
  figma:                        # optional — omit if no Figma source is known
    fileKey: ""                 # from the Figma file URL: figma.com/design/FILE_KEY/...
    nodeId: ""                  # from ?node-id= in the URL when the component is selected
    componentKey: ""            # stable REST API key — use this for durable references
usage:
  useCases:
    - primary-use
  requiredProps: []
  optionalProps: []
  commonPatterns: []
composition:
  slots: {}
  nestedComponents: []
  commonPartners: []
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

Governance template (only when the component has rules):

```yaml
# yaml-language-server: $schema=../../../../../.agent/skills/ai-component-metadata-yaml/schemas/component-governance.schema.json
component:
  name: ComponentName
  scope: XXX                    # scope code from the governance registry; prefixes every rule id
rules:
  - id: XXX-1
    kind: anti-pattern          # a usage to avoid
    scenario: What NOT to do.
    reason: Why it's wrong.
    alternative: What to do instead.
    # severity: warning         # optional — defaults: anti-pattern → info, parent-constraint → warning
  - id: XXX-2
    kind: parent-constraint     # variants forbidden in a named context
    context: named-parent-context
    forbidden:
      - variant: danger
```

Rule ids are authored and stable — never renumber, and never reuse a retired id. To repeal a rule, add `status: repealed` and leave it in place.

## Core Workflow

### 1. Analyze Component Structure
Identify:
- Component composition (slots, children)
- Available variants and states
- Props and their types
- Accessibility attributes

### 2. Resolve Figma references (if available)

If a Figma file or component URL is provided, extract and populate the `component.figma` block:

- **`fileKey`** — the path segment after `/design/` in the file URL: `figma.com/design/FILE_KEY/...`
- **`nodeId`** — the `?node-id=` value when the component is selected in Figma. The URL uses hyphens (`1-234`); convert to colons (`1:234`) for the REST API. Store whichever format the project uses consistently.
- **`componentKey`** — stable across renames and file moves. Retrieve from the Figma REST API (`GET /v1/files/:key/components`) or from a connected Figma MCP tool if available.

If no Figma source is known, omit the `figma` block entirely — do not include it with empty strings.

### 3. Generate the spec (and governance file, if rules exist)
Create the `.spec.yaml` file — and a `.governance.yaml` beside it when the component has citable rules. The `yaml-language-server` comment on line 1 enables:
- **Autocomplete** — Press Ctrl+Space in your IDE for property suggestions
- **Validation** — Red squiggles for invalid values
- **Hover docs** — Schema descriptions appear on hover

Requires the **Red Hat YAML** VS Code extension (`redhat.vscode-yaml`).

### 4. Multi-line JSX patterns
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

### 5. Validate
- IDE validates automatically via the `yaml-language-server` comment
- Schemas are also mapped in `.vscode/settings.json` as a fallback
- CLI validation (requires `js-yaml` + `ajv`):
  ```bash
  npx js-yaml path/to/file.spec.yaml | npx ajv validate \
    -s .agent/skills/ai-component-metadata-yaml/schemas/component-spec.schema.json \
    -d /dev/stdin
  ```

## Folder Structure (Required)

Co-locate both files alongside the component code:

```text
packages/design-system/src/components/[component-name]/
├── index.js                            (exports the component)
├── [component-name].jsx                (the component code)
├── [component-name].spec.yaml          (descriptive spec)
└── [component-name].governance.yaml    (normative rules — only if any exist)
```

## Schema Reference

The JSON Schemas are located at:
```
.agent/skills/ai-component-metadata-yaml/schemas/component-spec.schema.json
.agent/skills/ai-component-metadata-yaml/schemas/component-governance.schema.json
```

### Required spec sections
| Section | Purpose |
|---|---|
| `component` | Name, category (atoms/molecules/organisms), type, description |
| `usage` | Use cases, patterns |
| `aiHints` | Priority, keywords, natural language context |

### Optional spec sections
| Section | Purpose |
|---|---|
| `composition` | Slots, nested components, dependencies |
| `behavior` | States, interactions, responsive rules |
| `accessibility` | ARIA roles, keyboard support, WCAG level |
| `variants` | Variant dimensions and their options |

### Governance file sections
| Section | Purpose |
|---|---|
| `component` | Name + scope code (prefixes every rule id) |
| `rules` | Citable rules — `kind: anti-pattern` or `kind: parent-constraint` |

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
4. **Encode anti-patterns as governance rules** — Help AI avoid mistakes, with stable citations
5. **Be consistent** — All components should use the same structure
6. **Keep the split clean** — nothing normative in the spec, nothing descriptive in the governance file

## Success Metrics

Your spec + governance files are effective when:
- AI uses existing components instead of recreating
- Correct variants are selected based on context
- Accessibility is maintained in generated code
- Patterns are consistent across AI outputs
- Token usage is lower than JSON equivalents
