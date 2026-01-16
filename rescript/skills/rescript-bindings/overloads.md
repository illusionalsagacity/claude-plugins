# Function Overloads

Patterns for binding JavaScript functions with multiple signatures.

## Preferred: Optional Labeled Arguments

**When argument order is stable**, prefer optional labeled arguments (cleaner API):

```typescript
function configure(options?: { debug?: boolean; timeout?: number }): void;
```

**v11.1+** (no trailing unit needed):
```rescript
type configOptions = {debug?: bool, timeout?: int}

@module("lib") external configure: (~options: configOptions=?) => unit = "configure"

// Usage
configure()
configure(~options={debug: true})
```

**v10.x - v11.0** (trailing unit required):
```rescript
@module("lib") external configure: (~options: configOptions=?, unit) => unit = "configure"

// Usage
configure()
configure(~options={debug: true}, ())
```

## Alternative: Multiple Externals, Same JS Name

Use this pattern when:
- Argument order differs between overloads
- ReScript version doesn't support optional labeled arguments
- User explicitly prefers this style

```typescript
// TypeScript overloads
function create(name: string): Widget;
function create(options: Options): Widget;
function create(name: string, options: Options): Widget;
```

```rescript
type widget
type options = {debug?: bool, timeout?: int}

@module("lib") external create: string => widget = "create"
@module("lib") external createWithOptions: options => widget = "create"
@module("lib") external createWithNameAndOptions: (string, options) => widget = "create"
```

## Mixed Required and Optional

```typescript
function configure(options?: { debug?: boolean; timeout?: number }): void;
```

```rescript
type configOptions = {debug?: bool, timeout?: int}

@module("lib") external configure: (~options: configOptions=?, unit) => unit = "configure"

// Usage
configure()
configure(~options={debug: true}, ())
configure(~options={debug: true, timeout: 5000}, ())
```

## Mixed Required and Optional

```typescript
function request(url: string, options?: RequestOptions): Promise<Response>;
```

```rescript
type requestOptions = {
  method?: string,
  headers?: Dict.t<string>,
  body?: string,
}

// Option 1: Two externals
@module("lib") external request: string => promise<response> = "request"
@module("lib") external requestWithOptions: (string, requestOptions) => promise<response> = "request"

// Option 2: Optional labeled arg
// v11.1+
@module("lib") external request: (string, ~options: requestOptions=?) => promise<response> = "request"
// v10.x-v11.0 (trailing unit required)
@module("lib") external request: (string, ~options: requestOptions=?, unit) => promise<response> = "request"
```

## Type-Based Overloads

```typescript
function parse(input: string): JsonValue;
function parse(input: Buffer): JsonValue;
```

```rescript
type jsonValue

@module("lib") external parseString: string => jsonValue = "parse"
@module("lib") external parseBuffer: buffer => jsonValue = "parse"
```

## Return Type Overloads

```typescript
function getElementById(id: string): Element | null;
function getElementById(id: string, required: true): Element; // throws if not found
```

```rescript
type element

@val @scope("document") @return(nullable)
external getElementById: string => option<element> = "getElementById"

@val @scope("document")
external getElementByIdExn: (string, @as(json`true`) _) => element = "getElementById"
```

## Variadic Functions

```typescript
function log(...args: any[]): void;
console.log("a", "b", "c");
```

```rescript
// Using @variadic - all args must be same type
@variadic @val @scope("console") external log: array<string> => unit = "log"

// Usage
log(["a", "b", "c"])

// For mixed types, create multiple arities
@val @scope("console") external log1: 'a => unit = "log"
@val @scope("console") external log2: ('a, 'b) => unit = "log"
@val @scope("console") external log3: ('a, 'b, 'c) => unit = "log"
```

## Fixed + Variadic Arguments

```typescript
function format(template: string, ...args: any[]): string;
```

```rescript
// Can't directly express fixed + variadic, use multiple arities
@module("lib") external format: string => string = "format"
@module("lib") external format1: (string, 'a) => string = "format"
@module("lib") external format2: (string, 'a, 'b) => string = "format"
@module("lib") external format3: (string, 'a, 'b, 'c) => string = "format"
```

## Constructor Overloads

```typescript
class Date {
  constructor();
  constructor(value: number);
  constructor(value: string);
  constructor(year: number, month: number, date?: number);
}
```

```rescript
type date

@new external makeNow: unit => date = "Date"
@new external makeFromTimestamp: float => date = "Date"
@new external makeFromString: string => date = "Date"
@new external makeFromYearMonth: (int, int) => date = "Date"
@new external makeFromYearMonthDay: (int, int, int) => date = "Date"
```

## Callback Overloads

```typescript
function on(event: "click", handler: (e: MouseEvent) => void): void;
function on(event: "keydown", handler: (e: KeyboardEvent) => void): void;
```

```rescript
type mouseEvent
type keyboardEvent

@module("lib") external onClick: (@as("click") _, mouseEvent => unit) => unit = "on"
@module("lib") external onKeydown: (@as("keydown") _, keyboardEvent => unit) => unit = "on"
```

## Method Overloads

```typescript
class Builder {
  set(key: string, value: string): this;
  set(values: Record<string, string>): this;
}
```

```rescript
module Builder = {
  type t

  @send external set: (t, string, string) => t = "set"
  @send external setMany: (t, Dict.t<string>) => t = "set"
}
```

## Generic Overloads

```typescript
function identity<T>(value: T): T;
function identity<T>(value: T, clone: true): T; // returns cloned value
```

```rescript
@module("lib") external identity: 'a => 'a = "identity"
@module("lib") external identityClone: ('a, @as(json`true`) _) => 'a = "identity"
```

## Naming Convention

When creating multiple bindings for the same JS function, use descriptive suffixes:

```rescript
// Good: descriptive suffixes
external fetch: string => promise<response> = "fetch"
external fetchWithOptions: (string, options) => promise<response> = "fetch"
external fetchWithSignal: (string, options, signal) => promise<response> = "fetch"

// Avoid: numeric suffixes (unclear what differs)
external fetch1: string => promise<response> = "fetch"
external fetch2: (string, options) => promise<response> = "fetch"
```
