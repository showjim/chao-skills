---
name: atp-pattern-edit
description: >
  Edit ATP pattern files for Teradyne IG-XL DSSC, CMEM, and HRAM test programs
  with the patautoedit CLI. Always use this skill when the user asks to capture,
  source, expand, compress, or otherwise edit .atp or .atp.gz patterns; mentions
  Expand Loop, Expand Repeat, Compress Pattern, WFLAG, Add Opcode, or Remove
  Opcode; supplies a row-per-operation config CSV; or asks to run patautoedit.
---

# ATP Pattern Edit

Use the installed `patautoedit` command to apply a row-per-operation config CSV
to ATP files. One row is one **Operation**; rows that share a **Pattern** form a
chain and run top to bottom.

The CLI is the only engine. Do not import `patautoedit` as a library and do not
parse ATP files to guess pins, cycles, or counter values. The CLI's `check`
command validates the whole config before anything is written.

## 1. Check the CLI

Run:

```bash
patautoedit --version
```

The skill needs 1.17.0 or newer (`V1.17.0` is fine).

If the command is missing, tell the user the CLI is not installed and show:

```bash
pip install patautoedit
```

Ask for permission before running it. If the user agrees, run:

```bash
python -m pip install patautoedit
```

Then verify again. If the user declines, stop and leave them the install command.

## 2. Collect the run settings

Ask for the things the CLI cannot infer:

- The folder holding the candidate ATPs, or the specific ATP files.
- The Pattern for each Operation: the ATP filename as the config should write it.
- The Process Type for each Operation.
- The Cycle range, Pin or Method, and User String that the Process Type needs.

Run `patautoedit modes --json` to see the fields each Process Type requires.
Do not invent a cycle range, pin name, or counter run count. If the user has not
given one, ask.

If the user already has a config CSV, use it and skip to step 4.

## 3. Write the config CSV

Write `atp-pattern-edit-config.csv` in the current folder. If that file already
exists, ask before overwriting it and offer a new name.

Use these columns:

```csv
Pattern,Process Type,Pin or Method,Cycle,User String
Sample_ATP.atp,DSSC Capture,TDO,[7-7];[12-12],
Sample_ATP.atp,WFLAG,,[8-8],
```

- `Pattern`, `Process Type`, `Pin or Method`, and `Cycle` are the required
  columns. `User String` is optional. `Instance` and `Status` are ignored.
- One row is one Operation. Rows for the same Pattern are applied in order.
- For preloaded counters, write `cN=<count>` pairs separated by `;`, for example
  `c0=<count>; c7=<count>`. Use the user's real values in the config; use
  placeholders only in documentation.
- See `references/config-format.md` for every Process Type and its fields.

Show the CSV to the user before checking it. This is the moment to catch a wrong
Pattern, pin, or cycle range.

## 4. Check the plan

Run `check` with `--json`:

```bash
patautoedit check atp-pattern-edit-config.csv --atp-dir path/to/atps --json
```

Add `--atp FILE` (repeatable) instead of `--atp-dir` when the user named
specific files. Add `--pinmap FILE` when the user uses pin groups.

`check` writes nothing. Read the JSON on stdout; log lines and warnings are on
stderr. Show the user:

- Each candidate ATP and its Operations, in order.
- Every Pattern in `skipped` (no candidate ATP matched).
- Every entry in `problems`.
- If `counts.matched` is 0, explain that no Pattern matched the chosen folder.

If `problems` is not empty, or `ok` is false, stop. Fix the config or ask the
user; do not run.

If `skipped` is not empty, tell the user those Patterns will not be processed.
If they expected a match, check the `--atp-dir` folder and the Pattern spelling.

## 5. Run after confirmation

Ask the user to confirm the plan. Do not run before they say yes.

Then run:

```bash
patautoedit run atp-pattern-edit-config.csv --atp-dir path/to/atps --output-dir Output --json
```

Use `--output-dir` explicitly. The default is `Output/` under the current
folder; ask if the user wants another one. Never point it at the source ATP
folder. If the output folder already contains a file with the same name, ask
before overwriting it.

## 6. Report the result

Read the JSON and give a short report:

- `ok`
- `counts.succeeded` and `counts.failed`
- The output path for each succeeded file
- Each failed file and its `errors`, verbatim
- Each skipped Pattern
- Any `problems` or `warnings`

Keep the raw JSON available if the user asks for it. Do not dump it by default.

Exit codes: `0` success (including skipped Patterns), `1` a rejected config, a
failed file, or no matched file, `2` a usage error.

## References

- `references/cli.md` — commands, flags, JSON shape, exit codes.
- `references/config-format.md` — columns, Process Types, chains, counters.
