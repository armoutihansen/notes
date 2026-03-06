---
layer: 03_software_engineering
type: engineering
tool: typescript
status: evergreen
tags: [workflow]
created: 2026-03-02
---

# TypeScript Local Development

## Purpose

Reference for TypeScript tooling: compiler configuration, IDE integration, escaping the type checker when necessary, and typing third-party JavaScript.

## Implementation Notes

### `tsconfig.json` — Key Options

```json
{
  "compilerOptions": {
    "lib":                  ["esnext", "dom", "dom.iterable"],
    "target":               "esnext",
    "strict":               true,
    "skipLibCheck":         true,
    "verbatimModuleSyntax": true,
    "esModuleInterop":      true,
    "moduleDetection":      "force",
    "noUncheckedIndexedAccess": true,
    "outDir":               "./dist",
    "rootDir":              "./src"
  },
  "include": ["src/**/*"],
  "exclude": ["node_modules", "dist"]
}
```

| Option | Effect |
|---|---|
| `lib` | Which runtime APIs are available. Add `"dom"` for browser globals. |
| `target` | ECMAScript version of the emitted JavaScript. Pin to a specific version (e.g. `"es2024"`) before shipping. |
| `strict` | Enables all strict checks (`strictNullChecks`, `noImplicitAny`, etc.). **Always on for new projects.** |
| `skipLibCheck` | Skips type-checking of `.d.ts` files in `node_modules` — dramatically speeds up compilation. |
| `verbatimModuleSyntax` | Forces `import type` for type-only imports; simplifies module boundary handling. |
| `esModuleInterop` | Allows `import x from 'commonjs-module'` syntax when interoperating with CommonJS. |
| `moduleDetection` | Set to `"force"` to treat every file as a module. Correct for any new project. |
| `noUncheckedIndexedAccess` | Index access (`arr[i]`, `obj[key]`) returns `T \| undefined` — prevents off-by-one runtime crashes. |
| `outDir` / `rootDir` | Output directory and source root for compiled JS. |
| `include` / `exclude` | Glob patterns controlling which files are compiled. |

### TypeScript Language Server and IDE Integration

The TypeScript Language Server (tsserver) powers editor features in VS Code and most other editors that support the Language Server Protocol:

- Type inference and hover documentation
- Inline error highlighting
- Autocompletion and signature help
- Refactoring (rename, extract, organise imports)

The language server reads `tsconfig.json` automatically from your project root. No additional configuration is needed for VS Code — it ships with a bundled TypeScript version, which you can override per-workspace to match the project's installed version.

### `@ts-ignore` / `// @ts-nocheck` — Use Very Sparingly

Suppress errors on the next line:

```typescript
// @ts-ignore
const x: number = "not a number"; // error suppressed
```

Disable type checking for an entire file:

```typescript
// @ts-nocheck

const x: number = "not a number";    // no error
```

Both are escape hatches that defeat the type system. Use only:
- When integrating a genuinely untyped third-party script with no available types.
- During a large JS → TS migration where a specific file cannot be addressed yet.

Prefer `@ts-ignore` (single line) over `@ts-nocheck` (whole file). Add a comment explaining *why* the suppression is necessary.

### Declaration Files (`.d.ts`) for Typing JS Libraries

`.d.ts` files contain type information only — no runtime code. TypeScript uses them to provide type hints for JavaScript that exists outside the compiler's control.

**Global augmentation example** — typing a JS library loaded via `<script>` tag:

```typescript
// globals.d.ts
declare global {
  interface Window {
    google: GoogleAPI;
  }
}

interface GoogleAPI {
  accounts: {
    id: {
      initialize: (config: { client_id: string; callback: (r: any) => void }) => void;
      renderButton: (el: HTMLElement, opts: { theme?: string; size?: string }) => void;
      prompt: () => void;
    };
  };
}

export {}; // ensure this is treated as a module
```

**Module declaration** — typing an untyped npm package:

```typescript
// pregnantgoku.d.ts
declare module "pregnantgoku" {
  export function kamahameha(target: [number, number]): void;
  export type Saiyan = { name: string; powerLevel: number };
}
```

### `@types/` Packages — Community Type Definitions

[DefinitelyTyped](https://github.com/DefinitelyTyped/DefinitelyTyped) publishes community-maintained type declarations for popular JS libraries as `@types/<package>`:

```bash
npm install --save-dev @types/lodash
npm install --save-dev @types/node
```

TypeScript resolves these automatically. Check DefinitelyTyped before writing your own declarations.

**Priority for untyped libraries:**
1. Check for `@types/<lib>` on npm.
2. If unavailable, write a `.d.ts` module declaration.
3. As a last resort, add `"allowJs": true` to `tsconfig.json` and let TypeScript infer types from JS source.

### Vanilla Vite + TypeScript Setup

[Vite](https://vitejs.dev/) is the recommended build tool for new TypeScript projects — fast (esbuild for development, Rollup for production), simple config, and built-in HMR:

```bash
npm create vite@latest my-app -- --template vanilla-ts
cd my-app
npm install
npm run dev
```

Vite scaffolds a minimal `tsconfig.json` and a `vite.config.ts`. Key points:
- Vite transpiles TypeScript (strips types) but does **not** type-check during dev — run `tsc --noEmit` separately for type checking.
- Use the `vite-plugin-checker` plugin or a pre-commit hook with `tsc` for CI type checking.

## Trade-offs

- Enable `strict` on all new projects. Disabling it during a JS → TS migration is acceptable, but re-enable flag by flag as the codebase matures.
- `skipLibCheck: true` trades correctness for speed — usually the right call; downstream declaration errors are rarely actionable.
- Handwritten `.d.ts` files are brittle — they diverge from the actual library as it updates. Prefer `@types/` packages where available.

## References

- [TypeScript tsconfig reference](https://www.typescriptlang.org/tsconfig/)
- [DefinitelyTyped](https://github.com/DefinitelyTyped/DefinitelyTyped)
- [Vite documentation](https://vitejs.dev/)

## Links

- [[ts_type_system]]
- [[ts_objects_and_interfaces]]
