# MaluDB — Requirements

This document is the authoritative requirements specification for **MaluDB**, the memory DBMS proposed in [`white-paper.md`](white-paper.md). The white paper is the conceptual reference; this document is what the implementation must satisfy.

MaluDB is a **database management system for long-term institutional memory, human-AI knowledge sharing, and contextual recall**. It treats memories as first-class data objects governed by a single coherent DBMS rather than glued together from polyglot stores. The initial implementation language is **C**. The base platform is **PostgreSQL 17** on **Ubuntu 24.04 LTS**. The system extends PostgreSQL through C extensions, in-database SQL/PL/pgSQL objects, and external services where appropriate; it does not fork PostgreSQL in v1.

The default operator experience is an integrated MaluDB installation: installing MaluDB installs, configures, and manages the required PostgreSQL base, PostgreSQL extensions, MaluDB extension objects, and MaluDB services together. PostgreSQL remains the upstream PGDG PostgreSQL package foundation, not a forked or private embedded storage engine.

---

## 1. Scope

### 1.1 In scope (v1)

In this document, **v1** means the complete first product arc delivered across the staged roadmap in §9. **Stage 1** is intentionally narrower: it establishes the PostgreSQL + pgvector + extension substrate and does not implement the memory object model or later memory semantics.

- A governed object model for **source packages, claims, facts, Episode Objects, memories, Memory Detail Objects, workflow traces, generalized workflows, procedural memories, skills, and relationships**.
- **Bitemporal** valid-time and transaction-time semantics with explicit supersession, contradiction, and staleness state.
- A **derivation ledger** that traces every derived object back to its source evidence and the model/parser/policy/human action that produced it.
- A **verbatim source archive** that preserves raw inputs (documents, tickets, transcripts, logs, API payloads) when policy and retention permit.
- **Confidence and precision** stored as separate MAUT-weighted aggregates with their per-category breakdowns, weights, and evaluator versions.
- A **Subject-Verb-Predicate-Object-Relationship** organization layer that participates in indexing, retrieval planning, and embedding inputs.
- **Recursive Memory Detail Objects** addressable independently of their parent memory.
- **Hybrid retrieval** spanning relational/catalog filters, full-text search, graph traversal, temporal constraints, vector similarity, and recursive detail expansion, planned as a **search path** rather than a fixed pipeline.
- An **authorization model** that scopes grants to subjects, verbs, topics, projects, partitions, source types, workflows, and active memory pools — not only tables and collections.
- **Active Memory Pools**: scoped, low-latency working sets shared by humans and AI agents during a task.
- **Local Memory Nodes**: offline/edge subsets that synchronize back to the Enterprise Memory Core under governance.
- A **Workflow Extraction Engine** that proposes candidate workflow traces and generalized workflows from Episode Objects and source evidence, reviewable rather than auto-promoted.
- A **Skill Runtime** that executes governed skill packages as state machines with audit records.
- A **Model Registry** for embedding models, extraction models, rerankers, summarizers, with version-aware migration support.
- **Replay semantics**: reconstruct the historical, current-valid, transaction-time, or full bitemporal view of an Episode Object under the requesting account's privileges.
- An integrated installation and service layout that presents MaluDB as one managed system while using upstream PostgreSQL as the authoritative database foundation.
- Driver/API access for at minimum **C, Python, Node.js, and PHP** clients, plus an MCP-style integration surface for AI agents.

### 1.2 Out of scope (v1, may be added later)

- Active-active replication, distributed consensus, multi-master clustering, cross-region failover.
- A causal-inference subsystem (the system stores typed causal edges with metadata; it does not yet derive causal chains, evaluate interventions, or answer counterfactuals automatically).
- A bespoke storage engine. v1 builds on PostgreSQL's existing storage; a custom WAL/MVCC engine is a deferred decision.
- Automatic promotion of any model output to fact or memory without provenance, evidence linkage, and policy-controlled review.
- Treating any AI-generated output as authoritative truth.

### 1.3 Non-goals

- Replacing authoritative source systems (email, ticketing, source control, observability, transactional DBs). Those remain systems of record; MaluDB preserves, links, interprets, and recalls knowledge derived from them.
- Storing every transient message or intermediate thought without curation, validation, and lifecycle policy.

---

## 2. Target Platform

| Item | Target |
|---|---|
| OS | Ubuntu 24.04 LTS (Noble Numbat), x86_64 and arm64 |
| Database | PostgreSQL 17 from the **PGDG apt repository** (`apt.postgresql.org`), installed and configured by the default MaluDB installation bundle |
| Language (server-side extensions) | C, **C11** standard |
| Language (in-DB stored procedures) | SQL, PL/pgSQL; PL/Python sandboxed where required |
| Compiler | gcc 13.x default; Clang 18 for sanitizer/static-analysis builds |
| Build system | **PGXS Makefile** for extensions; `pg_buildext` / `dh_make_pgxs` for Debian packaging |
| LLVM JIT | LLVM 18 (Ubuntu 24.04 default) |
| License | **PostgreSQL License** (BSD-style) for own code; permissive deps only |

PG 16 must build cleanly through the same matrix to support migration; PG 18 is a target as soon as it lands in PGDG for `noble`.

The default installation MUST NOT require operators to provision PostgreSQL manually before installing MaluDB. MaluDB packages, containers, or deployment bundles MUST declare, install, configure, and validate the required PostgreSQL base and PostgreSQL extensions as part of the MaluDB install path. An advanced installation mode MAY target an existing compatible PostgreSQL cluster when the operator explicitly chooses that mode.

---

## 3. Functional Requirements

### 3.1 Memory Object Model

The DBMS MUST provide first-class object types with stable identifiers, version history, lifecycle state, and access-control metadata for each of the following:

| Object | Purpose |
|---|---|
| **Source Package** | Verbatim raw input (document, transcript, ticket, log, API payload) with timestamp, hash, retention class, and origin. |
| **Claim** | An assertion extracted from a source. May be unverified, contradicted, partially true, or imprecise. Always linked to one or more source references with offsets. |
| **Fact** | A claim, or set of claims, accepted as true within a defined scope according to verification rules. |
| **Episode Object** | The concrete DBMS representation of a specific remembered episode (installation, outage, decision, migration, etc.). Binds source evidence, structure, time, relationships, security labels, and lifecycle state. |
| **Memory** | A contextual record of an event, decision, discovery, lesson, dependency, or change. May aggregate multiple facts, claims, and Episode Objects. |
| **Memory Detail Object** | Addressable child or linked object representing a step, substep, parameter, command, validation, exception, source excerpt, or evidence item. Recursively containable. |
| **Workflow Trace** | The observed sequence of steps for one Episode Object/case. |
| **Generalized Workflow** | A repeatable process pattern derived from one or more traces. |
| **Procedural Memory Object** | Capability-oriented how-to knowledge for performing, adapting, validating, and repairing work. Distinct from a Workflow. |
| **Skill Package** | Governed, evidence-backed procedural memory packaged for reuse with execution policy and audit records. |
| **Relationship Edge** | Typed edge between any two governed objects (e.g., `supports`, `contradicts`, `supersedes`, `derived_from`, `verified_by`, `caused_by`, `depends_on`, `part_of`, `related_to`, `before`, `after`, `inside`, `with`, `from`, `has_detail`, `contains`). |

Every object MUST carry: stable ID, version, lifecycle state (`current`, `historical`, `stale`, `superseded`, `contradicted`, `consolidated`, `decayed`, `archived`, `retired`), partition membership, security labels, derivation ledger reference, and bitemporal fields (see §3.4).

### 3.2 Subject-Verb-Predicate-Object-Relationship (SVPOR) Organization

The DBMS MUST organize memory objects under a grammar-inspired semantic frame:

- **Subject** — primary entity (person, project, system, organization, customer, place, AI agent, document, application).
- **Verb** — action class or state (`installed`, `decided`, `discovered`, `failed`, `approved`, `migrated`, `learned`, `verified`, …).
- **Predicate** — structured semantic frame (purpose, rationale, outcome, actor, role, reason, environment, event_date, …).
- **Object** — the thing acted on or remembered. May carry a payload (summary + structured fields + embeddings) and may itself be another entity.
- **Relationship** — typed graph edge between objects (broader than English prepositions; see §3.1).

The SVPOR frame MUST be:

1. **An organizing/routing index**: the retrieval planner uses it to compartmentalize search before more expensive operations.
2. **A grant target**: privileges may be granted on subjects, verbs, topics, predicates, and relationship types.
3. **An embedding input**: when a memory chunk, source excerpt, workflow trace, or summary is embedded, the relevant subject, verb, predicate, and object frame MUST be incorporated into the embedded text. Query-time retrieval MUST use the same frame when extractable from the prompt or hints.

### 3.3 Confidence and Precision (MAUT-based)

The DBMS MUST model **confidence** and **precision** as **separate dimensions**, each computed via **Multi-Attribute Utility Theory (MAUT)** weighted aggregates with normalized category scores in `[0,1]` and weights summing to 1.

**Fact confidence** categories: extraction, source, evidence, truth, verification.
**Fact precision** categories: temporal, entity, relationship, location, semantic, scope.
**Memory confidence** categories: supporting facts, claim consistency, source diversity, inference, temporal coherence, contradiction status, staleness status.
**Memory precision** dimensions: temporal, actor, entity, causal, decision, procedural, contextual.
**Workflow confidence** factors: supporting memory count, outcome quality, source diversity, step consistency, evidence strength, contradiction status, current validity.
**Procedural-memory confidence** MUST NOT be copied from its supporting workflows; it MUST be evaluated against supporting outcomes, operational diversity, exception coverage, validation checks, and current applicability.

For every score the DBMS MUST persist, queryable at read time:
- The aggregate score.
- The per-category subscores.
- The weights used.
- The evaluator (model version, prompt template, policy version, or human reviewer ID) responsible for each subscore.
- The supporting evidence references used in the computation.

Weights MUST be policy-configurable by schema, partition, source class, fact type, memory type, and workflow family.

### 3.4 Bitemporal Model

Every governed memory object MUST carry the following temporal fields:

- `event_time` — when the remembered event occurred, if known.
- `valid_time_start` / `valid_time_end` — the period in which the assertion applies to the represented world.
- `transaction_time_start` / `transaction_time_end` — when the DBMS recorded, accepted, revised, or retired this version.
- `source_time` — timestamp supplied by the original source.
- `verification_time` — when the system, policy, or human verifier accepted/rejected/revised the object.
- `stale_after` — time or condition after which the object MUST be reviewed before operational use.

Retrieval MUST expose **explicit query modes**: `current_valid`, `historical_valid`, `as_of_transaction_time`, and `bitemporal_audit`. The default for end-user queries is `current_valid`.

Corrections MUST NOT overwrite history. The **Temporal Supersession Engine** closes prior valid-time windows, creates supersession edges, preserves the prior transaction-time record, updates staleness, and notifies derived artifacts (embeddings, workflow summaries, active pools, skill packages) that they may need review or rebuild.

Implementation guidance: `tstzrange` columns + `EXCLUDE USING gist (... WITH &&)` for non-overlap, optionally layered with the `periods` extension for ergonomics. Native SQL:2011 system-versioned tables are not yet in PostgreSQL — do not depend on them.

### 3.5 Derivation Ledger

The DBMS MUST maintain a **Derivation Ledger** as a first-class auditable structure linking, in both directions:

- source package → claim
- claim → fact
- fact / claim → Episode Object
- Episode Object → workflow trace
- workflow traces → generalized workflow
- workflows + memories → procedural memory
- procedural memory → skill package
- any object → embedding, summary, model-generated artifact

Each ledger entry MUST record: the parser, model, prompt template, policy version, verifier or human action that produced or revised the derived object; input hashes; transaction time; resulting object identifiers; and re-derivation eligibility.

The ledger MUST support **replay**: re-deriving any descendant from its source evidence under a chosen model/template/policy version, while preserving the prior derivation as historical record.

### 3.6 Verbatim Source Archive

Where policy, law, and retention permit, the DBMS MUST preserve raw source artifacts (documents, transcripts, tickets, logs, API payloads, screenshots, attachments) in immutable, addressable, hash-verified form. Source packages MUST be re-ingestible: future extraction, summarization, embedding, or workflow-mining models can operate on the original evidence without losing traceability to earlier derivations.

Lifecycle states for source packages: `held` (under legal/retention hold), `active`, `archived`, `pruned` (only when policy permits).

### 3.7 Recursive Detail and Memory Detail Objects

Memory Detail Objects MUST be:

- Addressable and queryable independently of their parent.
- Recursively containable (a detail may contain or link to other details).
- Reusable across memories, workflows, skills, and subjects via relationship edges (the same "uploaded ISO to Proxmox" detail may appear in many parent memories).

Retrieval MUST support **progressive expansion**: callers may request the parent only, the first level of details, all details to a specified depth, only details related to a subject or exception, or only the evidence behind a specific step.

### 3.8 Workflow Extraction Engine

The Workflow Extraction Engine is a required governed subsystem (not an optional summarization step). It MUST:

1. Select project/subject/action class/time window/source types/security domain to analyze.
2. Group events into candidate cases.
3. Extract steps from tickets, logs, transcripts, commands, documents, and Episode Objects with source links preserved.
4. Normalize wording into common action classes, objects, actors, tools, outcomes.
5. Order steps using event time + source time + transaction time, preserving uncertainty.
6. Construct per-case workflow traces with actors, inputs, outputs, evidence, confidence, and exceptions.
7. Cluster similar traces by subject type, action class, outcome, environment, tool stack, and exception pattern.
8. Propose **candidate** generalized workflows with provenance and review status. Candidates MUST NOT overwrite or auto-promote existing workflows or procedural memories.
9. Preserve **both positive and negative evidence** (failed/partial traces are first-class outputs).
10. Separate temporal sequence from causal assertion. A `caused_by` edge requires evidence beyond mere ordering.

### 3.9 Skill Runtime

Skill Packages MUST execute as governed state machines, not as free-form prompts. The runtime MUST:

- Bind the current account (human, agent, application, MCP server), active memory pool, task objective, authorized partitions, and source context.
- Enforce skill **applicability conditions** (project, environment, technology stack, time window, governing policy).
- Check **preconditions**, load only authorized memories and workflows, execute or present steps, evaluate validation checks, branch on known exceptions, and emit new claims/traces/procedural-memory updates.
- Produce an **auditable execution record** for every run, usable for conformance, refinement, staleness review, and incident investigation.

Packaging is allowed in multiple formats (system prompts, Markdown procedures, MCP tool definitions, application plugins) but the runtime contract is uniform.

### 3.10 Active Memory Pools and Local Memory Nodes

**Active Memory Pools** MUST:
- Be created from a prompt, API call, MCP request, or structured query.
- Materialize a scoped working set (memories, facts, workflows, skills, source references, pending claims) bounded by authorization, partitions, confidence thresholds, and validity windows.
- Support concurrent reads and writes by humans and AI agents, with every write attributed to an account.
- Preserve provenance, source pointers, confidence, precision, staleness, and access labels — they are not opaque caches.
- Support a promotion path: active observations → pending claims → verified facts → Episode Objects / workflow traces / procedural-memory updates / skill refinements.
- Expose real-time channels (HTTP API, WebSocket, or direct TCP) under the same identity, partition, and provenance rules as durable storage.

**Local Memory Nodes** MUST:
- Hold selected memories, pending claims, source snippets, task context, and synchronization metadata.
- Operate offline and synchronize back to the Enterprise Memory Core under governance — submitting new claims, Episode Objects, source packages, conflict records, deletions/tombstones, workflow updates, and promotion candidates.
- Never act as authoritative sources of record on their own.

### 3.11 Episode Replay Semantics

Replay MUST reconstruct an authorized, time-aware view of an Episode Object from durable evidence and derivation records. A replay must be able to answer:

1. What happened according to the current accepted view?
2. What evidence supports that view?
3. What did the DBMS believe at a prior transaction time?
4. What later sources or memories changed the interpretation?

Replay output MUST identify which source packages were inspected, which derived objects were included or hidden by policy, which temporal mode was used, and whether the replay is `historical`, `current_valid`, `as_of_transaction_time`, or `full_bitemporal`.

---

## 4. Retrieval Requirements

### 4.1 Search Path Planning

The DBMS MUST treat an incoming query as a request to **build a retrieval plan**, not as text to send directly to a vector index.

The planner MUST construct a **retrieval envelope** containing: authenticated account, roles, active memory pool, application context, agent delegation chain, allowed partitions, current task, query text, and structured fields supplied by the caller.

The planner MUST extract candidate retrieval cues — subject, verb/action class, object type, topic, time range, relationship hints, desired answer shape, evidence requirement, confidence threshold, precision requirement, recursive detail depth — and classify the likely intent. Initial path selection rules:

| Intent | Initial Path |
|---|---|
| Specific recall | subject + verb + temporal + catalog → graph/vector expansion |
| "Why" / "how do we know" | derivation ledger + evidence + supporting/contradicting edges |
| "How do we perform this task" | workflows → generalized workflows → procedural memory → skills + recursive detail |
| Date-sensitive | temporal indexes + bitemporal validity |
| Dependency/cause/relationship | graph traversal |
| Broad exploratory | scoped semantic search inside authorized partitions |

The plan MAY adapt: gather candidates from cheapest/most-precise indexes, then expand into graph, recursive detail, vector, source inspection, or workflow lookup as needed. Authorization, validity, confidence, precision, and provenance MUST be checked before result assembly.

### 4.2 Query Hints

The DBMS MUST accept an optional structured **query hint** package (intent, preferred_paths, subject, object_type, time_mode, detail_depth, evidence shape, minimum_confidence, partitions). Hints guide planning. Hints MUST NOT override security, access control, validity rules, provenance requirements, or DBMS safeguards. The system is free to ignore or downgrade a hint when it conflicts with policy or inferred intent.

### 4.3 Hybrid Recall

The retrieval engine MUST coordinate across:

- Relational/catalog filters (entity IDs, source IDs, fact IDs, workflow IDs, status, confidence/precision thresholds, partition membership, model version).
- Full-text search (`tsvector`/`tsquery` + GIN; `pg_trgm` for fuzzy match).
- Graph traversal over typed relationship edges.
- Temporal indexes (event_time, valid_time, transaction_time, verification_time, stale_after).
- Vector similarity search (pgvector, HNSW or IVFFlat) **scoped by subject, verb, predicate, object, project, or time partition** — global vector search is the fallback, not the default.
- Recursive detail expansion.

Result packages — not just ranked text — MUST include the underlying objects, source references, workflow steps, recursive depth used, confidence/precision/validity, permissions consulted, and links to evidence.

### 4.4 Pre-loaded / Cached Search Contexts

The DBMS MUST support pre-loaded subject-, topic-, or task-scoped buffer pools to bring high-value context close to active humans and agents. Pre-load operations use the same governed retrieval model as ordinary search and preserve all metadata; they are caches, not ungoverned shortcuts.

### 4.5 Authorization-aware Retrieval

Authorization MUST be evaluated:

1. During retrieval **planning** (path selection, partition restriction).
2. During **candidate expansion** (vector, graph, FTS each filter pre-rank).
3. During **result assembly** (final policy check, evidence redaction where required).

The system MUST NOT leak unauthorized information through vector similarity, graph traversal, summaries, relationship expansion, or active pool loading.

---

## 5. Security and Governance

### 5.1 Accounts

Every actor MUST have an account:

- Human users
- Applications
- Service accounts
- AI agents (with delegation chains)
- MCP servers
- Local memory nodes
- Administrative processes

Authentication MUST support: password, certificate, SCRAM, SSO/OIDC, API keys bound to a single account, and short-lived tokens for agent delegation.

### 5.2 Privileges

Privileges MUST apply to memory-specific objects and **semantic slices**, not only to tables and collections. Grantable scopes include:

- Subject (e.g., `Server X`, `Project Y`)
- Verb / action class (e.g., `decided`, `installed`)
- Topic (e.g., `authentication in PHP systems`)
- Project / Partition
- Source type (e.g., `email`, `ticket`)
- Workflow / generalized workflow / skill
- Active memory pool
- Relationship type

Built-in roles MUST include at least baseline patterns analogous to `CONNECT`, `RESOURCE`, `DBA`, plus memory-specific roles for `MEMORY_READ`, `MEMORY_WRITE`, `EVIDENCE_INSPECT`, `WORKFLOW_PROMOTE`, `SKILL_EXECUTE`, `MODEL_REGISTRY_ADMIN`, `AUDIT_VIEW`. Custom roles MUST be supported.

### 5.3 Audit and Observability

The DBMS MUST record: who accessed/changed/verified/promoted/synchronized/exported/deleted/derived governed objects; ingestion jobs; model executions and their prompt/template/policy versions; index rebuilds; archive recalls; synchronization events; recovery operations; quality and performance metrics; and traces.

Recommended baseline: `pgaudit` + `pg_stat_statements` + logical decoding to an append-only sink for compliance-grade trails.

### 5.4 Backup and Recovery

Backup MUST prioritize **durable assets that cannot be cheaply regenerated**:

- Verbatim source packages
- Catalog metadata
- Access-control state
- Human verification decisions
- Derivation ledger
- Transaction logs / WAL
- Memory objects, facts, claims, governance records

**Derived artifacts** (embeddings, vector indexes, summaries, routing structures) MAY be excluded from full backups if they can be rebuilt from source data and metadata. Tested rebuild ordering, restore validation, and audit records of the models/templates/source snapshots used during recovery are required.

### 5.5 Privacy, Retention, Legal Hold

The DBMS MUST support: retention policies per partition / source class / object type; legal holds that prevent pruning regardless of policy; per-subject erasure workflows that respect immutable evidentiary requirements (e.g., redaction with audit, not silent deletion).

---

## 6. Architectural Subsystems

| Subsystem | Responsibility |
|---|---|
| **Enterprise Memory Core** | Durable system of record for all governed objects, catalog, audit, model registry. |
| **Local Memory Node** | Offline/edge subset; synchronizes under governance. |
| **Active Memory Pool Manager** | Scoped working-memory lifecycle; real-time channels; promotion path. |
| **Temporal Supersession Engine** | Bitemporal window management, supersession, contradiction, staleness propagation, downstream invalidation. |
| **Security-Aware Retrieval Coordinator** | Builds retrieval envelopes, plans search paths, enforces authorization at planning/expansion/assembly. |
| **Transaction & WAL Manager** | Multi-model transaction boundary across object metadata, source links, graph edges, temporal windows, FTS, vector indexes, workflow records, audit logs. v1 leverages PostgreSQL's WAL/MVCC; partial commits are forbidden. |
| **Workflow Extraction Engine** | Process-mining over Episode Objects; produces candidate traces and generalized workflows for review. |
| **Skill Runtime** | Governed state-machine execution of skill packages with audit records. |
| **Model Registry** | Embedding/extraction/reranker/summarizer model identity, version, dimensions, embedding space, prompt-template policy, evaluation status, rollout state, derived-artifact map. Supports blue-green indexes, dual-space query routing, adapter alignment, staged background re-embedding. |
| **System Catalog** | Central metadata: users, roles, privileges, schemas, partitions, object types, relationship types, retention policies, source types, model versions, index definitions, active pools, local nodes, archive tiers, rebuild state, operational statistics. Readable through SQL and governed APIs. |
| **Derivation Ledger Service** | Lineage, replay, audit. |
| **Verbatim Source Archive Service** | Immutable hash-verified storage with tiered hot/warm/cold/archived placement. |
| **Driver / API Surface** | C, Python, Node.js, PHP drivers; SQL interface; prompt-plus retrieval API; MCP integration. |

### 6.1 Process Architecture

MaluDB v1 MUST use PostgreSQL as the authoritative transaction, storage, and query-execution boundary. Relational data, temporal records, relationship edges, full-text indexes, vector columns, pgvector indexes, catalog state, audit records, and derivation metadata live in PostgreSQL and participate in PostgreSQL WAL/MVCC semantics.

The default deployment MUST NOT split SQL storage and vector search into separate databases. Vector search is database work: pgvector runs inside PostgreSQL backend processes and is planned/executed through PostgreSQL queries, indexes, partitions, and access-control checks. Embedding generation is model work: embedding models run in governed worker processes or approved external model services and write vectors back to PostgreSQL through audited transactions.

Default process roles:

| Process / Service | Default placement | Responsibility |
|---|---|---|
| PostgreSQL server | Integrated MaluDB-managed PostgreSQL cluster | SQL engine, transactions, WAL/MVCC, storage, catalog, FTS, temporal indexes, relationship tables, pgvector storage/search, audit tables. |
| MaluDB C extension | Loaded into PostgreSQL backend processes | Server-side functions, type support, validation, catalog helpers, transaction-safe write APIs, optional background-worker hooks. |
| PostgreSQL background workers | PostgreSQL process space when required | Internal maintenance that benefits from database locality, such as invalidation queues, rebuild scheduling, lightweight asynchronous jobs, and cache coordination. |
| Embedding workers | Separate governed MaluDB service processes | Generate embeddings from source excerpts, memory chunks, workflow traces, summaries, and query packages using model-registry policy; write results and ledger entries back to PostgreSQL. |
| Extraction / summarization / workflow-mining workers | Separate governed MaluDB service processes | Produce claims, facts, summaries, workflow traces, candidate generalized workflows, and derived artifacts for review; preserve source links and derivation records. |
| Retrieval API / MCP service | Separate governed MaluDB service process | Expose prompt-plus retrieval and agent-facing APIs while delegating durable reads/writes and policy checks to PostgreSQL-backed MaluDB functions. |
| Active Memory Pool manager | Separate governed MaluDB service process backed by PostgreSQL | Manage low-latency task context, WebSocket or equivalent real-time channels, concurrent human/agent access, and promotion paths. |
| Skill Runtime | Separate governed MaluDB service process by default | Execute skill state machines, especially when actions may call external tools or applications; persist execution records and derived observations through PostgreSQL transactions. |
| Local Memory Node sync service | Separate governed MaluDB service process | Synchronize offline/edge subsets back to the Enterprise Memory Core with conflict records, tombstones, source packages, claims, and promotion candidates. |
| Verbatim archive service | Separate governed service or storage adapter with PostgreSQL catalog authority | Store immutable source blobs and tiered archive objects; PostgreSQL stores authoritative metadata, hashes, retention state, and access policy. |

External MaluDB services MUST NOT become independent systems of record. They MAY perform model execution, long-running extraction, real-time coordination, synchronization, archive placement, and effectful skill execution, but durable state changes MUST flow through PostgreSQL transactions and, for derived objects, Derivation Ledger entries. Any operation that cannot be made atomic with the relevant PostgreSQL state MUST document the failure mode, reconciliation behavior, and audit trail.

---

## 7. Non-Functional Requirements

### 7.1 Performance Targets (initial; refine after benchmarking)

- Single-subject scoped semantic search over a partition with 10⁷ memories: p95 < 200 ms with HNSW.
- Active memory pool load (subject-scoped, depth-1 expansion, ≤ 5,000 objects): p95 < 1 s.
- Episode replay (current_valid, no full audit): p95 < 500 ms.
- Workflow Extraction Engine MAY run asynchronously; SLAs are batch-oriented.

### 7.2 Scalability

The system MUST partition by subject, project, security domain, time range, workflow, source domain, retention policy, and active pool. Cross-partition queries MUST be explicit about authorization and result assembly.

The architecture MUST not preclude future horizontal distribution. Active-active replication is out of scope for v1, but partitioning, the catalog, and the WAL/transaction model MUST remain compatible with a future distributed deployment.

### 7.3 Reliability

- ACID guarantees inside the Enterprise Memory Core: full PostgreSQL semantics.
- Multi-model writes (object + source links + graph edges + temporal windows + FTS + vectors + audit) MUST be atomic. Partial writes MUST be impossible.
- Critical promotion and supersession operations MUST run at stronger isolation than low-risk maintenance.

### 7.4 Embedding Lifecycle

- Multi-view embedding policies MUST be configurable. Embedding budgets, lazy/deferred embedding, compact routing vectors, partition-specific embedding scopes, and rebuild prioritization are required.
- High-value active partitions MAY receive richer embeddings; cold historical partitions MAY rely on fewer embeddings until queried, restored, or selected for reprocessing.
- Embedding model migrations MUST avoid long downtime: blue-green indexes, dual-space query routing during transition, adapter-based alignment between embedding spaces, or staged background re-embedding are all acceptable.

### 7.5 Operability

- All subsystems MUST emit structured logs and metrics (Prometheus-compatible).
- The system catalog MUST expose statistics about predefined slices (subjects, verbs, topics, projects, security domains, active pools, time partitions).
- Health, readiness, and liveness endpoints MUST be exposed for each long-running service.

---

## 8. Technology Stack and Dependencies

| Layer | Choice | License | Notes |
|---|---|---|---|
| Database core | **PostgreSQL 17** (PGDG) | PostgreSQL License | Installed/configured by the default MaluDB bundle. PG 16 supported; PG 18 adoption tracked. No PostgreSQL fork in v1. |
| Vector | **pgvector** ≥ 0.8 | MIT | HNSW + IVFFlat, halfvec. Default vector engine; runs inside PostgreSQL, not as a separate vector database process. |
| Graph | **Apache AGE** ≥ 1.5 (Cypher) | Apache 2.0 | Used where Cypher is needed; recursive CTEs preferred for moderate graphs. PG 17 support landed in 2025. |
| Temporal | Native `tstzrange` + GIST exclusion; optionally `periods` extension | PostgreSQL / MIT | `temporal_tables` is unmaintained — do not use. |
| Partitioning | Native declarative partitioning + `pg_partman` | PostgreSQL | Avoids TimescaleDB TSL license. |
| FTS | Native `tsvector` + GIN; `pg_trgm` for fuzzy | PostgreSQL | `pg_search` (ParadeDB, AGPL) considered later if BM25 ranking required. |
| JSON | `JSONB` + `pg_jsonschema` | PostgreSQL / Apache 2.0 | For validated documents. |
| Audit | `pgaudit` + `pg_stat_statements` + WAL streaming | PostgreSQL | Logical decoding to immutable sink for compliance. |
| ML/embedding | Governed MaluDB worker services + pgvector storage; `pg_vectorize` for managed refresh | PostgreSQL | Embedding/model execution runs outside PostgreSQL by default and writes audited results back through SQL. `PostgresML` only if in-DB inference is a hard requirement. |
| Replication | Native logical replication; `pglogical` for selective use | PostgreSQL | Citus deferred (AGPL). |
| Build | **PGXS** Makefile; `pg_buildext` / `dh_make_pgxs` for `.deb` | — | Meson-for-extensions is experimental — not adopted in v1. |
| CI | GitHub Actions matrix on `ubuntu-24.04`, PG 16/17/18 | — | ASan/UBSan job; `clang-tidy` and `scan-build` jobs. |

**Avoid:**
- `pg_embedding` (Neon) — archived.
- `temporal_tables` — unmaintained.
- `ZomboDB` — heavy operational cost unless Elasticsearch is already deployed.
- TimescaleDB **TSL features** — license incompatibility for our redistribution model. Apache-2.0 core only is acceptable; using TSL features is not.

**License watchlist:** TimescaleDB TSL, Citus AGPL, ParadeDB AGPL.

**License contamination:** Linking GPL code into our `.so` is prohibited unless the entire derived work can be released under PostgreSQL License terms. LGPL is acceptable when dynamically linked with notices preserved.

---

## 9. Phased Implementation Plan

The roadmap is intentionally narrow at the start. Each stage delivers something usable and testable before scope expands. **Do not implement Stage N+1 features inside Stage N** even when the requirements above call for them — early stages establish the substrate; later stages layer the memory semantics on top.

### Stage 1 — Memory Core framework (PostgreSQL + pgvector relational foundation)

**Goal:** a working database that can store relational data with vector columns, plus the dev/build/test/packaging substrate that all later stages will use. No memory-specific object model yet.

In scope:
- PG 17 dev environment on Ubuntu 24.04 with PGDG.
- pgvector installed and verified end-to-end (HNSW + IVFFlat indexes, distance operators).
- A `maludb_core` extension skeleton built with PGXS, installable as a `.deb` via `pg_buildext`.
- Default MaluDB installation packaging that installs and configures PostgreSQL 17 from PGDG, pgvector, required PostgreSQL extension packages, and `maludb_core` together on a fresh Ubuntu 24.04 host.
- Bootstrap logic that initializes or selects the managed PostgreSQL cluster, validates `PG_CONFIG`, installs required extensions, runs `CREATE EXTENSION maludb_core`, and verifies vector search through the MaluDB-owned schema.
- A baseline relational schema for the catalog scaffolding the rest of the system will rely on (accounts, partitions, object-type registry, source-type registry, relationship-type registry, model registry stubs). Tables only — no business logic, no MAUT scoring, no bitemporal columns yet.
- A simple "memory record" demonstration table with at least one `vector` column to prove pgvector is wired up correctly through the extension's install path.
- `pg_regress` harness with smoke tests covering: extension load, table creation, vector insert/query, basic catalog reads.
- ASan/UBSan debug build target.
- `clang-tidy` and `scan-build` jobs.
- GitHub Actions matrix on `ubuntu-24.04` × PG 16/17 (PG 18 added when stable on PGDG).
- License declaration (PostgreSQL License), DCO, repository conventions, CLAUDE.md, requirements.md.

Explicitly NOT in Stage 1:
- Sources, claims, facts, Episode Objects, memories, Memory Detail Objects (Stage 2).
- Verbatim source archive (Stage 2).
- Bitemporal columns and the Temporal Supersession Engine (Stage 3).
- SVPOR organization layer (Stage 3).
- Derivation Ledger (Stage 2/3).
- Confidence/precision MAUT scoring (Stage 3).
- Retrieval planner / query hints / hybrid recall (Stage 4).
- Active Memory Pools, Local Memory Nodes, Skill Runtime, Workflow Extraction Engine (Stages 4–5).

Stage 1 done when: a fresh Ubuntu 24.04 box can install MaluDB through the default package/bundle path, which installs and configures PostgreSQL 17 + pgvector + `maludb_core`; run `CREATE EXTENSION maludb_core`; insert and similarity-search a vector through a `maludb_core`-owned table; and pass the regression suite under both default and ASan builds. The developer path MUST still support explicit `PG_CONFIG` builds across PG 16 and PG 17.

### Stage 2 — Memory object model and verbatim storage

**Goal:** internal layout of memories, documents, and verbatim source storage. The substrate from Stage 1 gets the first memory-specific object types.

- Schema for source packages, claims, facts, Episode Objects, memories, Memory Detail Objects, relationship edges.
- Verbatim Source Archive: immutable hash-verified storage, tiered placement, retention/legal-hold metadata.
- Document/JSON layout for memory payloads (`JSONB` + `pg_jsonschema`).
- Derivation Ledger schema + write API (without the MAUT scoring and supersession behavior — those land in Stage 3).
- C extension functions that enforce multi-model atomicity for object insertion.
- `pgaudit` + `pg_stat_statements` configured.

### Stage 3 — Bitemporal core, SVPOR, MAUT scoring

- Bitemporal columns (`event_time`, `valid_time_*`, `transaction_time_*`, `source_time`, `verification_time`, `stale_after`) using `tstzrange` + GIST exclusion.
- Temporal Supersession Engine: validity-window closure, supersession edges, staleness propagation, downstream invalidation hooks.
- SVPOR organization layer + routing indexes; SVPOR participation in embedding inputs.
- MAUT-based confidence and precision: per-category subscore tables, weights, evaluator metadata, aggregate computation functions.

### Stage 4 — Retrieval, hybrid search, query hints

- Apache AGE for graph traversal (or recursive CTE fallback) integrated.
- Native FTS (`tsvector`) + `pg_trgm`.
- Retrieval planner: envelope, cue extraction, intent classification, search-path selection.
- Query-hint API.
- Authorization-aware retrieval at planning, expansion, and assembly.

### Stage 5 — Workflow Extraction, Skills, Active Memory Pools

- Workflow Extraction Engine v1.
- Skill Runtime as a governed state machine.
- Active Memory Pool manager + WebSocket channel.
- Episode replay API.

### Stage 6 — Local Memory Nodes, Model Registry, driver surface

- Local Node sync protocol + conflict records.
- Model Registry with blue-green index migration, dual-space query routing, adapter-based alignment.
- Drivers/SDKs for C, Python, Node.js, PHP.
- MCP server reference implementation.

### Stage 7 — Hardening

- Performance benchmarking and tuning.
- Security review (`pgaudit`, RLS, semantic-slice grants).
- Documentation, examples, deployment guide, deb packaging finalized via `pg_buildext`.
- Public alpha.

---

## 10. Deferred / Open Decisions

These are explicitly **not** decided in v1; the implementation MUST keep them open.

- Custom storage engine vs. pure PostgreSQL extension long-term.
- Distributed/horizontal scaling architecture (Citus vs. sharding plus federation vs. fork).
- Exact MAUT default weights per object type (must be policy-configurable from day one, but the defaults need tuning data).
- Default embedding model and dimension (decoupled from the system; lives in the Model Registry).
- Causal reasoning subsystem.
- Whether the Skill Runtime ever executes effectful tools directly or always delegates to an external agent harness (MCP).
- Whether to adopt `pg_search` (ParadeDB, AGPL) for BM25 ranking or stay on native FTS.
- Whether `extension_control_path` (PG 18) becomes a hard requirement (would bump min PG to 18).

---

## 11. Glossary (canonical terms)

| Term | Meaning |
|---|---|
| **Episode Object** | DBMS object for a specific remembered episode, including evidence, time, actors, relationships, replay metadata. |
| **Memory Detail Object** | Addressable child/linked object representing a step, substep, parameter, command, validation, exception, source excerpt, or evidence item. |
| **Recursive Detail** | Ability to expand Memory Detail Objects only to the depth a task requires. |
| **Workflow Trace** | Observed step sequence from one Episode Object/case. |
| **Generalized Workflow** | Repeatable process pattern derived from multiple traces. |
| **Procedural Memory Object** | Capability-oriented how-to knowledge for performing, adapting, validating, and repairing work. |
| **Skill Package** | Governed procedural memory packaged for reuse by humans or AI agents with execution policy and audit records. |
| **Active Memory Pool** | Scoped working-memory space for a task, project, incident, or collaboration session. |
| **Derivation Ledger** | Auditable lineage from source through claims, facts, memories, workflows, procedural memories, skills, and derived artifacts. |
| **Temporal Supersession Engine** | DBMS subsystem for valid-time/transaction-time windows, corrections, supersession, staleness, current validity. |
| **Security-Aware Retrieval Coordinator** | Subsystem that builds authorization context, prefilters retrieval, post-validates candidates, assembles policy-aware result packages. |
| **Transaction and WAL Manager** | Subsystem providing MVCC, global commit sequencing, isolation policy, crash recovery, internal multi-model atomicity. |
| **SVPOR** | Subject-Verb-Predicate-Object-Relationship organization model. |
| **MAUT** | Multi-Attribute Utility Theory — the weighted-aggregate scoring model used for confidence and precision. |

---

*This document derives from [`white-paper.md`](white-paper.md). When a conflict arises between this document and the white paper, the white paper prevails for conceptual intent; this document prevails for implementation requirements. Both must be updated together when scope changes.*
