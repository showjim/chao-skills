---
name: shm-detect-tool
description: Analyse shmoo plot pass/fail patterns from ATE CHAR logs using CNN classification. Generate XLSX reports with auto-classification (Pass/Fail/Vol/Freq/Marginal/Hole) and multi-site correlation with overlay comparison. Use when working with shmoo log files (.txt), shmoo plot classification, CHAR log comparison, site-to-site shmoo overlay, or shmoo pattern recognition.
---

# SHM Detect Tool Skill

This skill provides access to the `shm-detect` CLI for analysing shmoo plots from ATE CHAR logs using a trained CNN model, and for generating site-to-site correlation reports.

## Prerequisites

Install via pip (Python 3.9+):
```bash
pip install shm-detect-tool
```

Use the virtual environment for running any commands:
```bash
source .venv/bin/activate
```

Verify installation:
```bash
shm-detect --version
```

## Classification Labels

The CNN model classifies each shmoo plot into one of six categories:

| Label | Meaning |
|-------|---------|
| **Pass** | All test conditions pass, normal working window |
| **Fail** | All conditions fail |
| **Vol** | Voltage-related failure pattern |
| **Freq** | Frequency-related failure pattern |
| **Marginal** | Edge-case pass, partially marginal conditions |
| **Hole** | Failure hole inside pass region |

## JSON Config

Both `analyse` and `correlate` require a JSON config defining how to parse the shmoo log format. A **built-in default config** is bundled with the tool — it targets the **Teradyne CZ Studio Enhanced** format (right Y-axis).

### Config Fields

| Field | Description |
|-------|-------------|
| `keyword_site` | Keyword identifying site information lines |
| `keyword_item` | Keyword identifying test instance/item lines |
| `keyword_start` | Y-axis start marker (shmoo body begins after this) |
| `keyword_end` | X-axis end boundary marker |
| `keyword_pass` | Regex for pass symbols (e.g., `\\+`, `P\|\\*`) |
| `keyword_fail` | Regex for fail symbols (e.g., `-\|E`, `\\.\|#`) |
| `keyword_y_axis_pos` | Y-axis position: `"left"` or `"right"` |

### Built-in Default Config (CZ Studio Enhanced, right Y-axis)

```json
{
  "keyword_site": "Site",
  "keyword_item": "<",
  "keyword_start": "Tcoef(AC Spec)",
  "keyword_end": "X Axis: Vcoef(DC Spec)",
  "keyword_pass": "\\+",
  "keyword_fail": "-|E",
  "keyword_y_axis_pos": "right"
}
```

### Example: Custom Format (left Y-axis)

```json
{
  "keyword_site": "Site: ",
  "keyword_item": "_SHM:",
  "keyword_start": "Tcoef(%)",
  "keyword_end": "Vcoef(%)",
  "keyword_pass": "P|\\*",
  "keyword_fail": "\\.|#",
  "keyword_y_axis_pos": "left"
}
```

## Available Commands

### 1. `analyse` — Classify Shmoo Plots

Parse a shmoo log, run CNN inference on each plot, and generate an XLSX report with classification labels and pass/fail highlighting.

```bash
shm-detect analyse --file <LOG_FILE> [--config <CONFIG>] [--model <MODEL>] [--gap {Disable|15|25}]
```

| Option | Default | Description |
|--------|---------|-------------|
| `--file` | *(required)* | Path to shmoo log file (.txt) |
| `--config` | bundled CZ Studio config | Path to JSON config file |
| `--model` | bundled default model | Path to trained model weights (.pth) |
| `--gap` | `Disable` | Multi-site parallel plot gap: `Disable`, `15`, or `25` |

**Examples:**

```bash
# Basic analysis with built-in defaults (CZ Studio format)
shm-detect analyse --file shmoo.txt

# With custom config
shm-detect analyse --file shmoo.txt --config my_config.json

# Multi-site side-by-side comparison
shm-detect analyse --file shmoo.txt --gap 25

# With custom model
shm-detect analyse --file shmoo.txt --model custom_model.pth
```

**Output:** `<log_file>_report.xlsx` in the same directory as the input file.

### 2. `correlate` — Multi-File Site-to-Site Comparison

Compare shmoo results across multiple log files and sites, generating an overlay report with differences highlighted.

```bash
shm-detect correlate --files <FILE1> [FILE2 ...] --config <CONFIG> [--sites <LABELS>] [--gap <N>]
```

| Option | Default | Description |
|--------|---------|-------------|
| `--files` | *(required)* | One or more shmoo log files to compare |
| `--config` | bundled CZ Studio config | Path to JSON config file |
| `--sites` | `""` (all sites) | Site labels per file, semicolon-separated |
| `--gap` | `25` | Column gap between sites in XLSX |

**Site label format:** Different files separated by `;`, sites within a file by `,`.
- `"0,1;0,2"` → file1: Sites 0 and 1, file2: Sites 0 and 2

**Examples:**

```bash
# Compare two logs (all sites)
shm-detect correlate --files log1.txt log2.txt --config config.json

# Specific sites
shm-detect correlate --files log1.txt log2.txt --config config.json --sites "0,1;0,2"

# Custom column gap
shm-detect correlate --files f1.txt f2.txt f3.txt --config config.json --gap 30
```

**Output:** A correlation XLSX report with site-by-site comparison and overlay column highlighting differences in yellow.

## Typical Workflows

### Workflow A: Analyse Shmoo Log

```
1. Confirm log file path exists
2. Determine config:
   - Default: built-in CZ Studio Enhanced format (inform user)
   - Custom: user provides JSON path, or create interactively (see below)
3. Run: shm-detect analyse --file <path> [--config <path>] [--gap 25]
4. Report the output XLSX path to the user
```

```bash
# Step 1-2: Use default config (CZ Studio Enhanced format)
# Step 3: Run analysis
shm-detect analyse --file shmoo.txt

# Step 4: Report path → shmoo.txt_report.xlsx
```

### Workflow B: Correlate Multiple Logs

```
1. Confirm all log file paths exist
2. Determine config (same as Workflow A)
3. Discover available sites from the log files:
   - Run: shm-detect analyse --file <first_file> --config <config>
   - Or grep for site keyword lines in the log files
4. Present discovered sites to user, ask which to compare
5. Run: shm-detect correlate --files <paths> --config <config> --sites "<labels>"
6. Report the output XLSX path to the user
```

```bash
# Step 3: Discover sites by checking the log
# (grep for "Site" lines in the log file to identify available site numbers)

# Step 5: Run correlation with user-selected sites
shm-detect correlate --files log1.txt log2.txt --config config.json --sites "0,1;0,2"
```

## Interactive Config Creation

When the user's log format does not match the built-in CZ Studio Enhanced default, guide them through creating a custom JSON config:

1. **Ask about site keyword** — "What text appears on site information lines?" (e.g., `"Site: "`, `"Site"`)
2. **Ask about item keyword** — "What text identifies test item/instance lines?" (e.g., `"_SHM:"`, `"<"`)
3. **Ask about Y-axis start** — "What keyword marks the Y-axis label start?" (e.g., `"Tcoef(%)"`, `"Tcoef(AC Spec)"`)
4. **Ask about X-axis end** — "What keyword marks the X-axis end?" (e.g., `"Vcoef(%)"`, `"X Axis: Vcoef(DC Spec)"`)
5. **Ask about pass symbols** — "What characters represent Pass?" (e.g., `P`, `*`, `+`)
6. **Ask about fail symbols** — "What characters represent Fail?" (e.g., `.`, `#`, `-`, `E`)
7. **Ask about Y-axis position** — "Is the Y-axis on the left or right side of the shmoo body?"

Then generate the JSON (escaping regex properly with `\\`) and save to a file the user specifies (or default to `SHM_keywords_setting.json` in the working directory).

**Regex escaping rules:**
- Literal `.` → `\\.`
- Literal `*` → `\\*`
- Literal `+` → `\\+`
- Multiple symbols → use `|` alternation: `P|\\*`

## Output Files

| Command | Output Filename |
|---------|----------------|
| `analyse` | `<log_file>_report.xlsx` |
| `correlate` | Correlation XLSX report in working directory |

## Important Notes

- **Default config** = Teradyne CZ Studio Enhanced format (right Y-axis). Always inform user which format is in use.
- **Default model** = bundled CNN model. Only specify `--model` if user provides a custom `.pth` file.
- **Intermediate CSV** is automatically cleaned up after report generation.
- **`--gap` option** arranges multi-site results side-by-side in the XLSX: `Disable` (sequential), `15` or `25` (column gap).
- **Site label format** for correlate: files separated by `;`, sites within file by `,`. Example: `"0,1;0,2"`.
- The tool does NOT require specifying `--config` — if omitted, the built-in CZ Studio default is used.
