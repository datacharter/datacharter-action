# DataCharter data checks — GitHub Action

[![GitHub Marketplace](https://img.shields.io/badge/Marketplace-DataCharter%20data%20checks-3B82C4?logo=github)](https://github.com/marketplace/actions/datacharter-data-checks)
[![test](https://github.com/datacharter/datacharter-action/actions/workflows/test.yml/badge.svg)](https://github.com/datacharter/datacharter-action/actions/workflows/test.yml)

Run **contract-governed data quality checks** in CI. This action runs
[DataCharter](https://github.com/datacharter/datacharter)'s
[`test`](https://datacharter.dev/cli.html) (data assertions declared in your
`charter.yaml`) and [`drift`](https://datacharter.dev/cli.html) (schema + PII
drift against a committed baseline), and **fails the job when they fail** — so a
broken contract blocks the PR, not the pipeline.

Results are written to the job's step summary as a ✓/✗ report.

## Usage

```yaml
name: data-checks
on: [push, pull_request]

jobs:
  data:
    runs-on: ubuntu-latest
    steps:
      - uses: actions/checkout@v5
      - uses: datacharter/datacharter-action@v1
        with:
          workspace: .           # directory containing charter.yaml
          run-drift: "true"      # also check schema/PII drift
```

Declare assertions in your `charter.yaml`:

```yaml
tests:
  id_not_null:  { type: not_null, relation: orders, column: id }
  id_unique:    { type: unique, relation: orders, columns: [id] }
  region_valid: { type: accepted_values, relation: orders, column: region, values: [US, EU] }
  has_rows:     { type: row_count, relation: orders, min: 1 }
  totals_ok:    { type: expression, relation: orders, expression: "amount >= 0" }
```

For the drift check, commit a baseline once (`datacharter drift . --update`
writes `.datacharter/schema.json`); CI then fails when tables, columns, or PII
declarations drift from it.

## Inputs

| Input | Default | Description |
| --- | --- | --- |
| `workspace` | `.` | Path to the workspace (the directory containing `charter.yaml`). |
| `version` | latest | Pin a `datacharter` version, e.g. `0.10.4`. |
| `run-tests` | `true` | Run the `tests:` assertions. |
| `run-drift` | `false` | Run the schema/PII drift check. |
| `select` | — | Run a single test by name. |

## Databases and credentials

File-based sources (CSV, Parquet, Excel, SQLite, DuckDB) committed to the repo
work with no setup. For live databases, `charter.yaml` credential references
(`${VAR}`) resolve from the environment — pass them as workflow secrets:

```yaml
      - uses: datacharter/datacharter-action@v1
        env:
          PG_PASSWORD: ${{ secrets.PG_PASSWORD }}
```

Every check runs through DataCharter's read-only engine: CI can never write to
your sources.

## Links

- Main project: <https://github.com/datacharter/datacharter>
- Docs: <https://datacharter.dev> · [CLI reference](https://datacharter.dev/cli.html)
- Blog: <https://datacharter.dev/blog/>

Apache-2.0.
