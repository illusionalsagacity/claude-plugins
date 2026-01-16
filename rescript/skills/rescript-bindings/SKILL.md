---
name: rescript-bindings
description: "Creates ReScript bindings for JavaScript/TypeScript libraries. Use when converting TypeScript definitions to ReScript, writing external function bindings, or binding to npm packages. Handles module imports, class bindings, type mappings (variants, records, generics), callbacks, and function overloads."
---

# ReScript Bindings

Create type-safe ReScript bindings for JavaScript/TypeScript libraries.

## Before You Start

**Ask these questions first:**

1. **ReScript version?** (v11+, v10.x, or older)
   - v11+: `@unboxed` variants, `@tag` discriminators, optional record fields
   - v10.1+: Optional record fields (`{name?: string}`)
   - Older: May need `@obj` for optional fields

2. **Zero-cost or ergonomic?**
   - **Zero-cost**: Generated JS is erased, no runtime overhead, uses `@unboxed`
   - **Ergonomic**: Idiomatic ReScript, may add thin wrappers for better DX

3. **TypeScript version?** When looking up type definitions, ensure you use the **exact version** the user expects. If unclear, ask.

## Check for Existing Bindings

Before creating bindings for a library or its dependencies:
1. **Search the codebase** for existing bindings (look for `@module("{library}")` patterns)
2. **Ask the user** if they already have bindings for dependencies you would otherwise create
3. Check for published binding packages (e.g., `@rescript/react`, `rescript-webapi`)

Avoid duplicating or conflicting with existing bindings.

## Critical: Verify Your Bindings

**Compiling ReScript does NOT ensure bindings are correct!** You must:
1. Write example ReScript code using the bindings
2. Check the generated JavaScript output matches expected behavior
3. Verify runtime behavior if possible

## Binding Reference

Choose the pattern that matches your use case:

| Pattern | When to Use | Reference |
|---------|-------------|-----------|
| Module imports | `import {x} from "pkg"` | [functions.md](functions.md) |
| Classes | JS classes with methods | [classes.md](classes.md) |
| Globals | `window.foo`, `Math.random` | [globals.md](globals.md) |
| Types | Records, variants, unions | [types.md](types.md) |
| Callbacks | Function parameters | [callbacks.md](callbacks.md) |
| Overloads | Multiple signatures | [overloads.md](overloads.md) |

## Quick Reference

### Essential Attributes

| Attribute | Purpose |
|-----------|---------|
| `@module("pkg")` | Import from npm package |
| `@val` | Bind global value |
| `@send` | Instance method (instance is first arg) |
| `@get` / `@set` | Property access |
| `@new` | Constructor |
| `@as("jsName")` | Rename for JS |
| `@variadic` | Rest/spread args |
| `@return(nullable)` | null or undefined → `option` |
| `@unboxed` | Zero-cost variant wrapper |

### Naming Conventions

- **Functions that may throw**: Suffix with `OrThrow` (e.g., `parseOrThrow`, `getExn`)
- **Overloaded functions**: Use descriptive suffixes (`fetchWithOptions`, not `fetch2`)

### Type Fidelity

If a TypeScript type cannot be fully represented in ReScript (mapped types, conditional types, complex generics), **inform the user** about the limitation and what approximation was made.

### Documentation Comments

Use `/** */` syntax (NOT JSDoc):

```rescript
/** Fetches data from the API. */
@module("api") external fetch: string => promise<response> = "fetch"
```