---
name: check-info-tool
description: Extract test conditions from Teradyne IG-XL ATE test programs (J750, UltraFLEX, UltraFLEXplus). List jobs, run analysis, and generate structured XLSM reports with pattern usage, flow binning, DC/AC specs, timing, and power pin levels.
---

# Check Info Tool Skill

This skill provides access to the Check Info Tool CLI for extracting critical test condition information from Teradyne IG-XL ATE test programs. It automates the retrieval of pattern usage, flow binning details, power pin levels, AC/timing data, and DC spec information, and produces structured Excel reports.

## Prerequisites

Install via pip (Python 3.9+):
```bash
pip install check-info-tool
```

Use the virtual environment for running any commands:
```bash
source .venv/bin/activate
```

Verify installation:
```bash
check-info --version
```

## Supported Platforms

| Tester | Input Formats |
|--------|--------------|
| UltraFLEXplus | `.igxl`, `.zip`, `.xlsm`, extracted directory |
| UltraFLEX | `.igxl`, `.zip`, `.xlsm`, extracted directory |
| J750 | `.igxl`, `.zip`, `.xlsm`, extracted directory |

## Available Commands

### 1. List Jobs

Parse an IG-XL test program and list all available jobs with their associated sheets:

```bash
check-info list -d <device_path> -p <platform> [options]
```

**Always use `--json` for reliable machine-readable output:**
```bash
check-info list -d ./test_program.igxl -p UltraFLEXplus --json -q
```

Output:
```json
{
  "jobs": {
    "Job_Production": {
      "Flow Table": "Flow_Prod",
      "DC Spec": "DC_Prod",
      "AC Spec": "AC_Prod",
      "PatternSet": ["PatSet_Prod"],
      "Global Spec": "Global Specs",
      "TestInstanceList": ["TestInst1", "TestInst2"]
    },
    "Job_Characterization": {
      "Flow Table": "Flow_Char",
      "DC Spec": "DC_Char",
      "AC Spec": "AC_Char",
      "PatternSet": ["PatSet_Char"],
      "Global Spec": "Global Specs",
      "TestInstanceList": ["TestInst1"]
    }
  }
}
```

### 2. Run Analysis

Run the full analysis on selected jobs and generate XLSM report files:

```bash
check-info run -d <device_path> -p <platform> -j <jobs> [options]
```

**Run all jobs:**
```bash
check-info run -d ./test_program.igxl -p UltraFLEXplus -j all -o ./results --json -q
```

**Run specific jobs (comma-separated, case-sensitive):**
```bash
check-info run -d ./test_program.igxl -p UltraFLEX -j "Job_Production,Job_Characterization" -o ./results --json -q
```

Output:
```json
{
  "output_files": [
    "/path/to/results/CheckInfo_2026-04-17__143052.xlsm"
  ],
  "jobs_run": ["Job_Production", "Job_Characterization"]
}
```

### 3. Version

```bash
check-info version
check-info --version
```

## Common Options

These options apply to both `list` and `run` commands:

| Flag | Default | Description |
|------|---------|-------------|
| `-d`, `--device` | (required) | Path to `.igxl`/`.zip`/`.xlsm` file or extracted directory |
| `-p`, `--platform` | (required) | Tester platform: `UltraFLEXplus`, `UltraFLEX`, or `J750` |
| `--cycle-mode` | `Period` | Display mode for timing values: `Period` or `Frequency` |
| `--context-env` | `""` | Spec context environment prefix |
| `--enable-vbt-check` | off | Enable VBT naming convention checker |
| `--power-order` | `""` | Path to Excel file defining power pin display order |
| `--pattern-path` | `""` | Path to pattern file directory |
| `--json` | off | Output results as JSON (always use this) |
| `-q`, `--quiet` | off | Suppress all progress output |

Run-specific options:

| Flag | Default | Description |
|------|---------|-------------|
| `-j`, `--jobs` | (required) | Comma-separated job names, or `all` |
| `-o`, `--output` | `.` | Output directory for generated reports |

## Output Files

| File | Description |
|------|-------------|
| `CheckInfo_YYYY-MM-DD__HHMMSS.xlsm` | Main report workbook — per-job analysis sheets, pattern/timing maps, sort bin descriptions, power pin levels, and instance parameters |
| `VBT_Checker_Report_*.xlsx` | (Only with `--enable-vbt-check`) VBT naming convention report |

## Report Contents

The generated `.xlsm` report contains the following information per job:

- **Flow Table Analysis** — test flow structure and execution order
- **Pattern Usage Map** — maps patterns to test instances with period and clock information
- **Sort Bin Descriptions** — bin numbers and descriptions across all test suites
- **Power Pin Levels** — DPS voltage levels per test instance with power order support
- **AC/Timing Data** — timing period and MCG clock values (Period or Frequency mode)
- **DC Spec Data** — DC specification values per test instance

## Typical Workflow

1. **Discover jobs** — list all available jobs in the test program
2. **Ask the user** — present the discovered job names and let the user decide which job(s) to process
3. **Run analysis** — execute analysis using only the user-selected jobs

```bash
# Step 1: List available jobs
check-info list -d ./test_program.igxl -p UltraFLEXplus --json -q

# Step 2: Present the job list to the user and ask which job(s) to analyze.
#         Wait for the user's selection before proceeding.

# Step 3: Run analysis on user-selected jobs (use exact names from step 1)
check-info run -d ./test_program.igxl -p UltraFLEXplus -j "Job_Production,Job_Characterization" -o ./results --json -q
# The JSON output contains "output_files" — provide these report paths to the user.
```

## Important Notes

- **Always use `--json -q`** for agent integration — ensures clean, parseable output on stdout
- **Job names are case-sensitive** — use the exact names from the `list` output
- **Exit codes**: `0` = success, `1` = error
- **Output channels**: stdout = structured data (JSON), stderr = progress/logs
- **Platform selection**: must match the actual tester the program was written for
- Supported input formats: `.igxl` (ZIP archive), `.zip`, `.xlsm` (Excel macro workbook), or an extracted directory containing tab-delimited `.txt` sheets
