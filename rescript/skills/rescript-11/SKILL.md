---
name: rescript-11
description: ReScript 11/11.1 syntax features. Use when writing ReScript in projects using v11+. Check package.json dependencies for rescript version. Key features include customizable variants (@unboxed, @as, @tag), record type spread, async/await, and array spread syntax.
---

# ReScript 11 Syntax Features

## ReScript 11.0

### Customizable Variants

Variants can now map directly to JavaScript discriminated unions with zero runtime overhead.

#### @unboxed for Primitive Variants

Strip wrapper tags, leaving only payloads:

```rescript
@unboxed
type jsonValue =
  | String(string)
  | Boolean(bool)
  | Number(float)

let values = [String("hello"), Boolean(true), Number(42.0)]
// Compiles to: ["hello", true, 42.0]
```

#### @unboxed with @as for String Enums

Create string enums that compile to their literal values:

```rescript
@unboxed
type status = | @as("pending") Pending | @as("active") Active | @as("done") Done

let current = Active
// Compiles to: "active"
```

No `toString`/`fromString` needed—the variant IS the string at runtime.

#### @tag for Discriminated Unions

Customize the discriminator property name:

```rescript
@tag("type")
type event =
  | @as("click") Click({x: int, y: int})
  | @as("keydown") KeyDown({key: string})

let e = Click({x: 10, y: 20})
// Compiles to: {type: "click", x: 10, y: 20}
```

#### Nullable Pattern Matching

```rescript
@unboxed
type nullable<'a> = Present('a) | @as(null) Null

let handleNullable = value =>
  switch value {
  | Present(v) => Some(v)
  | Null => None
  }
```

### Record Type Spread

Extend record types by spreading fields:

```rescript
type entity = {id: string, createdAt: Date.t}
type user = {...entity, name: string, email: string}
// user = {id: string, createdAt: Date.t, name: string, email: string}
```

### Record Type Coercion

Coerce between structurally compatible record types using `:>`:

```rescript
type full = {id: string, name: string, extra: int}
type partial = {id: string, name: string}

let asFull: full = {id: "1", name: "test", extra: 42}
let asPartial = (asFull :> partial)  // Zero-cost, type-level only
```

## ReScript 11.1

### Array Spread

Use spread syntax instead of `Array.concat`:

```rescript
let withNew = [...existing, newItem]
let prepended = [first, ...rest]
let combined = [...arr1, ...arr2]
```

### Omit Trailing Undefined in Externals

External function calls now omit trailing `undefined` arguments automatically. This is important when binding to JavaScript APIs that behave differently based on argument count.

```rescript
@val
external stringify: ('a, ~replacer: (string, JSON.t) => JSON.t=?, ~space: int=?) => string = "JSON.stringify"

let result = stringify(obj)
// Compiles to: JSON.stringify(obj)
// NOT: JSON.stringify(obj, undefined, undefined)
```

**Common mistake:** Forgetting that this behavior exists and manually handling undefined, or being surprised when a JS API works correctly without explicit undefined passing.
