# Callback Bindings

Patterns for binding JavaScript functions that accept callbacks.

## Simple Callback

```typescript
function onClick(handler: () => void): void {}
```

```rescript
@module("lib") external onClick: (unit => unit) => unit = "onClick"
```

## Callback with Arguments

```typescript
function onEvent(handler: (event: Event) => void): void {}
```

```rescript
@module("lib") external onEvent: (event => unit) => unit = "onEvent"
```

## Callback with Return Value

```typescript
function map<T, U>(arr: T[], fn: (item: T) => U): U[] {}
```

```rescript
@module("lib") external map: (array<'a>, 'a => 'b) => array<'b> = "map"
```

## Callback with Multiple Arguments

```typescript
function reduce<T, U>(arr: T[], fn: (acc: U, item: T) => U, initial: U): U {}
```

```rescript
@module("lib") external reduce: (array<'a>, ('b, 'a) => 'b, 'b) => 'b = "reduce"
```

## Callback with Index

```typescript
function forEach<T>(arr: T[], fn: (item: T, index: number) => void): void {}
```

```rescript
@module("lib") external forEach: (array<'a>, ('a, int) => unit) => unit = "forEach"
```

## Event Handler Pattern

```typescript
element.addEventListener("click", (e: MouseEvent) => {});
```

```rescript
type mouseEvent

@send external addEventListener: (
  element,
  @as("click") _,
  mouseEvent => unit
) => unit = "addEventListener"

// Usage
element->addEventListener(e => Console.log(e))
```

## Callback with Error-First Pattern (Node.js)

```typescript
function readFile(
  path: string,
  callback: (err: Error | null, data: string) => void
): void {}
```

```rescript
type nodeError

@module("fs") external readFile: (
  string,
  (Nullable.t<nodeError>, string) => unit
) => unit = "readFile"

// Usage
readFile("file.txt", (err, data) => {
  switch Nullable.toOption(err) {
  | Some(e) => Console.error(e)
  | None => Console.log(data)
  }
})
```

## Promise-Returning with Callback Alternative

```typescript
function fetch(url: string): Promise<Response>;
function fetch(url: string, callback: (resp: Response) => void): void;
```

```rescript
// Prefer promise version
@module("lib") external fetch: string => promise<response> = "fetch"

// If callback version needed
@module("lib") external fetchWithCallback: (string, response => unit) => unit = "fetch"
```

## Subscription Pattern (Returns Unsubscribe)

```typescript
function subscribe(callback: (value: T) => void): () => void {}
```

```rescript
@module("lib") external subscribe: ('a => unit) => (unit => unit) = "subscribe"

// Usage
let unsubscribe = subscribe(value => Console.log(value))
// Later: unsubscribe()
```

## Builder with Callback

```typescript
builder.onSuccess((data) => {}).onError((err) => {}).build();
```

```rescript
module Builder = {
  type t<'a>

  @send external onSuccess: (t<'a>, 'a => unit) => t<'a> = "onSuccess"
  @send external onError: (t<'a>, error => unit) => t<'a> = "onError"
  @send external build: t<'a> => result<'a> = "build"
}
```

## Async Callback (with @uncurry)

For callbacks passed to JS that must not be curried:

```typescript
function asyncOperation(callback: (result: string) => void): void {}
```

```rescript
@module("lib") external asyncOperation: (@uncurry (string => unit)) => unit = "asyncOperation"
```

Note: `@uncurry` ensures the callback is compiled as a single JS function, not a curried ReScript function.

## Labeled Arguments for Clarity

```rescript
// v11.1+ (no trailing unit)
@module("lib") external process: (
  ~onSuccess: response => unit,
  ~onError: error => unit,
  ~onProgress: float => unit=?,
) => unit = "process"

// Usage
process(
  ~onSuccess=resp => Console.log(resp),
  ~onError=err => Console.error(err),
)

// v10.x-v11.0 (trailing unit required)
@module("lib") external process: (
  ~onSuccess: response => unit,
  ~onError: error => unit,
  ~onProgress: float => unit=?,
  unit
) => unit = "process"
```

## This-Binding Callbacks

```typescript
element.addEventListener("click", function(this: Element, e: Event) {
  this.classList.add("clicked");
});
```

```rescript
// Use @this to access the bound this value
@send external addEventListener: (
  element,
  string,
  @this (element, event) => unit
) => unit = "addEventListener"
```
