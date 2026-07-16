---
name: nodejs-typescript-conventions
description: Enforces TypeScript-only source, pnpm for scripts and deps, ESM import and export defaults, async await over raw then chains, no any types, minimal let usage, private or readonly class fields, functional collection helpers. Do not use for plain JavaScript codebases.
---

# Node.js, JavaScript, and TypeScript conventions

## Procedures

**When writing or modifying application TypeScript**

1. Author runtime and shared source strictly in TypeScript.
2. Use `pnpm` exclusively for installs and script execution (`pnpm install`, `pnpm run build`). Do not introduce alternate package managers (npm, yarn, bun) for those roles.
3. When libraries omit types, prefer `@types/<package>` where published on DefinitelyTyped; rely on bundled types when shipped (Vitest includes types).
4. Before finishing edits, validate type correctness via the project toolchain (`pnpm run type-check` / `tsc` / editor diagnostics as applicable).
5. Prefer `const` bindings; reserve `let` for genuine reassignment and never declare with `var`.
6. Declare class fields `private`, `readonly`, or both unless a deliberate public surface is mandated by the dependency graph.
7. Prefer immutable chain helpers (`filter`, `map`, `reduce`, `find`) over manual `for` / `while` loops for collection transforms.
8. Prefer arrow functions for short pure operations and lexical `this` where stylistically consistent with surrounding modules.
9. Sequence asynchronous operations with `async` / `await` instead of chaining bare `.then` trees for primary control flow.
10. Ban `any`; supply concrete interfaces or type aliases covering inputs and outputs.
11. Replace CommonJS imports with ESM syntax (`import` / `export`); disallow `require` and `module.exports` in TypeScript modules.
12. Export a single default when the module exposes exactly one principal symbol; otherwise use named exports with consistent barrel ergonomics.

## Error Handling

1. When `async` awaits risk unhandled rejection, propagate errors to callers or map them to domain-safe results following service-layer norms.
2. When dependency scripts assume another package manager, align documentation and CI to pnpm-first flows before branching tooling.
