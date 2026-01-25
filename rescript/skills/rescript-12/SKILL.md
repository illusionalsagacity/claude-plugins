---
name: rescript-12
description: ReScript 12 syntax features. Use when writing ReScript in projects using v12+. Check package.json dependencies for rescript version. Key features include dict literal syntax, unified operators, nested record types, variant pattern spreads, and regex literals.
---

# ReScript 12 Syntax Features

## Dict Literal Syntax

Create and pattern match dictionaries with `dict{}`:

```rescript
// Creating dicts
let taskJson = dict{
  "id": task.id,
  "title": task.title,
  "createdAt": Date.toISOString(task.createdAt),
}

// Pattern matching dicts (matches AT LEAST these keys, not exactly)
let parseTask = json =>
  switch json {
  | dict{"id": id, "title": title} =>
    // This matches any dict containing "id" and "title" keys
    // Additional keys are ignored
    Some(make(~id, ~title))
  | _ => None
  }
```

**Important:** Dict pattern matching is partial—`dict{"a": x}` matches any dict containing key `"a"`, regardless of other keys present.

## Nested Record Types

Define record types inline without auxiliary type declarations:

```rescript
type user = {
  id: string,
  name: string,
  address: {
    street: string,
    city: string,
    zip: string,
  },
}

let user = {
  id: "1",
  name: "Alice",
  address: {
    street: "123 Main St",
    city: "Springfield",
    zip: "12345",
  },
}
```

## Unified Operators

Arithmetic operators work across `int`, `float`, and `bigint` with compiler-inferred specialization:

```rescript
let intResult = 1 + 2
let floatResult = 1.5 + 2.5
let bigintResult = 1n + 2n

// String concat still uses ++
let greeting = "hello" ++ " world"
```

## Bitwise Operators

F#-style bitwise operators replace deprecated OCaml-style functions:

```rescript
let andResult = a &&& b    // AND
let orResult = a ||| b     // OR
let xorResult = a ^^^ b    // XOR
let notResult = ~~~a       // NOT
let leftShift = a << 2     // Logical left shift
let rightShift = a >> 2    // Logical right shift
let unsignedShift = a >>> 2  // Unsigned right shift
```

## Variant Pattern Spreads

Reuse handlers for variant constructor subsets:

```rescript
type response =
  | Success(data)
  | NotFound
  | Unauthorized
  | ServerError(string)

let isError = response =>
  switch response {
  | Success(_) => false
  | ..._ => true  // Matches NotFound, Unauthorized, ServerError
  }
```

## Regex Literals

JavaScript-style regex syntax:

```rescript
let phonePattern = /\d{3}-\d{3}-\d{4}/
let globalMatch = /\w+/g

// Equivalent to:
let phonePattern = %re("/\d{3}-\d{3}-\d{4}/")
```
