# Code Quality Hook

**Type:** validation
**When:** Appended as constraints during agent execution

Enforces code quality rules that the agent must self-validate against.
The agent checks its own output for compliance with these rules.

## Rules

### No Dead Code
- No commented-out code blocks
- No unreachable code after return/throw statements
- No unused imports, variables, functions, or classes
- No TODO/FIXME/HACK comments -- resolve or remove them

### Documentation
- Every exported function, class, or interface has a doc comment
- Doc comments include: description, parameters, return value, thrown exceptions
- Every file has a module-level doc comment explaining its purpose
- Complex logic has inline comments explaining WHY, not WHAT

### Single Responsibility Functions
- Each function does exactly one thing
- If a function description requires "and", split it
- Maximum ~40 lines per function body
- Extract helper functions for complex sub-operations

### Code Style
- Descriptive variable names -- no abbreviations unless universally understood
- Early returns to avoid deep nesting (no more than 3 levels of indentation)
- Consistent error handling pattern throughout the codebase
- No global mutable state
- Configuration loaded once and injected, not accessed globally

### Architecture
- Clear separation of concerns: routes/controllers -> services -> data access
- No business logic in route handlers or controllers
- Interfaces/types define contracts between layers
- Dependencies injected, not created inline

## Validation

The agent must verify each rule against its output before finalizing. Any violation
must be corrected before the agent completes its work.
