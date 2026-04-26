# SOP 00 — When to Build a Function

Use this before touching the function canvas. Answer the gates in order. If any gate is a hard no, stop — don't build.

---

## Gate 1 — Is this actually reused?

Does this enrichment pattern exist in more than one place, or is a second use incoming this week?

- **Yes (used in 2+ tables, or second use is happening now):** continue to Gate 2.
- **No (single use, no second use scheduled):** don't build a Function. Inline the logic in the table. Revisit when you hit a second use.

> The 3-use variant: if a third table is committed to use this within 30 days (named, scheduled, owner identified — not speculative), build now to save the migration later. Default is 2+ in-hand uses.

---

## Gate 2 — Is it stable enough to version?

A Function's interface is a contract. Can you define the inputs and output shape now and commit to not changing them without bumping a version?

- **Yes:** continue to Gate 3.
- **No (you're still figuring out what the output even is):** don't build a Function yet. Build it inline in one table, run it, look at the real output, then come back and fill the canvas when you know what you're building.

---

## Gate 3 — Does Clay have this natively?

Check Clay's native enrichment providers and formulas. Does any built-in tool already do this?

- **No native equivalent:** continue to Gate 4.
- **Yes, native covers it:** use the native tool. Don't rebuild it as a Function unless you're wrapping it with custom logic that materially changes the output.

---

## Gate 4 — Is this actually a single concern?

Name what the Function does in one sentence without using "and."

- **One clear concern:** continue to Canvas.
- **You need "and":** you're describing two Functions. Split them. Run Gate 1–3 on each half separately.

---

## Gate passed — go fill the canvas

Copy `assets/templates/function-canvas.md`, fill it out, then run it through `sop/01-design.md` to finalize the interface before building anything.

---

## Quick reference — common stop cases

| Situation | Decision |
|---|---|
| First table ever needs this | Inline it. Come back at second use. |
| "I might need this again" | Inline it. Commit to a Function when "might" becomes "yes." |
| Enrichment exists in one table but you've copy-pasted the logic | This is the second use. Build the Function. |
| Clay's native "find email" covers 80% of what you need | Use native. Only build a Function if the 20% gap is real and recurring. |
| You can't name it without "and" | It's two Functions. Split first. |
