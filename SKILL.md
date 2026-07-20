---
name: component-spec
version: 3.0.0
author: Robin Di Capua
based_on: "ai-component-metadata-yaml by Robin Di Capua, itself adapted from ai-component-metadata by Cristian Morales — https://github.com/cris-achiardi/claude-skills/tree/main/skills/ai-component-metadata"
changes: "v3.0.0: extracted the governance half. This skill now generates ONLY the descriptive <component>.spec.yaml. Component-tier governance rules (anti-patterns, parent constraints) moved to the governance-authoring skill, which owns every governance file format across tiers. v2.0.0: split the single .metadata.yaml into .spec.yaml + .governance.yaml. v1: converted from JSON to YAML for ~25% token reduction."
license: MIT
description: Generate an AI-ready, descriptive component spec as a single YAML file (<component>.spec.yaml) with JSON Schema validation — API, variants, accessibility, usage examples, AI hints. A spec describes; it cannot be violated. Normative rules ("must / must not") are NOT this skill's job — those are authored by the governance-authoring skill. After writing a spec, offer to encode the component's governance rules now or defer them. Lower token cost than JSON, human-readable, IDE-validated via the yaml-language-server comment.
---

**Version:** 3.0.0  
**Last Updated:** 2026-07-20

# AI Component Spec Generator (YAML)

Generate a structured, AI-consumable **descriptive** spec for a component as a
single YAML file, validated via the `yaml-language-server` comment — no
TypeScript build step required:

- `<component>.spec.yaml` — **descriptive**: what the component *is* (API,
  variants, accessibility, usage examples, AI hints). A spec cannot be
  violated.

This skill is **spec-only**. It sits on one side of the design system's
foundational line — **a spec describes, governance prescribes.** The normative
side (citable rules about what you *must or must not do* — component
anti-patterns, parent constraints, and every composition/journey rule) is
owned by the **governance-authoring** skill. See
[Hand off governance](#6-hand-off-governance) below for the flow.

## Why YAML over JSON?

- **~25% fewer tokens** — No quotes on keys, no brackets, less structural punctuation. Savings compound on deeply nested data.
- **Multi-line strings** — JSX composition examples are readable with YAML block scalars instead of escaped `\n`.
- **Schema-validated** — The `yaml-language-server` comment gives IDE autocomplete + validation (requires the Red Hat YAML VS Code extension).
- **Universal** — Readable by any language, tool, or pipeline.

## Quick Start

1. Copy the spec template below into `[component-name].spec.yaml`
2. The `yaml-language-server` comment activates IDE validation automatically
3. Fill in component data
4. When the spec is done, run the [governance hand-off](#6-hand-off-governance)

```yaml
# yaml-language-server: $schema=../../../../../.agent/skills/component-spec/schemas/component-spec.schema.json
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

> **Nothing normative in the spec.** Do not add "must" / "must not" rules,
> anti-patterns, or forbidden variants here — they belong in the component's
> `.governance.yaml`, authored by the governance-authoring skill. Keeping the
> spec purely descriptive is what makes it safe for any agent to trust.

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

### 3. Generate the spec
Create the `.spec.yaml` file. The `yaml-language-server` comment on line 1 enables:
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
    -s .agent/skills/component-spec/schemas/component-spec.schema.json \
    -d /dev/stdin
  ```

### 6. Hand off governance

A spec is descriptive only — it can't express "this component must not use the
`danger` variant on a confirmation screen." Those rules live in a separate
`<component>.governance.yaml`, owned by the **governance-authoring** skill.

Once the spec is written, **ask the user whether to encode the component's
governance rules now or later:**

- **"Encode them now"** → hand off to the **governance-encode** skill (the
  write-path process: it gate-checks the rule, classifies the tier, assigns a
  stable citation id, and — for a component rule — authors it via the
  **governance-authoring** skill's component-governance format, then runs
  `npm run sync:governance`).
- **"Later"** → stop here. The spec stands on its own; a component with no
  rules simply has no governance file. Governance can always be added later
  via governance-encode when a real rule emerges (which is the common case —
  most rules are discovered after a misuse, not at authoring time).

Do not author governance YAML from this skill. If the user wants rules encoded,
route through governance-encode / governance-authoring so tier classification
and citation assignment stay in one place.

## Folder Structure (Required)

Co-locate the spec alongside the component code. A `.governance.yaml` appears
beside it only when the component has rules — authored by the
governance-authoring skill, not this one:

```text
packages/design-system/src/components/[component-name]/
├── index.js                            (exports the component)
├── [component-name].jsx                (the component code)
├── [component-name].spec.yaml          (descriptive spec — THIS skill)
└── [component-name].governance.yaml    (normative rules — governance-authoring skill, only if any exist)
```

## Schema Reference

The JSON Schema is located at:
```
.agent/skills/component-spec/schemas/component-spec.schema.json
```

The component-governance schema now lives with the governance-authoring skill
(`.agent/skills/governance-authoring/schemas/component-governance.schema.json`).

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
4. **Keep the spec purely descriptive** — nothing normative; route rules to governance-authoring via governance-encode
5. **Be consistent** — All components should use the same structure

## Success Metrics

Your spec files are effective when:
- AI uses existing components instead of recreating
- Correct variants are selected based on context
- Accessibility is maintained in generated code
- Patterns are consistent across AI outputs
- Token usage is lower than JSON equivalents
