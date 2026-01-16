# Class Bindings

Patterns for binding JavaScript classes. Model classes as ReScript modules with abstract `t` type.

## Property Binding Style

**Ask the user**: Do they prefer properties as a record type or as getter/setter functions?

**Important exception**: If a property getter returns a **polymorphic type**, use the getter function pattern. Using a record would cause the type to be inferred and "locked" on first access, causing issues when the same instance is used with different type parameters.

```rescript
// Getter pattern - safe for polymorphic returns
@get external getData: t => 'a = "data"

// Record pattern - type gets locked on first access (problematic for polymorphic)
type t = { data: 'a }  // Don't use if 'a varies per call
```

## Basic Class

```typescript
class MyClass {
  constructor(name: string) {}
  greet(): string {}
}

const obj = new MyClass("hello");
obj.greet();
```

```rescript
module MyClass = {
  type t

  @new @module("{MODULE_NAME}") external make: string => t = "MyClass"
  @send external greet: t => string = "greet"
}

// Usage
let obj = MyClass.make("hello")
let greeting = obj->MyClass.greet
```

## Class with Properties

```typescript
class User {
  name: string;
  readonly id: number;

  constructor(name: string) {}
  getName(): string {}
  setName(name: string): void {}
}
```

```rescript
module User = {
  type t

  @new @module("{MODULE_NAME}") external make: string => t = "User"

  // Property getters
  @get external name: t => string = "name"
  @get external id: t => int = "id"

  // Property setter (only for mutable properties)
  @set external setName: (t, string) => unit = "name"

  // Methods
  @send external getName: t => string = "getName"
  @send external setNameMethod: (t, string) => unit = "setName"
}
```

## Class with Static Methods

```typescript
class Math {
  static random(): number {}
  static floor(x: number): number {}
}
```

```rescript
module Math = {
  @val @scope("Math") external random: unit => float = "random"
  @val @scope("Math") external floor: float => int = "floor"
}
```

## Class from Module

```typescript
import { Parser } from "some-parser";

const p = new Parser({ strict: true });
p.parse(input);
```

```rescript
module Parser = {
  type t
  type options = {strict?: bool}

  @new @module("some-parser") external make: options => t = "Parser"
  @send external parse: (t, string) => result = "parse"
}

// Usage
let p = Parser.make({strict: true})
let result = p->Parser.parse(input)
```

## Class with Inheritance (Interface Pattern)

```typescript
class Animal {
  speak(): void {}
}
class Dog extends Animal {
  bark(): void {}
}
```

```rescript
module Animal = {
  type t
  @send external speak: t => unit = "speak"
}

module Dog = {
  type t

  @new @module("{MODULE_NAME}") external make: unit => t = "Dog"
  @send external bark: t => unit = "bark"

  // Inherit Animal methods via external cast
  external asAnimal: t => Animal.t = "%identity"
}

// Usage
let dog = Dog.make()
dog->Dog.bark
dog->Dog.asAnimal->Animal.speak
```

## Class with Generic Type Parameter

```typescript
class Container<T> {
  constructor(value: T) {}
  get(): T {}
  set(value: T): void {}
}
```

```rescript
module Container = {
  type t<'a>

  @new @module("{MODULE_NAME}") external make: 'a => t<'a> = "Container"
  @send external get: t<'a> => 'a = "get"
  @send external set: (t<'a>, 'a) => unit = "set"
}
```

## Singleton / Global Instance

```typescript
// console is a global instance
console.log("hello");
console.error("oops");
```

```rescript
module Console = {
  @val @scope("console") external log: 'a => unit = "log"
  @val @scope("console") external error: 'a => unit = "error"
  @val @scope("console") external warn: 'a => unit = "warn"
}
```

## Chained Methods (Fluent API)

```typescript
builder
  .setName("test")
  .setSize(100)
  .build();
```

```rescript
module Builder = {
  type t

  @send external setName: (t, string) => t = "setName"
  @send external setSize: (t, int) => t = "setSize"
  @send external build: t => result = "build"
}

// Usage with pipe
builder
->Builder.setName("test")
->Builder.setSize(100)
->Builder.build
```