# Karkain Data Studio

A CLI/TUI data exploration, transformation and visualization workspace for local, offline datasets, built exclusively on Karkain Language v1.2.0.

## Status

| Area | Status |
|---|---|
| Research | COMPLETE |
| Product design | COMPLETE |
| Repository creation | COMPLETE |
| Implementation | NOT STARTED |
| Compile/execute verification of the application | NOT STARTED (toolchain itself VERIFIED on Karkain v1.2.0) |

## What it does

Karkain Data Studio is a "data workbench" for the unglamorous, everyday work that happens before any dashboard exists: loading a CSV or line-oriented JSON export, sampling and profiling it, cleaning and reshaping it, deriving columns, inspecting distributions, and exporting a clean result. It keeps a local transactional scratchpad using the embedded `db2` database so exploration steps are reproducible and reversible. It is deliberately a workspace for data exploration and transformation, not a database administration console.

## Who it serves

- Data analysts who need to profile and clean a one-off extract without a server.
- Developers and ops engineers who want a scriptable, headless data inspection step in CI.
- Educators and students who need a zero-install, single-binary data tool.
- Offline and edge users who cannot rely on a JVM, Electron runtime or network service.

## Planned features

All items below are **(planned)**; none is implemented.

- **Data ingestion**
  - (planned) Load delimited text (CSV-like) files via line-oriented reads.
  - (planned) Ingest simple JSON records using `json_get_field` string extraction.
  - (planned) Register a named dataset from a local file path.
- **Profiling and exploration**
  - (planned) Row/column counts, null and blank tallies, per-column type inference.
  - (planned) Numeric summaries (min, max, mean, standard deviation) via `numerics`.
  - (planned) Top-N value frequency tables for categorical columns.
  - (planned) Sample and filter views over a registered dataset.
- **Transformation**
  - (planned) Select, rename, drop and derive columns through a step list.
  - (planned) Type coercion and trimming/normalization of string fields.
  - (planned) Reversible steps recorded in a session with undo to a checkpoint.
- **Visualization**
  - (planned) Terminal histograms, bar charts and sparklines rendered as text.
  - (planned) Cross-tabulation and binned distribution tables.
- **Persistence and export**
  - (planned) Materialize a dataset into the embedded `db2` scratchpad.
  - (planned) Export cleaned results to delimited text and simple JSON lines.
  - (planned) Save and reload a named exploration session.
- **Operations**
  - (planned) Config-file-driven command dispatcher and session diagnostics.

## Explicit non-goals

- Not a database administration tool (no server/role/backup management); that is Karkain Database Studio.
- Not a BI/reporting or dashboard product; that is Karkain Analytics Studio.
- No external SQL drivers, no joins, no multi-DBMS connectivity.
- No full JSON DOM/streaming; JSON support is the partial field-extraction parser only.
- No GUI, no web service, no multi-user sharing.
- No TLS, WebSocket or UDP networking.

## Karkain foundation

Exclusive foundation: Karkain Language v1.2.0. All application source will be Karkain. No other implementation language. No compiler modifications.

## Platform support status

- Windows x86-64 toolchain: VERIFIED.
- Linux cross-target output: PRESENT but execution NOT VERIFIED.
- macOS: NOT VERIFIED.

## Build and run status

No application binary exists yet. The intended generic commands are `karkain build src/main.kark -o karkain-data-studio` and `karkain run src/main.kark`. These become active when implementation starts; no compile or run of this application has been performed.

## Known blockers

- No native GUI toolkit in Karkain v1.2.0 (BLOCKED); CLI/TUI/service-first by design.
- `json` is PARTIAL (string field extraction only; no DOM/streaming/number-fidelity).
- `system`/`env`/`process`/`time` are PARTIAL shell wrappers; `get_args()` returns an empty array, so command dispatch must use a config file or verified argument path.
- No TLS/WebSocket/UDP; no concurrent HTTP server framework.
- `db2` is embedded with an SQL subset and no external drivers or joins.

## Repository

GitHub: https://github.com/ajit-ai/Karkain-Data-Studio
Issues: https://github.com/ajit-ai/Karkain-Data-Studio/issues
License: Apache-2.0.
