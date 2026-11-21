# Karkain Data Studio — Product Specification

**Document status:** Design COMPLETE. Research = COMPLETE. Repository creation = COMPLETE. Application implementation = NOT STARTED. Application compile/execute verification = NOT STARTED (toolchain path itself VERIFIED on Karkain v1.2.0).


## A. Product definition

**Product name:** Karkain Data Studio ("Data Studio").

**Purpose and problem.** Analysts and developers repeatedly need to inspect a raw extract, understand its shape, clean it, derive a few fields and see a quick distribution — work that is too small to justify a server, a JVM or an Electron application, yet too fiddly to do reliably by hand. Existing tools are desktop-GUI-first and heavyweight for this task.

**Vision.** A small, single-binary, headless data workbench in which a local dataset can be registered, profiled, transformed and visualized in text, with every step reproducible from the embedded transactional `db2` scratchpad.

**Intended users / segments.** Data analysts doing ad-hoc exploration; developers and ops engineers embedding data inspection in CI; educators and students; offline and edge users without a runtime stack.

**Usage scenarios.** (1) Profile a CSV export and report column types and null counts. (2) Clean and reshape a dataset through a short, reversible step list. (3) Render a numeric distribution as a terminal histogram. (4) Export a cleaned dataset to delimited text or JSON lines for downstream use.

**Market positioning.** A zero-runtime, CLI/TUI complement to GUI SQL clients — not a replacement. It occupies the headless/embedded niche that the researched incumbents leave underserved.

**Primary value proposition.** Fast, scriptable, offline data exploration with a local transactional scratchpad, shipped as one native Karkain binary.

**Explicit non-goals / boundaries.** No database administration; no BI dashboards (that is Karkain Analytics Studio); no external DBMS drivers or joins; no full JSON DOM; no GUI or web/multi-user service; no TLS/WebSocket/UDP. These boundaries are intentional and align with Karkain v1.2.0 capability limits.

## B. Functional architecture

**Module inventory.**

| Module | Responsibility |
|---|---|
| DS-CORE | Dataset registration; delimited-text and line-JSON readers (`io_read_lines`, `json_get_field`). |
| DS-STORE | `db2` scratchpad: create/insert/select/update/delete over scratch tables. |
| DS-PROFILE | Row/column counts, type inference, null/blank tallies, numeric summaries, frequency tables. |
| DS-TRANSFORM | Ordered, reversible transformation steps (select, rename, drop, derive, coerce, trim). |
| DS-VIZ | Terminal visualization: text histograms, bars, sparklines, cross-tabs. |
| DS-EXPORT | Delimited-text and JSON-lines writers; session save/load. |
| DS-CLI | Config-file-driven command dispatch and argument handling. |
| DS-DIAG | Structured logging, error codes and run diagnostics. |

**Core workflows.** Register dataset → profile → (transform → preview)* → visualize → export/save. Transformation steps write their result into the scratchpad so each preview is a materialized, inspectable state.

**Admin workflows.** Local configuration of default delimiters, sample sizes, output width and scratchpad location; session listing and cleanup. There is no multi-user administration model.

**Error/recovery workflows.** Malformed input rows are quarantined to a reject list with line numbers; a failed transform restores the scratchpad from the last committed checkpoint via `db_rollback`; export failures leave the previous output untouched.

**Roles & permissions.** Single-user, process-local. No authentication/authorization; file-system permissions are the only access boundary.

**Configuration/customization.** A local config file sets delimiters, encoding assumptions, sample size, terminal width and chart style; no machine-specific paths are stored in repository docs.

**Import/export.** Import: delimited text with configurable separator/quote, and JSON lines via partial field extraction. Export: delimited text and JSON lines. All formats are line-oriented to suit the available I/O and JSON primitives.

**Integration requirements.** Reads local files through `io`; may be invoked by CI as a compiled binary; no network integration is required in MVP.

## C. Competitor analysis

Researched competitors are **DBeaver**, **JetBrains DataGrip** and **Beekeeper Studio**, consistent with `docs/competitor-analysis/COMPETITORS.md` and portfolio research `domain2-data.md`. Claim labels below match that research: **[VERIFIED]** official-page fact (accessed 2026-11-21), **[INTERPRETATION]** synthesis, **[PROPOSAL]** Karkain recommendation, **UNKNOWN — not verified**.

**[VERIFIED]** DBeaver is a universal database tool with Community and commercial editions and broad driver coverage; DataGrip is a cross-platform, JVM-based database IDE "free for non-commercial use"; Beekeeper Studio is an open-source SQL editor/database manager emphasizing lightweight UX. **[INTERPRETATION]** They span maximal breadth, maximal IDE depth and minimal-friction usability, but all are desktop-GUI-first.

**Feature comparison matrix (Karkain = proposed).**

| Capability | Karkain Data Studio (proposed) | DBeaver | DataGrip | Beekeeper |
|---|---|---|---|---|
| GUI SQL IDE | Blocked; CLI/TUI only | Yes | Yes | Yes |
| Many external DBMS | No (constraint) | 80+/CData | 50+ | Multiple |
| Local transactional scratchpad | Yes (`db2`) | No | No | No |
| Zero-runtime single binary | Yes | No (JVM) | No (JVM) | No (Electron) |
| Full JSON DOM | No (partial) | Via drivers | Yes | JSON viewer |
| Multi-user web sharing | No | CloudBeaver/Team | No | No |

**Platform/deployment.** **[VERIFIED]** All three incumbents ship desktop builds for Windows/macOS/Linux (DBeaver also web/CloudBeaver; DataGrip JVM; Beekeeper Electron with portable build). Karkain Data Studio: one native Windows binary (VERIFIED toolchain), with Linux/macOS build targets present but execution unverified.

**Licensing.** **[VERIFIED]** DBeaver Community is open source; official edition pages list Lite $113/user/yr, Enterprise $255/user/yr, Ultimate $510/user/yr, CloudBeaver $1,025/yr (accessed 2026-11-21). DataGrip commercial price **UNKNOWN — not verified**; Beekeeper paid price **UNKNOWN — not verified**. No price is invented. Karkain Data Studio is Apache-2.0.

**Strengths/limitations.** **[VERIFIED/INTERPRETATION]** Incumbents have enormous connectivity and mature SQL ergonomics; they are perceived as "heavy" and are GUI-bound. Karkain offers zero-runtime offline operation but cannot match breadth.

**Market gaps / differentiation.** **[INTERPRETATION]** Headless, embedded, CI-friendly data workbenches are underserved. **[PROPOSAL]** Differentiate on single-binary offline operation, reproducible `db2` scratchpad workflows and text visualization — complementarity, not parity.

**Features deliberately not copied.** External SQL drivers/joins, GUI editors, CloudBeaver-style multi-user sharing, AI assistants, full JSON DOM.

## D. Technical architecture

**Application architecture.** A command dispatcher (DS-CLI) reads a config file, selects a workflow and drives DS-CORE/DS-STORE/DS-PROFILE/DS-TRANSFORM/DS-VIZ/DS-EXPORT in-process; DS-DIAG observes all stages.

**Component boundaries.** Ingestion is separated from storage so formats can be added without touching profiling; transformation is a pure ordered step list applied to scratch tables; visualization reads materialized results only.

**Process/service boundaries.** Single process, no daemon in MVP. A future read-only service mode is only a thin wrapper over the same modules using `http` building blocks.

**UI architecture.** The UI is CLI/TUI/service-first because Karkain v1.2.0 has no native GUI toolkit (BLOCKED); any GUI is a planned layer contingent on ecosystem capability and is not claimed. Output is deterministic text (tables, histograms, sparklines).

**Domain model.** Dataset, ScratchTable, ColumnProfile, TransformStep, VisualizationSpec, Session — see section E.

**Data storage strategy.** Source files remain untouched; derived state lives in an embedded `db2` scratchpad file plus a line-oriented append-only session log of steps.

**API/interface architecture.** Internal Karkain module APIs; external interface is the command/config surface and CLI text output. No REST API in MVP.

**Event/message architecture.** No event bus; cooperative `async` channels may sequence transform stages in later releases, but MVP is synchronous and deterministic.

**Logging/diagnostics/observability.** DS-DIAG emits structured run records (command, dataset, step count, rows processed, status, elapsed time) to a local log, with `karkain lint`/`check --format=json` used at build time.

**Configuration management.** Versioned local config file with defaults; no environment-specific secrets.

**Plugin/extension architecture.** A registry of reader/transform/renderer functions keyed by name; adding a function is a Karkain module change, not a new language.

## E. Data design

**Main entities.** `Dataset` (name, source path, format, row count); `ScratchTable` (session-local `db2` table); `ColumnProfile` (name, inferred type, null count, distinct count, numeric stats); `TransformStep` (ordinal, operation, parameters); `VisualizationSpec` (chart type, source column, bin count); `Session` (named list of steps + snapshot pointer); `RejectRecord` (line number, raw text, reason).

**Relationships.** A Session owns an ordered set of TransformSteps and one or more ScratchTables; a ColumnProfile belongs to a Dataset or ScratchTable; a VisualizationSpec references a ScratchTable column; RejectRecords belong to an ingestion run.

**Ownership and validation.** All scratch state is owned by a single session/process. Validation: identifier/field names checked before `db2` DDL; type inference applied before coercion; numeric parsing rejects non-numeric tokens without aborting the run.

**Lifecycle and persistence.** Sessions are created, mutated, saved and reopened; scratch tables are dropped on session close unless exported. Source data is read-only.

**Transactions, backup/restore.** Profiling and transforms run inside `db_begin`/`db_commit`; on error `db_rollback` restores the last checkpoint. "Backup" is a saved session + scratchpad file copy; "restore" reopens it.

**Migration.** No schema format exists before v1.2.0; session/config version fields allow future migrations.

**Import/export formats.** Delimited text (configurable separator/quote) and JSON lines (flat fields via `json_get_field`). Nested JSON is out of scope.

**Audit history.** The append-only session log records each step with input/output row counts, providing an audit trail of exploration.

**Storage decision.** A database is appropriate only as a local scratchpad: embedded `db2` gives transactional, reproducible intermediate state. Raw data stays in files; the session log is append-only for auditability.

## F. Karkain implementation architecture

**Required Karkain capabilities / compiler.** Karkain v1.2.0 only; front-end lint and Go-engine C-free native Windows build verified. No compiler modifications.

**stdlib actually present (per capability register).** `io` (PRESENT: `io_read_lines`, `io_write_file`, `io_append_file`), `db` (PRESENT/PARTIAL: `db2` SQL subset, `db_begin/commit/rollback`, prepared statements; no joins/external drivers), `numerics` (PRESENT: deterministic numeric primitives), `collections` (PRESENT), `crypto`/`encoding` (PRESENT: hashing/encoding for export integrity), `async` (PRESENT: cooperative channels, optional), `json` (PARTIAL: `json_get_field` only), `system`/`env`/`process`/`time` (PARTIAL shell wrappers), `filesystem` (PARTIAL, PowerShell-backed).

**Required ecosystem components.** None beyond the v1.2.0 stdlib and compiler; optional `karkain test`/`lint`/`ide info` tooling.

**Missing dependencies.** No full JSON DOM/streaming; no CSV parser stdlib module (line-oriented parsing must be hand-written in Karkain); no GUI toolkit; no portable process/time APIs.

**Native implementation opportunities.** A small line-oriented CSV tokenizer, deterministic numeric summaries in `numerics`, and a text chart renderer can all be pure Karkain.

**FFI/OS integration needs.** None required for MVP. File I/O uses `io` rather than OS shell-outs; no FFI is justified, so none is proposed.

**Build/packaging.** `karkain build src/main.kark -o karkain-data-studio` with the verified Go-engine/C-free native Windows target; single-binary distribution.

**Explicit blockers (recorded, not faked).** GUI BLOCKED; full JSON DOM PARTIAL; no external SQL drivers/joins; `get_args()` returns an empty array and `system()` returns an exit code rather than captured stdout on Windows, so config-file-driven dispatch is mandatory until probed; TLS/WebSocket/UDP BLOCKED. Each is a language/ecosystem concern; no alternate language may be introduced as a workaround.

## G. Non-functional requirements

Targets use workload + measurement method (practical machine ~4 GB RAM); all are PLANNED acceptance goals, not measured results.

| Area | Target (workload → method) |
|---|---|
| Performance | Profile a 100k-row, 12-column CSV → time from invocation to printed profile; report p50 of 5 runs. |
| Latency | Transform preview of 10k rows → elapsed time to rendered table. |
| Memory | Same 100k-row workload → peak RSS sampled from OS tool during run; must stay within the 4 GB budget. |
| CPU | Profile workload → average CPU% over run via OS tool. |
| Scalability | Row counts 1k/10k/100k → near-linear time growth recorded. |
| Reliability | 50 inject malformed rows → all quarantined with correct line numbers, session consistent. |
| Security | No network surface; file paths confined to config-declared directories (verified by review). |
| Privacy | All processing local; no telemetry (verified by log inspection). |
| Portability | Compile on Windows; produce Linux/macOS cross-target artifacts (execution unverified). |
| Maintainability | Each module independently lint-clean (`karkain lint`). |
| Accessibility | Text-only, deterministic output usable with screen readers and no color dependency. |
| Observability | Every run emits a complete DS-DIAG record. |
| Recovery | After mid-transform fault, `db_rollback` restores last checkpoint (fault-injection test). |

## H. Compatibility and portability

**Supported OS.** Windows toolchain VERIFIED; Linux and macOS NOT VERIFIED (cross-target output PRESENT, execution unverified). **Architectures.** x86-64 verified; other targets present but unverified. **Karkain v1.2.0 compatibility matrix.** Builds only with v1.2.0; no support for 1.1.0 or earlier. **Binary/package formats.** Native Windows `.exe`; source distribution with config file. **Install/uninstall.** Copy binary and config; uninstall by removal (no shared state). **Config/data migration.** Versioned fields; no pre-v1.2.0 backward compatibility required. **Upgrade/rollback.** Replace binary; sessions remain readable if config version is unchanged. **Cross-platform differences.** PowerShell-backed `filesystem` calls may behave differently; use `io` to minimize. **Unsupported features.** GUI, full JSON DOM, external SQL, TLS/WebSocket/UDP, multi-user sharing.

## I. Testing strategy

All tests are **PLANNED**; no tests have been run for this application.

- **Unit:** CSV tokenizer, type inference, numeric summaries → exact expected values on fixtures.
- **Integration:** ingest→profile→transform→export → expected output file and row counts.
- **E2E:** config-driven CLI runs produce deterministic text → golden-output comparison.
- **Platform-specific:** Windows native binary execution; cross-target artifact creation.
- **Security:** path-confinement review; no network calls observed.
- **Performance:** workload timings per section G recorded, not asserted.
- **Regression:** golden outputs frozen after each milestone.
- **Compatibility:** lint/build with v1.2.0 only.
- **Actual-compiled-executable:** run the built `.exe`, not source, for E2E.
- **Failure/recovery:** malformed rows and injected faults → quarantine and rollback verified.

## J. Implementation roadmap

**MVP.** DS-CORE ingestion (CSV), DS-STORE scratchpad, DS-PROFILE basic counts, DS-CLI config dispatch, DS-DIAG logging.
**Core functional release.** DS-TRANSFORM step list with checkpoints, JSON-lines ingestion, DS-EXPORT, session save/load.
**Advanced release.** DS-VIZ text charts, frequency/cross-tab views, optional cooperative stage sequencing.
**Production-readiness release.** Hardened validation, recovery tests, cross-target build artifacts, documentation.

**Feature dependencies.** Profiling depends on ingestion; transforms depend on scratchpad transactions; visualization depends on materialized transforms; export depends on a committed session.

**Milestones/gates.** Gate 1: compiled binary profiles a CSV. Gate 2: reversible transforms with rollback. Gate 3: text visualization and export. Gate 4: full test pass and packaging. No dates are assigned.

**Risks/mitigations.** `get_args()` empty → config-file dispatch (mitigated). Partial JSON → restrict to flat records (mitigated). No GUI → CLI/TUI by design (accepted). No external SQL → scratchpad scope (accepted).
