---
name: rescript-bindings
description: "Creates ReScript bindings for JavaScript/TypeScript libraries. Use when converting TypeScript definitions to ReScript, writing external function bindings, or binding to npm packages. Handles module imports, class bindings, type mappings (variants, records, generics), callbacks, and function overloads."
---

# ReScript Bindings

Create type-safe ReScript bindings for JavaScript/TypeScript libraries.

## Before You Start

**Ask these questions first:**

1. **What's the task?**
   - **Create**: New bindings from scratch
   - **Verify**: Check existing bindings against TypeScript API
   - **Update**: Update bindings for new library version
   - **Extend**: Add new bindings to existing set

2. **ReScript version?** (v11+, v10.x, or older)

   - v11+: `@unboxed` variants, `@tag` discriminators, optional record fields
   - v10.1+: Optional record fields (`{name?: string}`)
   - Older: May need `@obj` for optional fields

3. **Zero-cost or ergonomic?**

   - **Zero-cost**: Generated JS is erased, no runtime overhead, uses `@unboxed`
   - **Ergonomic**: Idiomatic ReScript, may add thin wrappers for better DX

4. **TypeScript version?** When looking up type definitions, ensure you use the **exact version** the user expects. If unclear, ask.

## TypeScript as Source of Truth

**CRITICAL**: Always use the actual TypeScript `.d.ts` files as the canonical reference for what bindings should look like.

**DO use:**
- `node_modules/{library}/*.d.ts` files
- `node_modules/@types/{library}/*.d.ts` files
- The library's TypeScript source if available

**DO NOT use as primary reference:**
- Migration guides (those are for consumers, not binding authors)
- Blog posts or tutorials
- Documentation summaries (may be outdated or incomplete)
- API documentation websites (use as supplementary only)

When verifying or updating bindings, read the TypeScript interfaces prop by prop and compare systematically against the ReScript bindings.

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

| Pattern        | When to Use                 | Reference                    |
| -------------- | --------------------------- | ---------------------------- |
| Module imports | `import {x} from "pkg"`     | [functions.md](functions.md) |
| Classes        | JS classes with methods     | [classes.md](classes.md)     |
| Globals        | `window.foo`, `Math.random` | [globals.md](globals.md)     |
| Types          | Records, variants, unions   | [types.md](types.md)         |
| Callbacks      | Function parameters         | [callbacks.md](callbacks.md) |
| Overloads      | Multiple signatures         | [overloads.md](overloads.md) |

## Quick Reference

### Essential Attributes

| Attribute           | Purpose                                 |
| ------------------- | --------------------------------------- |
| `@module("pkg")`    | Import from npm package                 |
| `@val`              | Bind global value                       |
| `@send`             | Instance method (instance is first arg) |
| `@get` / `@set`     | Property access                         |
| `@new`              | Constructor                             |
| `@as("jsName")`     | Rename for JS                           |
| `@variadic`         | Rest/spread args                        |
| `@return(nullable)` | null or undefined → `option`            |
| `@unboxed`          | Zero-cost variant wrapper               |

### Naming Conventions

- **Functions that may throw**: Suffix with `OrThrow` (e.g., `parseOrThrow`, `getExn`)
- **Overloaded functions**: Use descriptive suffixes (`fetchWithOptions`, not `fetch2`)
- **Unsafe bindings**: Suffix with `Unsafe` when:

  - The return type is a type variable, especially when inputs are also type variables with no enforceable relationship (e.g., `useQueryStatesUnsafe`)
  - The return type omits nullability that could exist at runtime (e.g., `Array.getUnsafe` returns `'a` instead of `option<'a>`)

  Follows ReScript convention: `Option.getUnsafe`, `Array.getUnsafe`, etc.

### Type Fidelity

If a TypeScript type cannot be fully represented in ReScript (mapped types, conditional types, complex generics):

1. **Suffix the binding with `Unsafe`** when the return type is a type variable (and especially when inputs are too)
2. **Inform the user** about the limitation and what approximation was made
3. **Document the override pattern** - users can create their own binding with concrete types:

```rescript
// Library provides generic/unsafe version:
@module("nuqs")
external useQueryStatesUnsafe: 'parsers => ('values, setQueryStates<'values>) = "useQueryStates"

// User creates type-safe version with concrete types in their application:
type myParsers = {name: singleParserBuilderWithDefault<string>, count: singleParserBuilder<int>}
type myValues = {name: string, count: Nullable.t<int>}  // name has default, count doesn't

@module("nuqs")
external useQueryStates: myParsers => (myValues, setQueryStates<myValues>) = "useQueryStates"
```

This pattern gives library authors safe defaults while letting application developers opt into precise types when they control both sides.

### Consider the Call Site

TypeScript types tell you what a value *is*, but the right ReScript type depends on how it will be *used*. When a TypeScript type is overly dynamic (mapped types, `Record<string, T>`, index signatures), ask: what does the consumer actually do with this value?

For example, if a library returns `Record<string, (event: Event) => void>` but those handlers will be spread onto a React element, `Dict.t<JsxEvent.Synthetic.t => unit>` is the faithful translation — but a record with optional fields for the known event handlers is more ergonomic and avoids needing `%identity` casts at the call site.

**Guideline:** Prefer ergonomic types that work well at the call site over 1:1 TypeScript mappings, especially when the TypeScript type is lazily typed. A `Record<string, T>` in TypeScript often means "we didn't enumerate the keys" rather than "the keys are truly arbitrary."

### Documentation Comments

Use `/** */` syntax (NOT JSDoc):

```rescript
/** Fetches data from the API. */
@module("api") external fetch: string => promise<response> = "fetch"
```
