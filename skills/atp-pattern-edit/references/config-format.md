# Config CSV format

One row is one Operation. Rows that share the same `Pattern` form a chain and
are applied top to bottom. Each Operation reads the previous Operation's output.

## Columns

| Column | Required | Notes |
| --- | --- | --- |
| `Pattern` | yes | ATP filename. It must match a candidate ATP by basename, with or without `.gz`. |
| `Process Type` | yes | The edit to apply. See the table below. |
| `Pin or Method` | no | Used only by DSSC Capture, DSSC Source, and CMEM/HRAM Capture. |
| `Cycle` | no | `[a-b];[c-d]` form. Used by DSSC Capture, DSSC Source, CMEM/HRAM Capture, WFLAG, and Add Opcode. |
| `User String` | no | Opcode text for Add Opcode and Remove Opcode; counter run counts for the Expand modes. |
| `Instance`, `Status` | no | Ignored. Kept for bookkeeping. |

Column names are case-insensitive, surrounding whitespace is ignored, and
column order does not matter. Unknown columns are reported and ignored.

## Process Types

| Process Type | Needs |
| --- | --- |
| `DSSC Capture` | Pin or Method, Cycle |
| `DSSC Source` | Pin or Method, Cycle |
| `CMEM/HRAM Capture` | Pin or Method, Cycle |
| `WFLAG` | Cycle |
| `Add Opcode` | Cycle, User String |
| `Remove Opcode` | User String |
| `Expand Pattern` | Nothing (old name of Expand Repeat) |
| `Expand Repeat` | Nothing; optional counter run counts in User String |
| `Expand Loop` | Nothing; optional counter run counts in User String |
| `Compress Pattern` | Nothing |

Run `patautoedit modes --json` for the same list from the installed CLI.

## Chains

Rows for the same Pattern run in row order:

```csv
Pattern,Process Type,Pin or Method,Cycle,User String
Sample_ATP.atp,DSSC Capture,TDO,[7-7];[12-12],
Sample_ATP.atp,DSSC Capture,PORN,[13-14],
Sample_ATP.atp,DSSC Source,TDI,[9-9],
```

This gives one output with three Operations. If one Operation fails, the chain
stops and no output is produced for that file.

## Preloaded counters

Some patterns use `loop cX` or `repeat cX` on a counter the test program loads
before the pattern starts. Put the run counts in `User String` on the Expand
rows:

```csv
Pattern,Process Type,Pin or Method,Cycle,User String
march_full.atp,Expand Loop,,,c0=<count>; c7=<count>; c8=<count>
march_full.atp,Expand Repeat,,,c0=<count>; c7=<count>; c8=<count>
```

- Write `cN=<run count>` pairs separated by `;`.
- Counters are `c0` to `c15`.
- Values are decimal, from 1 to 268,435,455, and each counter appears once.
- A run count is the N in `set cX N`. A VBT line
  `TheHdw.Digital.Patgen.Counter(tlPgCounterN) = X` writes the register
  directly, so its run count is `X + 1`.
- A `set cX` in the ATP wins from that vector on. It still needs its own
  `set_msb` first.
- A counter the Process Type needs, with no value and no `set`, fails the file.
- Values the Process Type does not need are ignored with one info line.

## Limits

- The config path uses Single time mode and Cycle index. Dual-mode or
  vector-index files need the desktop app's Classical tab.
- `Expand Pattern`, `Expand Repeat`, and `Compress Pattern` always use single
  time mode. `Expand Loop` supports single mode only.
