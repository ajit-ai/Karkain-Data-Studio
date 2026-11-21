# Karkain Data Studio — First Implementation Backlog

## First vertical slice

**Slice goal.** Prove the smallest real workflow end to end: register a delimited-text dataset, load it into the embedded `db2` scratchpad, compute basic profile counts, and print a deterministic summary as text. This exercises ingestion, storage, profiling, dispatch and diagnostics — no empty scaffolding.

**Karkain modules/files expected.**

- `src/main.kark` — config-driven command dispatch (DS-CLI).
- `src/core/dataset.kark` — dataset registration and line reader (DS-CORE, uses `io_read_lines`).
- `src/core/csv.kark` — hand-written line-oriented delimiter tokenizer.
- `src/store/scratch.kark` — `db2` create/insert/select inside `db_begin`/`db_commit` (DS-STORE).
- `src/profile/basic.kark` — row count, per-column null/blank tally (DS-PROFILE).
- `src/diag/log.kark` — run record emission (DS-DIAG).
- `configs/default.karkcfg` — dataset path, delimiter, sample size, output width.

**Exact input.** A file `examples/sample.csv` with content:

```
id,name,score
1,alice,10
2,bob,
3,carol,7
```

and a config selecting path `examples/sample.csv`, delimiter `,`, header on, profile command `profile`.

**Exact expected observable output** (stdout, deterministic):

```
dataset=sample rows=3 columns=3
column=id type=int nulls=0
column=name type=string nulls=0
column=score type=int nulls=1
status=ok
```

plus one DS-DIAG run record in the local log with command=`profile`, rows=3, status=`ok`.

**Verification command pattern (Karkain v1.2.0).** From the repository root, with the verified toolchain configuration applied:

```
karkain lint src/main.kark
karkain run src/main.kark --target native-x86_64-windows
karkain build src/main.kark -o karkain-data-studio --target native-x86_64-windows
.\karkain-data-studio
```

The built executable's stdout must match the golden output above byte-for-byte.

## Backlog items

| ID | Feature | Files/modules | Dependencies | Acceptance criteria | Tests | Complexity | Blocking Karkain capability | Definition of done |
|---|---|---|---|---|---|---|---|---|
| DS-001 | Config-driven CLI dispatch | `src/main.kark`, `configs/default.karkcfg` | none | Reads config, selects `profile`, ignores empty `get_args()` | Unit, E2E | M | `get_args()` empty; `system()` shell wrapper | Command runs from config with no argv reliance |
| DS-002 | CSV line ingestion | `src/core/csv.kark` | DS-001 | Tokenizes header+rows, honors delimiter/quote | Unit | M | No stdlib CSV parser (hand-written) | Fixture rows parsed exactly |
| DS-003 | Dataset registration | `src/core/dataset.kark` | DS-002 | Registers path/format, counts rows | Unit, integration | S | `filesystem` PARTIAL; use `io` | Named dataset resolves to file |
| DS-004 | db2 scratchpad create/insert | `src/store/scratch.kark` | DS-003 | Creates table, inserts rows in a transaction | Integration | M | `db2` SQL subset, no joins, no external drivers | Rows round-trip via SELECT |
| DS-005 | Basic profile counts | `src/profile/basic.kark` | DS-004 | Row/col counts and null tallies exact | Unit, integration | M | none | Golden profile output matches |
| DS-006 | Numeric summaries | `src/profile/stats.kark` | DS-005 | min/max/mean computed on `score` | Unit | M | float builtins byte-unreliable → use `numerics` | Stats match hand-computed values |
| DS-007 | Transform step list | `src/transform/steps.kark` | DS-004 | select/rename/derive applied in order | Unit, integration | L | none | Steps reproducible and reversible |
| DS-008 | Checkpoint rollback | `src/store/scratch.kark` | DS-007 | Fault restores last committed state | Failure/recovery | M | Crash-atomic file write PARTIAL | Rollback test passes |
| DS-009 | Reject quarantine | `src/core/csv.kark` | DS-002 | Malformed rows logged with line numbers | Unit, reliability | S | none | 50 bad rows all quarantined |
| DS-010 | Text visualization | `src/viz/charts.kark` | DS-005 | Histogram rendered as text | Unit, E2E | M | GUI BLOCKED | Deterministic ASCII output |
| DS-011 | Export delimited/JSON lines | `src/export/writer.kark` | DS-007 | Writes clean rows to file | Integration, E2E | M | JSON PARTIAL | Output re-imports correctly |
| DS-012 | Session save/load and diagnostics | `src/diag/log.kark`, `src/session.kark` | DS-005 | Session reloads; run record complete | E2E, observability | L | none | Golden run record emitted |
| DS-013 | Actual executable E2E | all | DS-001..DS-012 | Built `.exe` reproduces all goldens | Actual-compiled-executable | M | Go-engine/C-free target required | Binary E2E pass |

## Dependency order and gates

1. DS-001 → DS-002 → DS-003 → DS-004 → DS-005 (Gate 1: vertical slice runs).
2. DS-006, DS-009 (Gate 2: profiling complete).
3. DS-007 → DS-008 (Gate 3: reversible transforms).
4. DS-010, DS-011, DS-012 (Gate 4: visualization, export, sessions).
5. DS-013 (Gate 5: binary verification and packaging).

## Status

All items: **NOT STARTED**. Implementation must not begin until all designs and repository initialization are complete. No tests have been run for this application.
