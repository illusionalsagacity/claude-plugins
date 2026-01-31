# Claude Plugins

## rescript-lsp

A Claude Code plugin to hook up the Rescript LSP, run the formatter on changes, and make some Rescript-specific code skills available.

### Installation

See the [Claude Code plugin marketplace documentation](https://code.claude.com/docs/en/discover-plugins#add-from-github)

You can also clone this repo locally and install it that way, so you can edit the plugin to add your own stuff

### Requirements

- `@rescript/langugage-server`
- `jq`
- `xargs`
- `npx`

## rescript-bindings-writer agent

A Claude Code agent for creating, verifying, updating, or extending ReScript bindings for JavaScript/TypeScript libraries.

### Phase 1: Context Gathering

Before writing any code, the agent gathers critical context:

1. **Task Type** - One of:
   - `create` - New bindings from scratch
   - `verify` - Check existing bindings against TypeScript API
   - `update` - Update bindings for a new library version
   - `extend` - Add bindings to an existing set

2. **Required Information**:
   - Library name/version (from package.json)
   - APIs needed (specific list, "core API", or "all")
   - ReScript version (v11+ enables `@unboxed`, `@tag`, uncurried default)
   - Use context: **library** (published package) vs **application** (internal)

3. **For existing bindings** (verify/update/extend): Analyze patterns - module structure, props approach, variant style, naming conventions

### Phase 2: Task-Specific Workflows

**Verify**: Systematic comparison of ALL ReScript bindings against TypeScript `.d.ts` files. Uses a checklist, reports missing/mismatched/deprecated props per component, produces prioritized findings.

**Create**: Full binding creation process:

1. **Research** - Examine TS types, understand JS runtime behavior
2. **Write** - Use appropriate binding attributes (`@module`, `@send`, `@get`, etc.)
3. **Verify** - Compile examples, examine generated JS, compare against library docs
4. **Iterate** - Revise until JS output matches expected usage

### Type Fidelity Verification

The agent explicitly handles TypeScript patterns that change return types (like builder patterns with `withDefault`). When TypeScript uses conditional types, mapped types, or overloads that **cannot be represented in ReScript**, the agent:

1. Identifies the limitation explicitly
2. Explains why ReScript can't express it
3. Presents options to the user (e.g., "all fields nullable" vs "user defines custom types")
4. Documents the trade-off in the bindings

### Output Format

Every binding includes:

- The `.res` binding code
- Example usage demonstrating each binding
- Expected JavaScript output for verification
- Type fidelity report (what's fully represented vs approximated)
- Caveats and limitations

The key philosophy: **never silently simplify** - if TypeScript types can't be fully represented, report back with options and let the user decide on trade-offs.

## rescript-coding-conventions skill

Coding conventions and type design guidance for writing idiomatic ReScript.

### Type Design

- **Model states as variants, not booleans** - Combine multiple `bool` flags into a single variant that represents the entire state, preventing invalid combinations (e.g., `Loading | Loaded('data) | Closed` instead of `{isOpen: bool, isLoading: bool}`)
- **Use the compiler to enforce business rules** - Create opaque types with `make`/parse functions for validated data (e.g., `SocialSecurityNumber.t` with `fromString: string => result<t, error>`)
- **Opaque type aliases** - When a type wraps a primitive but shouldn't be interchangeable with other primitives, make it opaque in the `.resi` interface

### Module Organization

- **Submodules for related types** - Define closely-tied types as submodules (e.g., `Task.Priority.t`)
- **`@unboxed` variants with `@as`** - Compile variant constructors directly to string values, eliminating `toString`/`fromString` boilerplate
- **Comparison at the right level** - If an ID type has equality, the entity type should too

### Style

- Prefer pattern matching over `if`/`else` or ternaries
- Prefer functional composition (`Array.map`, `Option.map`, `Result.map`) over mutable refs and imperative loops
- Use punning for React props and record fields
- Choose the most semantic function (e.g., `Array.some` over `Array.forEach` with a mutable ref)
- Flatten nested `switch` expressions where practical

## rescript-workflow skill

A phased development workflow that uses `%todo` to work _with_ the compiler instead of against it.

ReScript's compiler halts until the program type-checks. Without a strategy, you end up stuck in a non-compiling state, reactively chasing type errors. The solution: sketch structure with `%todo` _before_ implementing. This keeps code compiling while you work out the shape.

### Phases

1. **Interface Design** - Add type signatures to `.resi` files first
2. **Implementation Skeletons** - Write `let fn: type = %todo("description")` placeholders
3. **Write Tests** - Complete tests against skeletons (they'll fail at runtime, that's fine)
4. **Implementation** - Replace `%todo` with working code
5. **Verification** - Run `rescript --warn-error +110` to ensure no `%todo` remains

### Refactoring

Same approach - sketch new helpers with `%todo`, wire up the callers (compiles because signatures exist), then fill in implementations one at a time. The compiler becomes an ally showing you what's left to implement, not a blocker preventing progress.
