---
name: react-frontend-conventions
description: "Constrains functional React with TypeScript: local state colocation, explicit props instead of spreads, Context for cross-cutting state, useMemo for heavy derivations, use-prefixed hooks, component tests. Applies to the intranet project (React + Vite + Ant Design). Do not use for class components or projects outside intranet."
---

# React frontend conventions

## Procedures

**When building or auditing React UI (intranet)**

1. Implement components as functions only; reject class components.
2. Author components as TypeScript in `.tsx` files.
3. Keep component state colocated nearest to consumers rather than centralized by default.
4. Pass props explicitly between parents and children; avoid spreading arbitrary prop bags into DOM or leaf components (`{...props}` on shared primitives is disallowed unless a documented exception exists).
5. Split components that exceed roughly 100 lines into smaller cohesive units rather than increasing file size indefinitely.
6. Apply Context API where multiple distinct subtrees consume the same cross-cutting UI state without prop drilling overload.
7. Use Ant Design components and patterns as the primary UI library; do not introduce additional CSS-in-JS or styling libraries without explicit approval.
8. Avoid excessive fragmentation into tiny wrapper components lacking clear abstraction value.
9. Wrap expensive derivations dependent on arrays or heavy objects with `useMemo` and precise dependency arrays.
10. Name custom hooks with the `use` prefix (`useAuth`, `useLocalStorage`, `useUrl`).
11. Before introducing a sizeable bespoke component, investigate existing Ant Design components and confirm whether a standard component covers the need.
12. Add automated tests for every component surfaced to users or shared across features.
13. Prefer composing existing primitives and feature modules before authoring net-new equivalents; escalate ambiguous duplication through design review paths.

## Error Handling

1. When Ant Design gaps block layout parity, check the component's API and `style` / `className` customization before layering ad hoc inline styles.
2. When prop drilling depth suggests Context misuse, refactor boundaries so providers wrap coherent subtrees rather than the entire router tree without reason.
