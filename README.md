# Log Quality Analyzer

A Python-based tool for analyzing log files to detect noisy logging, sensitive data exposure, and incorrect severity usage. Designed for integration into unit test workflows and CI/CD pipelines.

## Overview

This tool analyzes log files and generates an HTML report identifying three categories of logging issues:

1. **Noisy Logs** - Excessive logging at verbose levels (DEBUG, TRACE, INFO)
2. **Sensitive Data Exposure** - Detection of PII, tokens, passwords, and other sensitive information
3. **Severity Violations** - Failure conditions logged at incorrect severity levels

## Files

- `noisyLogDetector.py` - Main analyzer script
- `rules.yml` - Configuration file defining detection rules

## Requirements

- Python 3.6+
- PyYAML library

## Installation

```bash
pip install pyyaml
```

## Usage

### Basic Usage

```bash
python3 noisyLogDetector.py <log_file> <output.html>
```

**Arguments:**
- `<log_file>` - Path to the log file to analyze
- `<output.html>` - Path for the generated HTML report

**Example:**
```bash
python3 noisyLogDetector.py app.log report.html
```

### Integration in Unit Test Workflows

#### GitHub Actions Example

```yaml
name: Log Quality Check

on: [push, pull_request]

jobs:
  analyze-logs:
    runs-on: ubuntu-latest
    steps:
      - uses: actions/checkout@v3

      - name: Set up Python
        uses: actions/setup-python@v4
        with:
          python-version: '3.x'

      - name: Install dependencies
        run: pip install pyyaml

      - name: Run tests and capture logs
        run: |
          ./run_tests.sh > test.log 2>&1 || true

      - name: Analyze log quality
        run: |
          python3 noisyLogDetector.py test.log log-report.html

      - name: Upload report
        uses: actions/upload-artifact@v3
        with:
          name: log-quality-report
          path: log-report.html
```

#### Local Test Integration

```bash
# Run your tests and save logs
./your_test_command > test_output.log 2>&1

# Analyze the logs
python3 noisyLogDetector.py test_output.log report.html

# Open the report
xdg-open report.html  # Linux
open report.html      # macOS
start report.html     # Windows
```

## Configuration

Edit `rules.yml` to customize detection rules:

### Sensitive Patterns

Add regex patterns to detect sensitive information:

```yaml
sensitive_patterns:
  - '(?i)\btoken\s*[:=]\s*[A-Za-z0-9_-]{10,}\.[A-Za-z0-9_-]{20,}\.[A-Za-z0-9_-]{20,}\b'
  - '(?i)\b(api[_-]?key|apikey)\s*[:=]\s*[A-Za-z0-9]{20,}\b'
  - '(?i)\b(password|passwd|pwd)\s*[:=]\s*\S{6,}\b'
```

### Noisy Log Levels

Define which log levels are considered noisy:

```yaml
noisy_log_levels:
  - DEBUG
  - TRACE
  - INFO
```

### Failure Keywords

Keywords that indicate failure conditions:

```yaml
failure_keywords:
  - 'failed'
  - 'error'
  - 'exception'
  - 'timeout'
```

### Required Severity on Failure

Log levels that must be used when failures occur:

```yaml
required_severity_on_failure:
  - ERROR
  - WARN
```

## Supported Log Formats

The analyzer recognizes log lines with the following timestamp formats:

- `HH:MM:SS` or `HH:MM:SS.ssssss` (e.g., `04:31:14` or `04:31:14.109764`)
- `YYYY-MM-DD HH:MM:SS` or `YYYY-MM-DD HH:MM:SS.sss` (e.g., `2024-11-11 04:31:14`)
- `Mon DD HH:MM:SS` (e.g., `Nov 11 04:31:14`)

Lines without recognized timestamps are ignored.

## Output

The tool generates an HTML report with three sections:

1. **Noisy Logs** - Lines logged at noisy levels
2. **Sensitive / PII Logs** - Lines containing sensitive data (redacted in report)
3. **Severity Violations** - Failures not logged at appropriate severity

Each entry includes:
- Line number in the original log file
- Reason for flagging
- Log content (with sensitive data redacted as `[REDACTED]`)

## Exit Codes

- `0` - Analysis completed successfully
- `1` - Error (missing file, invalid configuration, etc.)

## CI/CD Best Practices

1. **Always run on test logs** - Don't analyze production logs in CI
2. **Archive reports** - Save HTML reports as build artifacts
3. **Set thresholds** - Optionally fail builds if too many issues found
4. **Regular reviews** - Periodically review and update `rules.yml`

## Troubleshooting

### Logs not being detected

- Verify timestamp format matches supported patterns
- Check that timestamps are at the start of each line
- Ensure log lines have proper log level markers (ERROR, WARN, INFO, etc.)

### Configuration errors

```
Error: rules.yml is missing required keys: ...
```

Ensure all required keys are present in `rules.yml`:
- `sensitive_patterns`
- `failure_keywords`
- `noisy_log_levels`
- `required_severity_on_failure`

### File not found

```
Log file not found: <path>
```

Verify the log file path is correct and the file exists.

## Contributing

1. Fork the repository
2. Create a feature branch
3. Make your changes
4. Update `rules.yml` if adding new detection patterns
5. Submit a pull request

