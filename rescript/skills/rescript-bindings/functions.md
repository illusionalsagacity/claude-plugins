# Function Imports

Patterns for binding JavaScript/TypeScript module imports.

## Default Export

```typescript
import DEFAULT_NAME from "{MODULE_NAME}";
```

```rescript
@module("{MODULE_NAME}") external defaultName: {TYPE} = "default"
```

**Example:**

```typescript
import axios from "axios";
```

```rescript
@module("axios") external axios: axiosInstance = "default"
```

## Named Export

```typescript
import { FUNCTION_NAME } from "{MODULE_NAME}";
```

```rescript
@module("{MODULE_NAME}") external functionName: {TYPE} = "{FUNCTION_NAME}"
```

**Example:**

```typescript
import { readFile } from "fs/promises";
```

```rescript
@module("fs/promises") external readFile: string => promise<string> = "readFile"
```

## Named Export with Rename

```typescript
import { ORIGINAL_NAME as ALIAS } from "{MODULE_NAME}";
```

```rescript
@module("{MODULE_NAME}") external alias: {TYPE} = "{ORIGINAL_NAME}"
```

**Example:**

```typescript
import { default as lodash } from "lodash";
```

```rescript
@module("lodash") external lodash: lodashT = "default"
```

## Multiple Named Exports

```typescript
import { foo, bar, baz } from "{MODULE_NAME}";
```

```rescript
@module("{MODULE_NAME}") external foo: {TYPE_FOO} = "foo"
@module("{MODULE_NAME}") external bar: {TYPE_BAR} = "bar"
@module("{MODULE_NAME}") external baz: {TYPE_BAZ} = "baz"
```

## Namespace Import

```typescript
import * as NAME from "{MODULE_NAME}";
```

For namespace imports, create a module with the bindings:

```rescript
module Name = {
  @module("{MODULE_NAME}") external foo: {TYPE} = "foo"
  @module("{MODULE_NAME}") external bar: {TYPE} = "bar"
}
```

## Subpath Imports

```typescript
import { helper } from "{MODULE_NAME}/utils";
```

```rescript
@module("{MODULE_NAME}/utils") external helper: {TYPE} = "helper"
```

## Side-Effect Import

```typescript
import "{MODULE_NAME}";
```

```rescript
%%raw(`import "{MODULE_NAME}"`)
```

## CommonJS Require

```typescript
const pkg = require("{MODULE_NAME}");
```

```rescript
@module("{MODULE_NAME}") external pkg: {TYPE} = "default"
```

Or if the module doesn't use default export:

```rescript
type pkgModule
@module("{MODULE_NAME}") external pkg: pkgModule = "{}"
```

## Function with Options Object

```typescript
import { configure } from "pkg";
configure({ debug: true, timeout: 5000 });
```

```rescript
type configOptions = {
  debug?: bool,
  timeout?: int,
}

@module("pkg") external configure: configOptions => unit = "configure"

// Usage
configure({debug: true, timeout: 5000})
```

## Async Function

```typescript
import { fetchData } from "api";
const data = await fetchData(url);
```

```rescript
@module("api") external fetchData: string => promise<'a> = "fetchData"

// Usage
let data = await fetchData(url)
```