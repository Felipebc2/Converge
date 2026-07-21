# Converge — Design System

Status: normative  
Version: 1.0  
Source stage: 6 — Design System  
Visual direction: Graphite Signal  
Accessibility requirement: WCAG 2.2 AA

## 1. Purpose

This document defines Converge's visual and behavioral rules. It is the normative source for themes, tokens, typography, layout, components, states, motion, accessibility, and data visualization.

The product MUST feel precise, sophisticated, and operational. It MUST avoid both a generic SaaS dashboard and a raw terminal appearance. Complexity is presented through progressive disclosure, with context and explicit control.

## 2. Principles

1. Control with context: state, source, impact, and action MUST remain close together.
2. Dominant neutrals: color signals function; it does not fill the interface without purpose.
3. Hierarchy before decoration: spacing, typography, and tonal layers take precedence over borders and shadows.
4. Moderate density: fast scanning without a compact mode in the current scope.
5. Parity across themes and providers: no function or information depends on the theme or provider.
6. Verifiable accessibility: keyboard operation, focus, contrast, semantics, and reduced motion are acceptance criteria.

## 3. Token Architecture

Use three levels:

```text
primitives → semantic → component
```

- `primitives`: values with no interface meaning.
- `semantic`: stable roles across themes, such as surface, text, interaction, and state.
- `component`: exclusive aliases when a component requires its own contract.

Components MUST NOT consume primitives directly. Tokens MUST be exposed as CSS Custom Properties and, when useful, as typed TypeScript objects.

### 3.1 Naming Convention

```text
--cv-{category}-{role}-{state?}
```

Examples:

```css
--cv-color-surface-1
--cv-color-text-primary
--cv-color-interaction-hover
--cv-button-primary-background
--cv-space-4
--cv-radius-md
```

## 4. Color

### 4.1 Graphite Signal Contract

- Graphite is the structural foundation.
- Periwinkle or blue-gray is the color for interaction, focus, links, and navigation.
- Signal red is a controlled brand signature used in the logo, editorial highlights, reinforced selection, and a small number of primary actions.
- Danger uses an independent semantic red scale.
- Provider, state, and data series MUST NOT reuse brand tokens.
- No state is communicated by color alone.

### 4.2 Reference Primitives

The values below are the implementable baseline. The token pipeline MUST generate the sRGB conversion. Adjustments are allowed only to correct gamut or contrast while preserving hue and semantic role.

#### Neutrals

| Token | OKLCH | Expected use |
|---|---:|---|
| `neutral-0` | `oklch(100% 0 0)` | white |
| `neutral-50` | `oklch(98% 0.004 260)` | Light background |
| `neutral-100` | `oklch(95% 0.006 260)` | Light surface |
| `neutral-200` | `oklch(90% 0.009 260)` | Light subtle border |
| `neutral-400` | `oklch(67% 0.015 260)` | disabled text |
| `neutral-600` | `oklch(46% 0.018 260)` | Light secondary text |
| `neutral-800` | `oklch(27% 0.018 260)` | elevated Dark surface |
| `neutral-900` | `oklch(20% 0.016 260)` | Dark surface |
| `neutral-950` | `oklch(14% 0.014 260)` | Dark background |

#### Brand and Interaction

| Token | OKLCH | Role |
|---|---:|---|
| `signal-500` | `oklch(61% 0.22 25)` | brand signature |
| `signal-600` | `oklch(54% 0.21 25)` | hover on light background |
| `signal-300` | `oklch(76% 0.13 25)` | detail on dark background |
| `periwinkle-500` | `oklch(63% 0.12 270)` | Light interaction |
| `periwinkle-400` | `oklch(72% 0.10 270)` | Dark interaction |
| `periwinkle-200` | `oklch(88% 0.05 270)` | Light tonal selection |
| `periwinkle-900` | `oklch(29% 0.06 270)` | Dark tonal selection |

#### Semantic States

| State | Reference color | Rule |
|---|---:|---|
| Success / Running | `oklch(66% 0.16 150)` | green, with icon and label |
| Warning | `oklch(76% 0.15 80)` | amber, with explicit impact |
| Danger / Error | `oklch(58% 0.21 20)` | distinct from `signal-*` by token and context |
| Info | `oklch(66% 0.13 245)` | informational blue |

### 4.3 Semantic Tokens by Theme

| Role | Light | Dark |
|---|---|---|
| `background` | `neutral-50` | `neutral-950` |
| `surface-1` | `neutral-0` | `neutral-900` |
| `surface-2` | `neutral-100` | `neutral-800` |
| `surface-elevated` | `neutral-0` | `oklch(24% 0.018 260)` |
| `text-primary` | `neutral-950` | `neutral-50` |
| `text-secondary` | `neutral-600` | `oklch(75% 0.012 260)` |
| `text-disabled` | `neutral-400` | `oklch(57% 0.012 260)` |
| `border-subtle` | `neutral-200` | `oklch(31% 0.014 260)` |
| `border-strong` | `oklch(72% 0.012 260)` | `oklch(46% 0.015 260)` |
| `interaction` | `periwinkle-500` | `periwinkle-400` |
| `interaction-hover` | `oklch(56% 0.14 270)` | `oklch(79% 0.09 270)` |
| `selection-bg` | `periwinkle-200` | `periwinkle-900` |
| `focus-ring` | `periwinkle-500` | `periwinkle-400` |
| `brand-signal` | `signal-600` | `signal-300` |

Every foreground and background combination MUST pass automated testing. Minimums: 4.5:1 for normal text and 3:1 for large text, essential icons, control borders, and focus indicators.

## 5. Typography

### 5.1 Families

| Role | Family | Application |
|---|---|---|
| Display | Playfair Display | page titles and selected prominent numbers |
| Interface | Annapurna SIL | navigation, controls, labels, and short text |
| Reading | Crimson Text | Markdown and extended reading |
| Technical | configurable monospace | read-only terminal, code, paths, and aligned data |

Fallbacks MUST preserve the typographic category. Fonts MUST NOT block the first interface render.

### 5.2 Scale

| Token | Size | Line height | Use |
|---|---:|---:|---|
| `text-xs` | 12 px | 1.35 | supporting metadata |
| `text-sm` | 14 px | 1.4 | dense data, tables, labels |
| `text-md` | 16 px | 1.5 | standard body |
| `text-lg` | 18 px | 1.4 | section title |
| `text-xl` | 20 px | 1.35 | panel title |
| `text-2xl` | 24 px | 1.3 | compact page title |
| `text-3xl` | 32 px | 1.2 | restricted editorial emphasis |

Controls use an approximate line height of 1.35; body text, 1.5; extended reading, 1.6–1.7. Labels, buttons, tables, and operational data MUST NOT use Playfair Display.

## 6. Spacing, Geometry, and Elevation

### 6.1 Spacing

4 px base scale:

```text
0, 4, 8, 12, 16, 24, 32, 40, 48, 64
```

4 px increments support compact internal adjustments. Layout and major separation prioritize multiples of 8 px.

### 6.2 Radii

| Token | Value | Use |
|---|---:|---|
| `radius-sm` | 6 px | small controls |
| `radius-md` | 8 px | fields and buttons |
| `radius-lg` | 10 px | panels |
| `radius-xl` | 12 px | elevated surfaces |
| `radius-pill` | 999 px | tags, states, and compact actions only |

### 6.3 Borders and Shadows

- Default border: 1 px with `border-subtle`.
- Emphasized border: 1 px with `border-strong`.
- Shadows appear only on drawers, popovers, menus, tooltips, and elevated modals.
- Nested cards and broad decorative shadows are prohibited.

## 7. Layout and Responsiveness

### 7.1 Global Shell

- Persistent global header with logo, navigation, active project, notifications, and settings.
- Primary navigation: `Overview`, `Analytics`, `Agents`, `Context Files`.
- `Agent Loop` belongs to the selected Session.
- Do not display a user avatar in the local scope without authentication.

### 7.2 Reference Breakpoints

| Range | Composition |
|---|---|
| `≥ 1280 px` | wide desktop |
| `1024–1279 px` | narrow desktop |
| `768–1023 px` | tablet |
| `< 768 px` | mobile |

Reflow MUST also consider usable width, not only the viewport.

### 7.3 Session Workspace

- Wide desktop: 240–320 px File Tree; flexible main area with a 480 px minimum; 320–400 px auxiliary area.
- Narrow desktop: collapsible File Tree; auxiliary area becomes a drawer.
- Tablet: one main surface at a time; tree and details become overlays.
- Mobile: prioritizes consultation and monitoring; critical actions remain accessible and confirmed.
- Only one main surface and, at most, one auxiliary surface may remain open simultaneously.
- Resizable panels have bounds, keyboard operation, and per-project persistence of the last valid width.

## 8. Components

Every interactive component specifies `default`, `hover`, `focus-visible`, `active`, `selected`, `disabled`, `loading`, and, when applicable, `invalid` or `error`.

### 8.1 Buttons

Variants: `primary`, `secondary`, `ghost`, `danger`, `icon`.

- Standard height: 36 px; compact: 32 px.
- Touch target: at least 44 × 44 CSS px.
- `primary` uses periwinkle interaction by default; Signal red is reserved for genuinely selected, non-destructive brand actions.
- `danger` uses the semantic token and confirmation proportional to impact.
- Loading preserves width, prevents repetition, and keeps the label understandable.

### 8.2 Fields

- Standard height: 36 px.
- Persistent label; placeholder does not replace the label.
- Help and error messages remain close to their source.
- Invalid state includes text and an icon in addition to color.

### 8.3 Tabs

- Selection uses an underline and tonal contrast.
- When implemented as a `tablist`, arrow keys navigate, `Home`/`End` are supported when appropriate, and focus remains independent from selection.

### 8.4 Statuses and Tags

- Status combines an icon or marker, a label, and semantic color.
- `Running`, `Warning`, `Error`, `Stale`, `Offline`, `Syncing`, and `Needs Attention` have stable text.
- Pills do not replace explanatory content when there is an impact or required action.

### 8.5 Overlays

- Inline or contextual panel: content connected to the task.
- Popover: small, transient action.
- Drawer: history or extensive auxiliary content.
- Modal: permissions, destructive confirmation, or critical decision.
- Drawers and modals trap focus, support `Escape` when safe, and return focus to the trigger.

### 8.6 Terminal and File Tree

- The terminal is read-only. It does not present a prompt that suggests free-form typing.
- Allowed actions such as `Start`, `Stop`, and `Force Kill` remain outside the text surface and follow their confirmation contracts.
- The File Tree is collapsible and resizable; at smaller widths, it becomes an overlay above the content.
- Editing in the current scope is limited to compatible files inside an existing `.specify/`; other files are shown in read-only mode.

## 9. Feedback and Content States

- Initial loading: structural skeleton matching the final layout.
- Refresh: preserves previous data, indicates updating, and shows a timestamp.
- Empty state: explains the reason, expected content, and next valid action.
- Local error: close to its source, with a recovery action.
- Banner: global or persistent issue.
- Toast: transient confirmation or non-blocking failure.
- Operational states display source, time, impact, and available action.
- Notifications MUST NOT store secrets or sensitive payloads.

File conflicts block silent overwrites and offer `Compare`, `Reload`, and confirmed `Overwrite`.

## 10. Motion

| Use | Duration |
|---|---:|
| recurring state | 150–200 ms |
| drawer or structural change | 200–250 ms |

- Standard curve: ease-out-quart.
- Prefer `transform` and `opacity`; avoid animating layout properties.
- Motion only communicates state, progress, opening, closing, or spatial relationships.
- Do not use decorative loading choreography.
- With `prefers-reduced-motion`, remove displacement, parallax, and sequences; use a short crossfade or instant change.

## 11. Accessibility

- WCAG 2.2 AA in Light and Dark themes.
- Complete keyboard interface.
- Visible focus: 2 px, 2 px offset, and minimum 3:1 contrast.
- Focus order follows visual and semantic order.
- Ambiguous icons have accessible labels; a tooltip is never the only source of information.
- Tables, trees, tabs, menus, terminal, and resizers use appropriate semantics.
- Relevant asynchronous changes use live-region announcements without excess.
- 200% zoom and reflow at 320 CSS px MUST NOT cause loss of essential content.

## 12. Iconography

- Official library: Phosphor Icons.
- Base sizes: 16, 18, 20, and 24 px.
- Consistent weight by context.
- An icon does not replace text when the action or state may be ambiguous.
- Provider logos MUST NOT act as the only identifier; use name, provider, Session, and state.

## 13. Data Visualization

- A chart communicates trends and relationships; a table communicates exact values and supports inspection.
- The categorical palette is independent of brand, providers, and states.
- Color is reinforced by a label, shape, line pattern, or marker.
- An axis starts at zero when magnitude comparison requires it; truncation is explicit.
- Tooltips are keyboard-accessible.
- Every relevant chart has a text summary and a tabular alternative when needed.
- Locale, unit, and precision are consistent.
- Costs are labeled as estimates when the source cannot guarantee accuracy.

## 14. Themes

- The default follows the system.
- The user can explicitly select Light or Dark.
- The preference is persisted.
- Themes have functional and informational parity.
- Changing the theme MUST NOT cause a prolonged flash or reset context.

## 15. Acceptance Criteria

An implementation is compliant only when it:

- [ ] uses semantic tokens, with no direct primitives in components;
- [ ] passes automated contrast audits in both themes;
- [ ] preserves keyboard navigation and essential actions;
- [ ] displays consistent visible focus;
- [ ] does not communicate state by color alone;
- [ ] respects `prefers-reduced-motion`;
- [ ] preserves the read-only terminal;
- [ ] limits editing to the approved safe scope;
- [ ] applies structural reflow at the defined breakpoints;
- [ ] keeps provider, brand, and states semantically separate;
- [ ] covers interactive and content states;
- [ ] does not introduce nested cards, decorative shadows, default modals, or disabled future controls.

## 16. Governance

- This file takes precedence over exploratory mockups when they conflict.
- Normative changes require a decision record and an update to this document.
- Color values may receive technical gamut or contrast corrections without changing the Graphite Signal direction.
- Stage 7 screen specifications MUST reference these tokens and contracts without redefining them locally.
- Future functionality remains hidden until it has a usable implementation.

## 17. Out of Scope for This Document

- Detailed content and flow for each of the eight screens.
- Production code or React components.
- Complete Spec Kit, workflow editor, collaboration, split layout, and future plugins.
- HTML, CSS, or JavaScript code from exploratory prototypes.
