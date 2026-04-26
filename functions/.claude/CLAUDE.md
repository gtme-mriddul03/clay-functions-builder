# Functions — Context

This folder is a register of shipped Clay Functions. It is not where function files are stored.

**File locations (per the v2 skill):**

| Artifact | Where it lives |
|---|---|
| Canvas draft (pre-build) | `../skill/working/<function-name>-canvas.md` |
| Function spec | `../skill/docs/<function-name>-spec.md` |
| Usage doc | `../skill/docs/<function-name>-usage.md` |
| ADR (if needed) | `../skill/decisions/<date>-<function-name>.md` |

**To build a function:** invoke `/clay-functions-v2` and start at `sop/00-when-to-build.md`.

**When a function ships:** add a row to `README.md` in this folder with the function name and links to its spec and usage doc in `../skill/docs/`.