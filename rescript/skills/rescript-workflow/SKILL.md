---
name: rescript-workflow
description: Phased development workflow for ReScript. Use when implementing new features or modules in ReScript projects. Guides through interface design, implementation skeletons with %todo, test writing, implementation, and compiler verification.
---

# ReScript Phased Development Workflow

When implementing ReScript features, follow this phased approach. For large features spanning multiple modules, pause for user review after each phase before proceeding.

**Review behavior:** If the user is in auto-accept mode (accepting edits without review), skip the review pauses and proceed through phases continuously. Only pause for explicit review when the user is actively reviewing each change.

## Phase 1: Interface Design

If there is an interface `*.resi` file for the module and the new code is intended to be public, add the type annotation there first:

```rescript
let log: string => unit
```

## Phase 2: Implementation Skeletons

Write implementation skeletons using explicit type annotations and `%todo("<DESCRIPTION>")`:

```rescript
let log: string => unit = %todo("Takes a string and logs it to stdout")
```

For React event handlers passed to JSX elements, you can omit the type annotation since the compiler infers it from the element prop:

```rescript
let handleClick = %todo("Handle button click, update state")
// ...
<button onClick={handleClick}> ... </button>
```

### Phase 1-2 Review

When pausing after interfaces and skeletons, provide:

1. Brief summary of what was completed
2. List of `.resi` files created for reviewing interfaces
3. A diagram showing how the modules connect

Example:

```
## Phase 2 Complete: Implementation Skeletons

Created 12 modules for the kanban feature.

### Interface files for review:
- src/kanban/TaskId.resi
- src/kanban/Task.resi
- src/kanban/Board.resi
- src/kanban/BoardOperations.resi

### Module structure:
┌─────────────┐     ┌─────────────┐
│   TaskId    │     │  ColumnId   │
└──────┬──────┘     └──────┬──────┘
       └───────────────────┤
                           ▼
              ┌─────────────────────────┐
              │  Task  │ Column │ Board │
              └───────────┬─────────────┘
                          ▼
              ┌─────────────────────────┐
              │    BoardOperations      │
              └─────────────────────────┘

Ready for Phase 3 (Tests)?
```

## Phase 3: Write Tests

Before implementing the functions, write fully implemented tests. The tests themselves should be complete (not using `%todo`), so they will fail with a runtime exception when run against the `%todo` skeletons:

```rescript
describe("BoardOperations", () => {
  it("adds task to specified column", t => {
    let board = Board.createDefault()
    let columnId = board->Board.columns->Array.getUnsafe(0)->Column.id
    let task = Task.make(
      ~id=TaskId.make(),
      ~title="Test task",
      ~createdAt=Date.make(),
      ~updatedAt=Date.make(),
    )

    let result = BoardOperations.addTask(board, ~columnId, ~task)

    t->expect(Result.isOk(result))->toBe(true)
  })
})
```

This follows red-green-refactor: tests are complete and failing, serving as guidance for future work, then implement the actual code to make them pass.

## Phase 4: Implementation

Implement the actual functions, replacing `%todo` placeholders with working code.

## Phase 5: Compiler Verification

After implementation, run the ReScript compiler with warning-as-error for unused `%todo`:

```sh
# ReScript 11
rescript -warn-error +110

# ReScript 12+
rescript --warn-error +110
```

Run via your package manager (e.g., `npx`, `yarn`, `pnpm exec`) as appropriate for the project.

This ensures no `%todo` placeholders remain in the codebase. If any `%todo` remains, the build will fail with an error rather than a warning.

### Phase 4-5 Review

When pausing after implementation, provide:

1. Summary of what was implemented
2. Test results (pass/fail)
3. Compiler verification result (any remaining `%todo`?)
4. Assumptions and trade-offs not explicitly in the plan
5. Any deviations from the plan or interfaces
6. Any known limitations or TODOs

Example:

```
## Phase 5 Complete: Implementation Verified

All 12 modules implemented. Compiler verification passed (no remaining %todo).

### Test results:
✓ 15 tests passing
✗ 0 tests failing

### Assumptions & trade-offs:
- Assumed task titles have max length of 200 characters
- Board auto-saves on every change (simpler UX, more writes)

### Deviations from plan:
- Used `ReactEvent.Mouse.t` for drag events (ReactEvent.Drag not in bindings)

### Known limitations:
- Drag position doesn't account for scroll offset

Ready for final review?
```

---

## Refactoring

ReScript's compiler halts progress until the program type-checks. This differs from TypeScript/JavaScript where you can run partially broken code and discover issues at runtime. Without a strategy, refactoring can leave you stuck in a non-compiling state, chasing type errors reactively.

**Use `%todo` to sketch structure before implementation**, even when refactoring. This keeps the code compiling while you work out the shape of the change, turning the compiler from a blocker into an ally.

### Refactoring Process

1. **Understand first** - Read the code being refactored and identify its callers
2. **Check test coverage** - If tests don't exist for the code being changed, write them first
3. **Sketch the new structure with `%todo`**:
   - Define new functions/modules with explicit type annotations and `%todo` bodies
   - This establishes the target structure while maintaining a compiling program
4. **Write tests for new behavior** - Before implementing `%todo` bodies, write tests that clarify expected behavior
5. **Edit existing code to use the new structure** - Since signatures exist, this compiles
6. **Fill in `%todo` implementations** - One at a time, always maintaining compilability
7. **Verify** - Run tests, then `rescript --warn-error +110` to catch any remaining `%todo`

### Example: Breaking Down a Function

Instead of editing the original function and reactively creating helpers as the compiler complains:

```rescript
// 1. Sketch the new helpers with %todo
let validateInput: string => result<validatedInput, validationError> =
  %todo("Validate and parse the raw input")

let processValidated: validatedInput => output =
  %todo("Core processing logic on validated input")

// 2. Write tests for the new behavior

// 3. Edit the original function to use them (compiles because signatures exist)
let process = (input: string): result<output, error> => {
  input
  ->validateInput
  ->Result.map(processValidated)
}

// 4. Fill in each %todo one at a time
```

### Preserving Interfaces

- **If `.resi` doesn't change**: Internal refactoring only, callers unaffected
- **If `.resi` must change**: Identify all callers first, update interface and callers together
- The compiler will show you every caller that needs updating when you change a signature