---
name: sass-js-ts-html
description: Guidance for writing and editing SASS/SCSS, JavaScript, TypeScript, and HTML in front-end projects. Use when editing .scss, .sass, .js, .ts, .tsx, .html files or when the user asks about styling, scripts, or markup.
---

# SASS, JavaScript, TypeScript & HTML

Concise guidance for working with this stack. Apply when editing or discussing these technologies.

## SASS / SCSS

### Module system
- Prefer `@use 'path' as ns` over `@import` for variables, mixins, and functions. `@import` is deprecated.
- Use `@use 'sass:color'`, `@use 'sass:math'`, etc. for built-in modules.
- One `@use` per file; use namespaces to avoid clashes (e.g. `ns.$variable`, `@include ns.mixin()`).

### Nesting and structure
- Nest only 2–3 levels deep; flatten when selectors get long.
- Use `&` for modifiers and pseudo-classes (e.g. `&:hover`, `&.is-active`).
- Keep block names clear and consistent (e.g. BEM-like: `.block__element--modifier` or `.block-element-modifier`).

### Variables and mixins
- Define project variables in a single place (e.g. `_variables.scss`) and `@use` where needed.
- Use mixins for repeated patterns; use placeholders (`%name`) when the output is only extended, not mixed in.
- Prefer `@include` over `@extend` across unrelated selectors to avoid unexpected selector growth.

### General
- Use SCSS syntax (with `{}` and `;`) unless the project explicitly uses indented `.sass`.
- Comment with `//` for line comments; reserve `/* */` for CSS output that should appear in the built file.

---

## JavaScript

### Style and compatibility
- Prefer `const`; use `let` only when reassignment is needed. Avoid `var`.
- Use arrow functions for short callbacks; use regular `function` when `this` or named recursion is needed.
- Prefer template literals for strings that include variables or line breaks.
- Use optional chaining (`?.`) and nullish coalescing (`??`) where they improve readability.

### Structure
- Prefer `async/await` over raw Promises for sequential async logic.
- Use destructuring for objects and arrays when it clarifies intent.
- Prefer `for...of` for iterating arrays; avoid `for...in` for arrays.

### Modules
- Use ES modules (`import` / `export`) when the environment supports them.
- Prefer named exports for libraries and re-exports; default export for a single main export (e.g. a component or app entry).

---

## TypeScript

### Types
- Prefer `interface` for object shapes; use `type` for unions, intersections, and mapped types.
- Avoid `any`; use `unknown` and narrow with type guards when the type is not known.
- Use strict options (`strict`, `noImplicitAny`) when possible.

### Practices
- Type function parameters and return types for public APIs and non-obvious cases; rely on inference for simple locals.
- Use `readonly` for arrays and object properties that should not be mutated.
- Prefer `const` assertions and literal types where they improve correctness (`as const`, string literal unions).

### React / TSX (when applicable)
- Type component props with an interface (e.g. `interface Props { ... }`).
- Use `React.FC<Props>` only if the project already does; otherwise `function Comp(props: Props)`.
- Type event handlers as `React.ChangeEvent<HTMLInputElement>`, etc., when needed.

---

## HTML

### Structure and semantics
- Use semantic elements: `header`, `nav`, `main`, `article`, `section`, `aside`, `footer`.
- Use heading levels in order (`h1` → `h2` → `h3`); don’t skip levels for styling (use CSS instead).
- Use `<button>` for actions, `<a>` for navigation; avoid `div`/`span` with click handlers when a button or link is appropriate.

### Accessibility
- Prefer visible labels; associate with `for`/`id` or wrap the control in `<label>`.
- Add `alt` to images (empty `alt=""` for decorative images).
- Use `aria-*` only when HTML semantics are insufficient (e.g. custom widgets, live regions).

### General
- Use lowercase for tags and attributes; double quotes for attribute values.
- Prefer a single root or a clear layout root; keep structure flat enough to be readable.
- Use `data-*` attributes for custom data needed by JS; avoid non-standard attributes for behavior.

---

## Cross-cutting

- **Naming**: Use consistent naming (camelCase in JS/TS, kebab-case or BEM in CSS/SCSS and often in HTML class names).
- **Files**: Co-locate when it helps (e.g. component folder with `.tsx` + `.scss`); keep shared styles and types in a clear place.
- **Formatting**: Match existing project style (indent, quotes, semicolons); use the project’s formatter/linter.
