# Type Mappings

Patterns for mapping TypeScript types to ReScript.

## Records vs Objects

**Prefer records** over object types unless:
- The user explicitly requests object types
- The binding requires structural typing (rare)

```rescript
// Preferred: Record (nominal typing)
type user = {name: string, age: int}

// Avoid unless necessary: Object (structural typing)
type user = {"name": string, "age": int}
```

Records provide better type safety through nominal typing and are idiomatic ReScript.

## Records (Interfaces/Objects)

```typescript
interface User {
  name: string;
  age: number;
  email?: string;
}
```

```rescript
type user = {
  name: string,
  age: int,
  email?: string,  // v10.1+ optional field syntax
}
```

**Mutable fields**: If the JS code expects to mutate a field, prefix with `mutable`:

```rescript
type counter = {
  mutable count: int,
  name: string,
}
```

## Readonly Records

```typescript
interface Config {
  readonly apiUrl: string;
  readonly timeout: number;
}
```

```rescript
// ReScript records are immutable by default
type config = {
  apiUrl: string,
  timeout: int,
}
```

## Abstract/Opaque Types

Use abstract types for JS objects you don't need to destructure:

```typescript
// Some complex JS object
declare class Response { ... }
```

```rescript
type response  // Abstract - no internal structure exposed
```

## Union Types with @unboxed (v11+)

```typescript
type StringOrNumber = string | number;
```

```rescript
@unboxed
type stringOrNumber =
  | String(string)
  | Number(float)

// Compiles to just the value (zero-cost)
let x = String("hello")  // JS: "hello"
let y = Number(42.0)     // JS: 42
```

## Literal Union Types

```typescript
type Status = "pending" | "success" | "error";
```

**Ask user preference**: If ReScript version supports `@as`, regular variants are preferred over polymorphic variants. Ask if they want polymorphic variants.

```rescript
// Recommended (v10+): Regular variant with @as
type status =
  | @as("pending") Pending
  | @as("success") Success
  | @as("error") Error

// Alternative: Polymorphic variant (ask user if they prefer this)
type status = [#pending | #success | #error]
```

## Discriminated Union

```typescript
type Result =
  | { type: "success"; data: string }
  | { type: "error"; message: string };
```

```rescript
// Using @tag to specify discriminator field
@tag("type")
type result =
  | @as("success") Success({data: string})
  | @as("error") Error({message: string})
```

## Null vs Undefined

```typescript
// Returns null
function maybeNull(): string | null {}

// Returns undefined
function maybeUndefined(): string | undefined {}

// Returns either
function maybeNullish(): string | null | undefined {}
```

```rescript
// For null: use Nullable
@module("lib") external maybeNull: unit => Nullable.t<string> = "maybeNull"

// For undefined: use option
@module("lib")
external maybeUndefined: unit => option<string> = "maybeUndefined"

// For null or undefined: use @return(nullable)
@module("lib") @return(nullable)
external maybeNullish: unit => option<string> = "maybeNullish"
```

## Generic Types

```typescript
interface Container<T> {
  value: T;
  map<U>(fn: (x: T) => U): Container<U>;
}
```

```rescript
type container<'a> = {
  value: 'a,
}

@send external map: (container<'a>, 'a => 'b) => container<'b> = "map"
```

## Array Types

```typescript
function process(items: string[]): number[] {}
function processReadonly(items: readonly string[]): void {}
```

```rescript
@module("lib") external process: array<string> => array<int> = "process"
@module("lib") external processReadonly: array<string> => unit = "processReadonly"
```

## Tuple Types

```typescript
type Point = [number, number];
type NameAge = [string, number];
```

```rescript
type point = (float, float)
type nameAge = (string, int)
```

## Function Types

```typescript
type Handler = (event: Event) => void;
type Mapper<T, U> = (value: T) => U;
```

```rescript
type handler = event => unit
type mapper<'t, 'u> = 't => 'u
```

## Record with Index Signature

```typescript
interface StringMap {
  [key: string]: string;
}
```

```rescript
type stringMap = Dict.t<string>
```

## Intersection Types

```typescript
type Named = { name: string };
type Aged = { age: number };
type Person = Named & Aged;
```

```rescript
// Flatten into single record
type person = {
  name: string,
  age: int,
}
```

## Mapped/Partial Types

```typescript
type Partial<T> = { [P in keyof T]?: T[P] };

interface User {
  name: string;
  age: number;
}
type PartialUser = Partial<User>;
```

```rescript
// Make all fields optional manually
type partialUser = {
  name?: string,
  age?: int,
}
```

**Important**: Mapped types, conditional types, and other advanced TypeScript type-level features cannot be directly replicated in ReScript. When binding these, **inform the user about any fidelity loss** and manually expand the type for the specific use case.

## Enum Types

```typescript
enum Direction {
  Up = "UP",
  Down = "DOWN",
}
```

```rescript
type direction =
  | @as("UP") Up
  | @as("DOWN") Down
```

## Numeric Enum

```typescript
enum Status {
  Pending = 0,
  Success = 1,
  Error = 2,
}
```

```rescript
type status =
  | @as(0) Pending
  | @as(1) Success
  | @as(2) Error
```
