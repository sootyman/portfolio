---
description: Run the QC review process on recent changes
---
Perform a 3-iteration deep review of all changes in the current session:

1. **Correctness & Completeness**: Do the changes implement what was requested? Are there edge cases missed?
2. **Architecture & Tech Debt**: Do changes follow project patterns (see CLAUDE.md)? Any unnecessary complexity introduced?
3. **Security, Performance, Production Readiness**: No exposed secrets? Proper input validation? Performance implications?

Check `git diff` for unstaged changes and `git diff --staged` for staged changes. Reference specific files and line numbers.
