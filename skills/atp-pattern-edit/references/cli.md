# patautoedit CLI

`patautoedit` applies a row-per-operation config CSV to ATP files. It needs
Python 3.10 or newer. Install it with `pip install patautoedit`.

## Commands

### `patautoedit run CONFIG [options]`

Apply the config and write edited ATPs to the output folder.

```bash
patautoedit run config.csv --atp-dir path/to/atps --output-dir Output --json
```

### `patautoedit check CONFIG [options]`

Validate the config and print the plan. Writes nothing.

```bash
patautoedit check config.csv --atp-dir path/to/atps --json
```

### `patautoedit modes [--json]`

List the Process Types and the fields each one requires.

```bash
patautoedit modes --json
```

### `patautoedit --version`

Print the version.

## Common options

| Option | Meaning |
| --- | --- |
| `--atp-dir DIR` | Folder holding `*.atp` and `*.atp.gz` files. Defaults to the config file's folder. Scans that folder only, not subfolders. |
| `--atp FILE` | Candidate ATP file. Repeatable. Overrides `--atp-dir`. |
| `--pinmap FILE` | Optional pinmap file. Use it when the config names a pin group. |
| `--output-dir DIR` | `run` only. Folder for edited files. Defaults to `Output/` under the current folder. |
| `--json` | Print one JSON object on stdout. Logs and warnings go to stderr. |

## JSON: `run`

```json
{
  "version": "V1.17.0",
  "ok": true,
  "config": "/path/config.csv",
  "output_dir": "/path/Output",
  "outputs": [
    {
      "source": "/path/Sample.atp",
      "output": "/path/Output/Sample.atp",
      "operations": 3
    }
  ],
  "failed": [
    {
      "source": "/path/Bad.atp",
      "errors": ["..."]
    }
  ],
  "skipped": ["Missing.atp"],
  "warnings": ["Cannot find atp file: Missing.atp"],
  "problems": [],
  "counts": {
    "matched": 2,
    "succeeded": 1,
    "failed": 1
  }
}
```

`counts.matched` is the number of candidate ATPs that matched a Pattern.
Skipped Patterns are not counted.

## JSON: `check`

```json
{
  "version": "V1.17.0",
  "ok": true,
  "config": "/path/config.csv",
  "plan": [
    {
      "source": "/path/Sample.atp",
      "operations": ["DSSC Capture", "WFLAG"]
    }
  ],
  "skipped": ["Missing.atp"],
  "warnings": ["Cannot find atp file: Missing.atp"],
  "problems": [],
  "counts": {
    "matched": 1
  }
}
```

`check` has no `output_dir`, `outputs`, or `failed`. It does not write.

## JSON: `modes`

A list of `{"process_type": "...", "requires": ["..."]}` objects. Read it at
runtime instead of assuming what a newer Process Type needs.

## Exit codes

| Code | Meaning |
| --- | --- |
| `0` | Success. A Pattern that matches no candidate is a warning and a skip, not a failure. |
| `1` | A rejected config, a failed file, or no matched file. |
| `2` | Usage error, such as a missing config, ATP file, or pinmap. |

## Behavior notes

- The CLI always uses Single time mode and Cycle index. Dual-mode or
  vector-index files need the desktop app's Classical tab.
- A Pattern that names no candidate ATP is warned and skipped. If no Pattern
  matches at all, the run exits 1.
- The whole config is checked before any file is written.
- Warnings and log lines go to stderr. Parse stdout as JSON.
