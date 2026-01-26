# Claude Plugins

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

## rescript-lsp

A Claude Code plugin to hook up the Rescript LSP, and run the formatter on changes.

### Installation

See the [Claude Code plugin marketplace documentation](https://code.claude.com/docs/en/discover-plugins#add-from-github)

You can also clone this repo locally and install it that way, so you can edit the plugin to add your own stuff

### Requirements

- `@rescript/langugage-server`
- `jq`
- `xargs`
- `npx`
