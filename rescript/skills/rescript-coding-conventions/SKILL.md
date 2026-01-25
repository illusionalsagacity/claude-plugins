---
name: rescript-coding-conventions
description: General ReScript coding conventions, style, and type design instructions. Use when writing ReScript code.
---

# ReScript

## General Guidance

Remember that ReScript is a functional programming language. Prefer pure functions, immutability, and pattern matching.

Most UI states can and should be modelled as finite state machines using reducers and variant types. Combine multiple `bool` flags into a single variant that models the entire state, rather than allowing for invalid combinations.

**Incorrect**:

```rescript
type t<'data> = {
  isOpen: bool,
  isLoading: bool,
  data: 'data,
}
```

**Correct**:

```rescript
type t<'data> =
  | Closed
  | Loading
  | Loaded('data)
```

Model business entities, validated / unvalidated data so that the compiler enforces the requirements and prevents bugs. In other words, use the compiler to enforce the business rules.

For example, to create a 'validated' data type, create a `make` function to parse the unvalidated type:

```rescript
module SocialSecurityNumber = {
  type t
  let fromString: string => result<t, [#InvalidFormat]>
}
```

The error type should be suited to the constraints on the entity.

For types that alias primatives, but should not be directly comparable to other primatives not of the same type, make the type opaque in the `.resi` interface.

**TaskId.res**:

```rescript
type t = int
let eq = (a, b) => a == b
```

**TaskId.resi**:

```rescript
type t
let eq: (t, t) => bool
```

Start with records since they are already nominally typed, unless the user explicitly asks for encapsulation, getters, setters, or fluent builder functions, etc.

The intent is not to produce a complex architecture, but a simple one with clear separation of concerns, ease of testing and maintenance.

## Module Organization

### Submodules for Related Types

When a type is closely tied to another type and only used in that context, define it as a submodule:

```rescript
// Task.res
module Priority = {
  @unboxed
  type t = | @as("low") Low | @as("medium") Medium | @as("high") High
}

type t = {
  id: TaskId.t,
  title: string,
  priority: Priority.t,
  // ...
}
```

The `@unboxed` with `@as` decorators means the variant compiles directly to the string value, eliminating the need for `toString`/`fromString` functions for JSON encoding. This keeps related code cohesive and the namespace clean.

### Comparison Functions at the Right Abstraction Level

If an ID type has an equality function, the entity type should also have one:

```rescript
// Column.resi
type t
let id: t => ColumnId.t
let eq: (t, t) => bool  // Compare by ID internally
```

```rescript
// Column.res
let eq = (a, b) => ColumnId.equal(id(a), id(b))
```

This allows callers to work at the right abstraction level:

```rescript
// Prefer this:
columns->Array.filter(col => !Column.eq(col, targetColumn))

// Over this:
columns->Array.filter(col => !ColumnId.equal(Column.id(col), Column.id(targetColumn)))
```

### Module Types for Backend Implementations

When a module claims to implement an interface (e.g., "implements BoardStorage.Backend"), enforce it via the `.resi` file:

```rescript
// LocalStorageBackend.resi
include BoardStorage.Backend
```

This makes the contract explicit and compiler-enforced, not just a comment.

## Bindings and Libraries

If bindings for a particular library or browser API are unclear or missing, ask the user where to find them. Identify these before starting to write code.

## Type Errors

If you encounter type inference errors you're having trouble with, ask the user for assistance. Common issues include:

- Record types not found: may need module prefix like `{TheModule.foo: "bar"}`
- Record field acccess: `value.TheModule.property`
- Variant constructors not in scope: may need to open module or prefix like `let value = TheModule.Variant("payload")`

Do not use `Obj.magic` as a workaround without user input.

## Style

- Utilize punning of react props and record fields to create succinct code. e.g.
  ```rescript
  let dataTestId = "my-button"
  <div dataTestId>{React.string("Click Me")}</div>
  ```
- Prefer pattern matching over `if` / `else` expressions or ternary operators.
- Try to flatten nested switch expressions into one top-level one, except if you have many
  branches of the switch that only look at a one case in a long tuple, >6 elements.
- Choose the most semantic function for the job at hand, for example don't use `Array.forEach`
  with mutable refs when `Array.some` will read better.

### Functional Composition Over Imperative Code

Prefer chaining with `Array.map` / `Option.map` / `Result.map` over mutable refs and imperative loops:

```rescript
// Prefer this:
let updateColumn = (board, columnId, updater) => {
  let index = board.columns->Array.findIndex(col => ColumnId.equal(Column.id(col), columnId))

  board.columns
  ->Array.get(index)
  ->Option.map(updater)
  ->Option.map(column => {...board, columns: Array.with(board.columns, index, column)})
}

// Over this (mutable ref, imperative):
let updateColumn = (board, columnId, updater) => {
  let found = ref(false)
  let newColumns = board.columns->Array.map(column => {
    if ColumnId.equal(Column.id(column), columnId) {
      found := true
      updater(column)
    } else {
      column
    }
  })
  if found.contents {
    Some({...board, columns: newColumns})
  } else {
    None
  }
}
```
