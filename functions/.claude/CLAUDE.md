# Functions — Context

This folder is the register and home for all shipped Clay Functions.

## File locations

Each function gets its own subfolder:

```
functions/
  <verb_noun>/
    canvas.md   ← pre-build thinking tool (filled before building)
    spec.md     ← interface contract; includes ADRs inline (filled during design, updated on every change)
    usage.md    ← for table builders (filled after first real run)
```

When the skill runs, it checks for a `functions/` folder in the working root and creates it if missing. Files are always written here — never to the skill's global installation directory.

## Register

When a function ships, add a row to `README.md`:

| Function | Version | Spec | Usage doc | Status |
|---|---|---|---|---|

## To build a function

Invoke `/clay-functions-v2` and start at `sop/00-when-to-build.md`.
