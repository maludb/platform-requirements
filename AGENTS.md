# AGENTS.md

Project instructions for Codex working in this repository.

## Project

**MaluDB** — a memory DBMS for long-term institutional memory, human-AI knowledge sharing, and contextual recall. Built in **C** as PostgreSQL extensions on **Ubuntu 24.04 LTS**, with **PostgreSQL 17** (PGDG) as the foundation.

### Authoritative documents

1. [`white-paper.md`](white-paper.md) — the conceptual reference. **Do not modify** without explicit instruction; it is a research-grade specification.
2. [`requirements.md`](requirements.md) — the implementation requirements. This is what the system must satisfy. Update this when scope changes; keep it in sync with the white paper.

If this repository contains code that conflicts with `requirements.md`, raise the conflict — do not silently change one to match the other.

### Status

Planning / specification phase. No implementation code yet.

### Staging strategy

The project builds out in stages. Stay inside the current stage's scope unless the user explicitly expands it.

- **Stage 1 (current): Memory Core framework on PostgreSQL + pgvector.** Goal is a working database that can store relational data with vector columns, with the dev environment, extension skeleton, packaging, and CI all in place. No memory-specific object model yet — that comes in Stage 2.
- **Stage 2: Memory object model.** Sources, claims, facts, Episode Objects, memories, Memory Detail Objects, relationship edges. Internal layout of memories, documents, and verbatim source storage.
- **Stage 3+: Bitemporal, SVPOR, derivation ledger, retrieval planner, workflow extraction, skills, active memory pools, local nodes.** Expanded per `requirements.md` §9.

When the user gives a task, ask which stage it belongs to if it's not obvious. Do not implement Stage 2+ features in Stage 1 even if the requirements doc calls for them.

## Stack

| Layer | Choice |
|---|---|
| OS | Ubuntu 24.04 LTS, x86_64 + arm64 |
| Database | PostgreSQL 17 from PGDG (`apt.postgresql.org`) |
| Language | C, C11 standard |
| In-DB | SQL, PL/pgSQL |
| Build | PGXS Makefile (NOT Meson, NOT CMake) |
| Vector | pgvector (HNSW) |
| Graph | Apache AGE — only when Cypher is needed; prefer recursive CTEs for moderate graphs |
| Temporal | Native `tstzrange` + `EXCLUDE USING gist`; optionally the `periods` extension |
| FTS | Native `tsvector` + GIN; `pg_trgm` for fuzzy match |
| Audit | `pgaudit` + `pg_stat_statements` |
| Partitioning | Native declarative + `pg_partman` |
| License | **PostgreSQL License** (BSD-style) for our code |

### Avoid

- `pg_embedding` (Neon) — archived.
- `temporal_tables` extension — unmaintained since ~2017.
- TimescaleDB **TSL features** (compression, continuous aggregates with refresh policies, etc.) — license incompatibility. Apache-2.0 core only would be acceptable but offers little we don't get from `pg_partman`.
- ZomboDB — heavy operational cost (separate ES cluster).
- GPL-licensed dependencies linked into our extension `.so` — incompatible with PostgreSQL License redistribution.
- Native SQL:2011 system-versioned tables — not yet in PostgreSQL.
- Meson for extension builds — experimental; not adopted in v1.

### License watchlist

TimescaleDB TSL, Citus AGPL, ParadeDB AGPL. If adopting any of these, surface the license question to the user first.

## Development environment

Setup on Ubuntu 24.04:

```bash
# PGDG repo for PostgreSQL 17
sudo install -d /usr/share/postgresql-common/pgdg
sudo curl -fsSL https://www.postgresql.org/media/keys/ACCC4CF8.asc \
  -o /usr/share/postgresql-common/pgdg/apt.postgresql.org.asc
echo "deb [signed-by=/usr/share/postgresql-common/pgdg/apt.postgresql.org.asc] \
  https://apt.postgresql.org/pub/repos/apt noble-pgdg main" | \
  sudo tee /etc/apt/sources.list.d/pgdg.list
sudo apt-get update
sudo apt-get install -y \
  postgresql-17 postgresql-server-dev-17 postgresql-server-dev-all \
  postgresql-17-pgvector postgresql-17-pgaudit postgresql-17-partman \
  build-essential clang-18 llvm-18-dev pkg-config bison flex \
  libicu-dev libssl-dev libreadline-dev zlib1g-dev liblz4-dev libzstd-dev \
  libxml2-dev
```

When working on this project, the current PostgreSQL major and binary path are:

```bash
PG_CONFIG=/usr/lib/postgresql/17/bin/pg_config
```

## Build / test / install

Once an extension exists (Phase 0+), the conventional commands are:

```bash
# Build
make PG_CONFIG=/usr/lib/postgresql/17/bin/pg_config

# Install (requires sudo for system PG)
sudo make install PG_CONFIG=/usr/lib/postgresql/17/bin/pg_config

# Regression tests
sudo --preserve-env make installcheck \
  PG_CONFIG=/usr/lib/postgresql/17/bin/pg_config

# Sanitizer build (single-version)
make COPT="-fsanitize=address,undefined -fno-omit-frame-pointer -O1 -g" \
  PG_CONFIG=/usr/lib/postgresql/17/bin/pg_config

# Static analysis
scan-build-18 make PG_CONFIG=...
clang-tidy-18 src/*.c -- $(pg_config --cflags) -I$(pg_config --includedir-server)
```

When sanitizer-enabled, run with:

```bash
ASAN_OPTIONS=detect_leaks=0:abort_on_error=1:print_stacktrace=1:strict_string_checks=1:check_initialization_order=1
UBSAN_OPTIONS=print_stacktrace=1:halt_on_error=1
```

## Coding conventions

### Memory and error handling (PostgreSQL backend code)

- **Never** use `malloc`/`free` for backend data. Use `palloc`, `palloc0`, `repalloc`, `pfree` — they are bound to the current `MemoryContext`.
- Long-lived state goes in `TopMemoryContext` or a child anchored there; create explicit contexts with `AllocSetContextCreate` and switch with `MemoryContextSwitchTo`.
- For SPI: tuples returned by SPI live in `SPI_palloc`'d memory; copy out (`SPI_palloc`, `heap_copytuple`) before `SPI_finish`.
- Errors: `ereport(ERROR, (errcode(...), errmsg(...), errdetail(...), errhint(...)))`. `ERROR` longjmps out of the call stack — design for it. Use `PG_TRY/PG_CATCH/PG_END_TRY` only when you must, and always re-throw unless intentionally absorbing.
- Background workers register in `_PG_init` via `RegisterBackgroundWorker` (static) or `RegisterDynamicBackgroundWorker`. Set `bgw_notify_pid` for waitable workers.
- Shared memory + LWLocks (PG 15+): in `_PG_init` install `shmem_request_hook`; from inside it call `RequestAddinShmemSpace(size)` and `RequestNamedLWLockTranche(name, n)`. Then install `shmem_startup_hook` for the actual `ShmemInitStruct`. Module must appear in `shared_preload_libraries`. Calling `RequestAddinShmemSpace` outside the hook errors out — this is the most common port bug.

### Multi-PG-version support

Target PG 16, 17, 18. Use feature macros:

```c
#include "pg_config.h"
#if PG_VERSION_NUM >= 170000
  /* PG 17+ specific code */
#endif
```

### Atomic multi-model writes

Per `requirements.md` §3, a single logical operation may touch object metadata, source links, graph relationships, temporal windows, FTS, vector indexes, workflow records, and audit logs. **Partial commits are forbidden.** Wrap multi-model writes in a single SQL transaction (PostgreSQL's WAL/MVCC handles atomicity in v1). Document any operation that cannot be wrapped this way and surface the design tradeoff.

### Bitemporal discipline

Corrections must **never** silently overwrite history. They close a `valid_time_end`, open a new version, and create a `supersedes` edge. The Temporal Supersession Engine owns this — application code should call its API rather than mutating temporal columns directly.

### SVPOR everywhere

When embedding a memory chunk, source excerpt, workflow trace, or summary, the **subject, verb, predicate, and object frame must be incorporated into the embedded text**. Query-time retrieval applies the same frame. This is not optional — it is part of the retrieval contract in `requirements.md` §3.2 and §4.

### Authorization-aware retrieval

Authorization is checked at three points: planning, candidate expansion, and result assembly. **Never** apply authorization only to the final answer — vector similarity, graph traversal, summaries, and active-pool loading can leak otherwise.

### Provenance is mandatory

Every derived object must have a Derivation Ledger entry recording its parser/model/prompt/policy/verifier and inputs. Derivations without ledger entries are bugs.

### Comments

Default to no comments. Add one only when the *why* is non-obvious — a hidden constraint, a workaround for a specific PG quirk, an invariant that would surprise a reader. Don't restate what the code does. Don't reference task tracking or callers.

## Testing

- **pg_regress** for SQL-level tests (`sql/` + `expected/`, listed in `REGRESS`).
- **isolation tester** (`pg_isolation_regress`) for concurrency / locking / MVCC behavior — listed in `ISOLATION`.
- **TAP tests** (`TAP_TESTS=1`) for full-cluster end-to-end, recovery, replication, GUC matrices.
- **C unit tests**: PG has no in-tree unit framework. Prefer exposing internals via `CREATE FUNCTION` test shims and exercising from pg_regress. Embed Check or Unity only for pure leaf functions built as a separate binary linked against a static archive.
- Coverage via `--enable-coverage` on PG, `lcov`/`gcov`.

## CI

GitHub Actions matrix on `ubuntu-24.04`, PostgreSQL versions 16, 17, 18 (when available on PGDG). Separate jobs for: ASan+UBSan (single PG version, debug build), `clang-tidy`, `scan-build`, optional Coverity nightly. PGDG repo install pattern is as shown in the dev-environment section above.

## Repository conventions

- License: **PostgreSQL License** (BSD-style, matches PG core).
- Contributor sign-off: DCO required.
- Vet third-party libs: jemalloc=BSD-2 ✓, mimalloc=MIT ✓, RocksDB=Apache-2/GPL-2 dual (use Apache-2) ✓. No GPL-only deps statically linked.
- Commit messages: short imperative subject, body explains the *why*. Reference `requirements.md` section numbers when implementing a specific requirement.
- Branch naming: `phase-N/<topic>` for roadmap work; `fix/<topic>` for fixes; `spike/<topic>` for exploration.

## What "done" looks like for a task

Before reporting any feature complete:

1. The change implements the relevant `requirements.md` sections — verify by referencing them.
2. pg_regress tests pass on PG 17 default build.
3. ASan+UBSan build of the same tests passes — no leaks (or documented suppressions), no UB.
4. `clang-tidy` and `scan-build` produce no new warnings.
5. The change preserves provenance for every derived object it produces.
6. Multi-model writes inside the change are atomic — no partial states observable.
7. Authorization is checked at planning, expansion, and assembly for any retrieval path touched.
8. Bitemporal corrections, if any, go through the Temporal Supersession Engine.

If any of these can't be satisfied yet (because the dependency hasn't been built), say so explicitly. Don't fake it.

## Working with the user

- The user is the project owner / lead author of the white paper. Treat the white paper as a research artifact: faithful to its conceptual model, but the implementation is allowed to choose pragmatic shortcuts in v1 — call them out.
- When a requirement in the white paper is ambiguous or expensive to implement literally, propose a v1 simplification with the tradeoff explicit, and ask before committing.
- Don't introduce dependencies (especially copyleft ones) without confirmation.
- For any change that affects `requirements.md`, update both that doc and any implementation in the same change. Don't let them drift.
- Exploratory questions get 2–3 sentences with the main tradeoff, not a plan.

## Things in flux to watch

- PostgreSQL 18 GA (Sept 2025): introduces `extension_control_path` and OAuth — bumping `min_version` to 18 unlocks both, decision deferred.
- Apache AGE on PG 18 — track readiness.
- Meson-as-extension-build — experimental upstream; do **not** adopt yet.
- LLVM JIT default on Ubuntu 24.04 is 18; PGDG packages may move forward — pin via `llvm-config` if needed.
- SQL/PGQ (SQL:2023 property graph queries) — not in vanilla PG yet; do not depend on it.
- Native SQL:2011 system-versioned tables — patches circulating, no commit. Don't depend on them.
