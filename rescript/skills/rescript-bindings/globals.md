# Global Bindings

Patterns for binding global JavaScript values and functions.

## Simple Global Value

```typescript
const value = window.innerWidth;
```

```rescript
@val external innerWidth: int = "innerWidth"

// Or with explicit scope
@val @scope("window") external innerWidth: int = "innerWidth"
```

## Global Function

```typescript
setTimeout(() => {}, 1000);
parseInt("42", 10);
```

```rescript
@val external setTimeout: (unit => unit, int) => float = "setTimeout"
@val external parseInt: (string, int) => int = "parseInt"
```

## Nested Global Object

```typescript
Math.random();
Math.floor(3.14);
JSON.stringify(obj);
```

```rescript
@val @scope("Math") external random: unit => float = "random"
@val @scope("Math") external floor: float => int = "floor"
@val @scope("JSON") external stringify: 'a => string = "stringify"
```

## Deeply Nested Global

```typescript
window.location.href;
document.body.style.color;
```

```rescript
@val @scope(("window", "location")) external href: string = "href"
@val @scope(("document", "body", "style")) external color: string = "color"
```

## Global with Null/Undefined Return

```typescript
const el = document.getElementById("app"); // Element | null
```

```rescript
@val @scope("document") @return(nullable)
external getElementById: string => option<element> = "getElementById"
```

## Global Object as Module

```typescript
console.log("hello");
console.error("error");
```

```rescript
module Console = {
  @val @scope("console") external log: 'a => unit = "log"
  @val @scope("console") external log2: ('a, 'b) => unit = "log"
  @val @scope("console") external log3: ('a, 'b, 'c) => unit = "log"
  @val @scope("console") external error: 'a => unit = "error"
  @val @scope("console") external warn: 'a => unit = "warn"
}
```

## Global Constructor

```typescript
new Date();
new Date("2024-01-01");
new Date(timestamp);
```

```rescript
type date

@new external makeDateNow: unit => date = "Date"
@new external makeDateFromString: string => date = "Date"
@new external makeDateFromTimestamp: float => date = "Date"
```

## Window/Document Properties

```typescript
window.localStorage.getItem("key");
document.querySelector(".class");
```

```rescript
module LocalStorage = {
  @val @scope(("window", "localStorage")) @return(nullable)
  external getItem: string => option<string> = "getItem"

  @val @scope(("window", "localStorage"))
  external setItem: (string, string) => unit = "setItem"

  @val @scope(("window", "localStorage"))
  external removeItem: string => unit = "removeItem"
}

module Document = {
  @val @scope("document") @return(nullable)
  external querySelector: string => option<element> = "querySelector"

  @val @scope("document")
  external querySelectorAll: string => nodeList = "querySelectorAll"
}
```

## Process/Environment (Node.js)

```typescript
process.env.NODE_ENV;
process.cwd();
```

```rescript
module Process = {
  module Env = {
    @val @scope(("process", "env")) @return(undefined_to_opt)
    external get: string => option<string> = ""

    // Specific env var
    @val @scope(("process", "env")) @return(undefined_to_opt)
    external nodeEnv: option<string> = "NODE_ENV"
  }

  @val @scope("process") external cwd: unit => string = "cwd"
  @val @scope("process") external argv: array<string> = "argv"
}
```

## Global This

```typescript
globalThis.fetch(url);
```

```rescript
@val @scope("globalThis")
external fetch: string => promise<response> = "fetch"
```