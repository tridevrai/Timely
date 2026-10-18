# Discovered Rules

## Mandated

- ALWAYS validate input at system boundaries (forms, APIs, imported files).

## Forbidden

- NEVER commit secrets or credentials; keep them in environment variables or a secrets manager.
- NEVER lower the coverage floor or disable a failing test to make a check pass.
- NEVER add a dependency without pinning its version in a lockfile.
