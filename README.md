# Component Spec (YAML) — Claude Code Skill

A [Claude Code](https://claude.ai/code) skill that generates structured,
AI-readable **descriptive** context for design system components as a single
co-located YAML file, validated against a JSON Schema:

- `<component>.spec.yaml` — **descriptive**: what the component *is* (API,
  variants, accessibility, usage examples, AI hints). A spec cannot be
  violated.

This skill is **spec-only**. It follows one test — **a spec describes,
governance prescribes** — and owns only the descriptive half. Normative rules
("must / must not": component anti-patterns, parent constraints, and every
composition/journey rule) are authored by the sibling
[governance-authoring](https://github.com/robindicapua/governance-authoring)
skill, driven by the project's `governance-encode` write-path skill. After a
spec is written, this skill offers to encode the component's governance rules
now or defer them.

**Based on** [ai-component-metadata](https://github.com/cris-achiardi/claude-skills/tree/main/skills/ai-component-metadata)
by Cristian Morales. Converted to YAML for ~25% token reduction on deeply
nested component data; v2 split spec from governance into two files; **v3
extracted governance entirely into the governance-authoring skill**, leaving
this skill spec-only.

---

## What it does

When you ask Claude to generate or update component context, this skill
instructs it to produce a `.spec.yaml` for the component. It tells AI agents:

- **When** to reach for this component (use cases, keywords, priority)
- **How** to use it correctly (required/optional props, composition patterns)
- **How it behaves** (states, interactions, responsive rules)
- **Accessibility requirements** (ARIA role, keyboard support, WCAG level)
- **Available variants** and what each one is for

**What it deliberately does not do:** encode rules. "What to avoid" —
anti-patterns, forbidden variants, one-primary-CTA-per-step, and any other
normative constraint — is not a spec's job. Those are citable governance rules
authored by the [governance-authoring](https://github.com/robindicapua/governance-authoring)
skill. For multi-step flows and composition constraints that span multiple
components (e.g. "a checkout flow may only have one primary CTA visible at a
time"), that skill is the one to use — its content model is rules-first across
whole compositions.

---

## Why YAML over JSON?

| | JSON | YAML |
|---|---|---|
| Token cost | Baseline | ~25% fewer tokens |
| Multi-line code examples | Escaped `\n` strings | YAML block scalars (`\|`) |
| Schema validation | External tooling required | Inline via `yaml-language-server` comment |
| Human readability | Moderate | High |
| Machine readability | Universal | Universal |

The savings compound on deeply nested component data and are meaningful at
scale when AI agents load metadata for a full component library in a single
context window.

---

## Prerequisites

- [Claude Code](https://claude.ai/code) CLI installed
- A design system with components in a consistent folder structure
- (Optional) [Red Hat YAML](https://marketplace.visualstudio.com/items?itemName=redhat.vscode-yaml) VS Code extension for IDE validation and autocomplete

---

## Installation

### 1. Copy the skill into your project

Place this skill folder anywhere in your repo. The conventional location is:

```
.agent/skills/component-spec/
```

If you use a different path, update the schema references described in the [Customization](#customization) section below.

### 2. Tell Claude Code about it

In your project's `CLAUDE.md` or `AGENTS.md`, add a line pointing to your skills directory:

```markdown
Before starting any task, check if a relevant skill exists in `.agent/skills/`.
If one matches, read it and follow its instructions.
```

Claude Code reads `CLAUDE.md` automatically at session start, so this is enough — no plugin configuration required.

### 3. (Optional) Enable IDE validation

Add the schema mapping to `.vscode/settings.json` so the `yaml-language-server` comment works without the Red Hat extension needing to resolve relative paths:

```json
{
  "yaml.schemas": {
    ".agent/skills/component-spec/schemas/component-spec.schema.json": [
      "packages/ui/src/components/**/*.spec.yaml"
    ]
  }
}
```

The component-governance schema is mapped by the governance-authoring skill,
not here.

---

## Usage

Ask Claude to generate a spec for a component:

```
Generate the spec for the Button component.
```

Or for a whole batch:

```
Generate spec files for all components in packages/ui/src/components/.
```

Claude will co-locate a `.spec.yaml` file next to each component source file,
validate it against the schema, and then ask whether to encode any governance
rules now (via governance-encode) or later.

### Example output

```yaml
# yaml-language-server: $schema=../../../../../.agent/skills/component-spec/schemas/component-spec.schema.json
component:
  name: Button
  category: atoms
  description: Triggers an action or navigates to a destination.
  type: interactive
  path: packages/ui/src/components/button/button.tsx

usage:
  useCases:
    - form submission
    - primary page action
    - navigation trigger
  requiredProps:
    - children
  commonPatterns:
    - name: primary-action
      description: The main call to action on a page or section.
      composition: |
        <Button variant="filled" size="md">
          Get started
        </Button>

accessibility:
  role: button
  keyboardSupport: Enter and Space activate the button.
  screenReader: Reads the button label. Use aria-label when the label is icon-only.
  focusManagement: standard
  wcag: AA

aiHints:
  priority: high
  keywords: [button, cta, action, submit, click, trigger]
  context: >
    Reach for Button whenever the user needs to trigger an action.
    Prefer filled variant for primary actions, outline for secondary.
    Never use Button for navigation — use a Link component instead.
```

---

## Schema

One schema ships with this skill.

### `component-spec.schema.json`

The descriptive spec. Required sections: `component`, `usage`, `aiHints`.

| Section | Required | Purpose |
|---|---|---|
| `component` | Yes | Name, category (atoms/molecules/organisms), functional type, source path |
| `component.figma` | No | Figma `fileKey`, `nodeId`, and `componentKey` — lets AI agents and Code Connect tools resolve the exact component in Figma |
| `usage` | Yes | Use cases, props, composition patterns |
| `aiHints` | Yes | Priority (high/medium/low), intent-matching keywords, natural language context |
| `composition` | No | Slots, nested sub-components, common partners, npm dependencies |
| `behavior` | No | States, event interactions, responsive rules |
| `accessibility` | No | ARIA role, keyboard support, screen reader, focus management, WCAG level |
| `variants` | No | Variant dimensions and their options |

**Functional types:** `interactive`, `display`, `container`, `input`, `navigation`

**Atomic Design categories:** `atoms`, `molecules`, `organisms`

> Looking for the component-governance schema? It moved to the
> governance-authoring skill:
> `.agent/skills/governance-authoring/schemas/component-governance.schema.json`.

#### Figma fields

| Field | Where to find it |
|---|---|
| `fileKey` | The segment after `/design/` in any Figma file URL: `figma.com/design/FILE_KEY/...` |
| `nodeId` | The `?node-id=` value in the URL when you select a component. Use `1:234` (colon) format for the REST API, `1-234` (hyphen) as it appears in the URL |
| `componentKey` | Stable across renames and file moves. Retrieve via the Figma REST API (`GET /v1/files/:key/components`) or from your Figma MCP tool |

```yaml
component:
  name: Button
  figma:
    fileKey: "abc123XYZ"
    nodeId: "1:234"
    componentKey: "a1b2c3d4e5f6a1b2c3d4e5f6"
```

---

## Customization

This skill is built for a specific project structure and Figma setup, but it is designed to be adapted. Here is what to change for your project.

### Adapt to your folder structure

The skill's `SKILL.md` contains a "Folder Structure" section that shows where spec files should live relative to component source files. Update that section to match your project:

**Default (this repo):**
```
packages/ui/src/components/[component-name]/
├── [component-name].tsx
└── [component-name].spec.yaml
```

**Alternative: flat components folder**
```
src/components/
├── Button.tsx
└── Button.spec.yaml
```

Edit `SKILL.md` → "Folder Structure (Required)" section to reflect your actual layout, and update the `component.path` description in `component-spec.schema.json` if needed.

### Update the schema reference path

Every `.spec.yaml` file starts with a `yaml-language-server` comment that points to its schema. The path is relative to the file's location. If the skill lives somewhere other than `.agent/skills/component-spec/`, count the directory levels and adjust accordingly.

**Skill at `.agent/skills/component-spec/` — component 5 levels deep:**
```yaml
# yaml-language-server: $schema=../../../../../.agent/skills/component-spec/schemas/component-spec.schema.json
```

Update the Quick Start template in `SKILL.md` so Claude generates the correct paths automatically.

### Adapt the component categories

The schema enforces `atoms`, `molecules`, `organisms` (Atomic Design). If your design system uses a different taxonomy — for example `primitives`, `patterns`, `layouts` — edit the `enum` in `component-spec.schema.json`:

```json
"category": {
  "type": "string",
  "enum": ["primitives", "patterns", "layouts"],
  "description": "Your system's component category taxonomy."
}
```

Then update `SKILL.md` → "Component Categories" section to match.

---

## CLI validation

Validate a file without an IDE:

```bash
# Requires js-yaml and ajv to be installed
npx js-yaml path/to/button.spec.yaml | npx ajv validate \
  -s .agent/skills/component-spec/schemas/component-spec.schema.json \
  -d /dev/stdin
```

---

## License

MIT — see [SKILL.md](./SKILL.md) for full attribution.
