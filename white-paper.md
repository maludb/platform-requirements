# A Scalable Database Management System for Long-Term Institutional Memory, Human-AI Knowledge Sharing, and Contextual Recall

**Authors:** Edward Honour, Joseph Lehman, Moshin Ali, Chadi Abi Fadel, Otman Mechbal, Arjun Nadir, Mohammed Khan, Alan Nadal Palini, Rashab Jolly, Eben Ray

# Abstract

As AI agents become participants in organizational work, the need for durable institutional memory becomes more urgent. Conventional databases are effective at storing facts: what is known or recorded, what changed, who owns a record, or what value a system currently reports. They are less effective at preserving memories: how knowledge came to exist, why a decision was made, what evidence supported it, what exceptions occurred, and how humans or AI agents should use that context later.

This paper proposes a scalable database management system for long-term institutional memory, human-AI knowledge sharing, and contextual recall. The proposed architecture treats memories as first-class data objects linked to facts, claims, evidence, provenance, temporal validity, relationships, workflows, and source material. It combines structured storage, semantic search, temporal reasoning, graph relationships, verbatim source archives, access control, and active memory pools for real-time human-agent collaboration.

By preserving not only what an organization knows but how that knowledge was formed and used, the system is intended to reduce repeated context reconstruction, support reliable AI-agent continuity, and provide a durable foundation for deriving workflows, procedural memories, skills, and future model-driven improvements.

# Motivation: The Systems Gap

Existing systems that attempt to preserve and retrieve organizational memory usually assemble several specialized technologies. Relational databases handle structured records. Document stores preserve source material. Vector databases support semantic search. Graph databases represent relationships. Time-series databases record events and operational history. This polyglot approach can be useful, but it pushes the core semantics of memory into application logic.

That shift creates long-term operational risk. Provenance, temporal validity, evidence links, security enforcement, embedding lifecycle management, relationship traversal, and workflow derivation become integration concerns rather than native database responsibilities. As the system grows, the same approach can also create practical challenges for scalability, performance tuning, backup and recovery, restore validation, cross-system consistency, security policy enforcement, auditability, and lifecycle management. The central problem is that memory behavior becomes distributed across multiple products instead of governed inside a single coherent DBMS.

A database of memories requires these capabilities to operate together around a shared memory model. Facts, claims, source documents, embeddings, relationships, event histories, workflows, procedural memories, and active collaboration pools must be governed by common system services. Those services include a catalog, access-control model, transaction boundary, audit trail, and query surface. The goal is not merely to connect existing databases. The goal is to define a single database system whose primary purpose is representing, maintaining, retrieving, and operationalizing memories and workflows over time.

This framing is important because memories are not only stored information. They are contextual structures. They link what is known to how it became known, when it was valid, who or what produced it, what evidence supports it, what other memories it relates to, and how humans or AI agents can use it in future work. A memory DBMS therefore needs multi-model capabilities, but those capabilities must be organized around memory objects rather than around separate storage products.

# Goals, Objectives, and Non-Goals

The proposed memory DBMS is intended to solve a long-term organizational problem rather than a narrow application-storage problem. It must preserve institutional knowledge across changing teams, systems, tools, and AI capabilities while remaining scalable, governable, and operationally useful (Walsh & Ungson, 1991; Stein & Zwass, 1995; Argote & Ingram, 2000). The following goals, objectives, and non-goals clarify the design priorities of the proposed system.

## Goals

The primary goal of the proposed memory DBMS is to preserve institutional memory as a first-class organizational asset. The system should store claims, facts, memories, workflows, provenance, temporal scope, and evidentiary context in a unified architecture that can scale across many years of organizational activity, many source systems, and many interacting humans and AI agents. It should ingest and reconcile heterogeneous material, including documents, conversations, tickets, logs, databases, and time-series or event data from different platforms and operational environments, while maintaining the indexes, relationships, provenance structures, confidence states, precision states, and validity states needed for long-term use.

The system should also make institutional memory operationally useful. Humans and AI agents should be able to retrieve contextual memory, inspect supporting evidence, reuse validated organizational knowledge, and share task-specific working context through active memory pools. Repeated successful events should be convertible into workflow traces, procedural memories, skills, and competencies without losing the original evidence from which those patterns were derived. The architecture should be future-proof in an evolving AI landscape by separating durable memory structures from replaceable AI components, allowing improved embedding models, extraction models, ranking models, vector indexes, multi-vector retrieval methods, and other AI subsystems to be upgraded without redesigning the core memory model. Finally, the system should be practical to deploy on common hardware and operating systems, with an enterprise focus on Linux servers and a path for desktop implementations where local memory nodes, development environments, or smaller deployments are appropriate.

## Objectives

The first objective is to define native data structures for the full memory lifecycle: sources, claims, facts, memories, workflow traces, generalized workflows, procedural memories, skills, relationships, provenance, confidence, precision, and temporal validity. These structures should support exact search, semantic search, temporal search, graph traversal, and provenance inspection within governed partitions and scoped retrieval paths. They should also preserve historical truth, current validity, staleness, contradiction status, and supersession relationships so that the system can distinguish what was true at a prior time from what remains safe to use now.

The second objective is to support both retrospective and continuous memory formation. The system should ingest legacy artifacts as well as active streams from current systems, including cross-platform event and time-series sources. It should allow humans and AI agents to create topic-scoped or project-scoped shared working sets that dynamically gather relevant memories, facts, workflows, and source references for a specific task, domain, project, or technology stack. Within those shared pools, high-speed active memory caches should allow collaborators to load and reuse concentrated context, such as authentication-related memories and workflows for a given project or technical environment.

The third objective is to make the AI-dependent parts of the system governable and replaceable. Embeddings, reranking strategies, retrieval pipelines, vector search algorithms, extraction models, and other AI-derived artifacts should be modular, versioned, reevaluable, and replaceable over time. The architecture should support side-by-side evaluation and controlled adoption of improved AI techniques, including newer embedding models and richer retrieval strategies such as multi-vector search, without requiring loss of provenance or redefinition of the underlying memory objects. Across all of these objectives, the system must maintain security, governance, auditability, and access control for both human and AI use.

## Non-Goals

This paper does not prescribe a single vendor, foundation model, embedding model, vector database, retrieval algorithm, or cloud provider as the permanent implementation choice. It also does not propose a vector-only architecture or treat AI-generated outputs as authoritative truth without evidence, policy, verification, and auditability. The memory DBMS is intended to use AI and vector retrieval as important components, not as substitutes for provenance, temporal reasoning, access control, and human judgment.

The proposed system is also not intended to replace authoritative source systems such as email platforms, ticketing systems, source control, observability systems, or transactional databases. Those systems remain sources of record for their own domains; the memory DBMS preserves, links, interprets, and recalls institutional knowledge derived from them. The initial design also does not attempt to solve full high-availability clustering, active-active replication, or every form of distributed failover. Those capabilities may be added later, but they are intentionally out of scope while the core memory, provenance, retrieval, and model-maintenance architecture is being defined.

Finally, the system is not intended to preserve every transient message, intermediate thought, or temporary artifact as durable institutional memory without curation, validation, and lifecycle policy. Nor does the paper assume that all institutional reasoning can or should be automated. Human judgment, organizational governance, verification, and policy remain necessary parts of the architecture.

# 1. Memories vs Facts

A native database management system for long-term institutional memory must distinguish between **facts**, **claims**, **evidence**, and **memories**. These concepts are closely related, but they are not interchangeable (Walsh & Ungson, 1991; Stein & Zwass, 1995). A conventional database may store records as discrete values, rows, documents, or objects, but a database of memories must preserve not only what is known, but also how it became known, when it was believed to be true, how precise the knowledge is, how reliable it is, and whether it remains useful over time.

This distinction is essential because institutional knowledge is often derived from imperfect sources. Emails, meeting notes, chat messages, tickets, documents, system logs, and human recollections may each contain useful information, but they do not all carry the same evidentiary weight (Argote & Ingram, 2000; Buneman et al., 2001; Ko et al., 2007; Moreau & Missier, 2013). A message may state an intention, a ticket may indicate completion, a log may record an event, and a later document may reinterpret the significance of that event. A database of memories must therefore avoid treating every extracted statement as a fact. Instead, it must model the path from source material to claim, from claim to fact, and from fact to memory (Buneman et al., 2001; Moreau & Missier, 2013).

In this paper, a **claim** is an assertion made by a source. A claim may be explicit, such as a sentence in an email, or inferred, such as an extracted assertion from a meeting transcript. A claim may be true, false, incomplete, ambiguous, stale, or unverifiable. A **fact** is a claim, or a set of related claims, that has been accepted as true according to the evidence, rules, policies, or verification standards of the system. A **memory** is a contextual representation of something that happened, was decided, was discovered, was changed, was learned, or became significant over time.

This relationship may be summarized as follows:

> Sources provide evidence. Evidence supports claims. Claims may be verified into facts. Facts and claims may be organized into memories. Memories provide contextual recall and operational guidance.

## 1.1. Facts

A fact is an assertion that the system accepts as true within a defined scope. Facts are usually more atomic than memories. They may describe a specific event, property, relationship, status, measurement, identity, or timestamp. Examples include:

-   "Server X was installed on March 12, 2019."
-   "Alice was the project manager for Project Y."
-   "The production database was migrated from Server A to Server B."
-   "The customer approved the revised contract on July 8, 2024."
-   "Version 2.1.4 was deployed to production."

However, facts in an institutional memory system should not be treated as context-free truths. A fact may be true only within a specific time period, project, department, system configuration, or operational context. For example, the fact that Server X was the production server in 2019 may remain historically true, but it may no longer be operationally valid in 2026. Therefore, facts should retain temporal scope, provenance, verification status, confidence, and precision (Kulkarni & Michels, 2012; Moreau & Missier, 2013).

Newly derived memories may also force the system to reevaluate earlier facts. For example, a fact such as "On March 25, 2023, there were three A100 GPUs in the server room" may later be challenged by a newly derived memory stating that all A100 GPUs were removed from the server room on March 24, 2023. In that case, the system should not silently overwrite the old fact. It should recognize a contradiction, inspect the source evidence and timeline, and determine whether the earlier fact was wrong, whether the newer memory was wrong, or whether the validity window of one or both objects must be narrowed. In other cases, a later memory may not contradict the earlier fact but may instead supersede it by showing that a once-true fact later became invalid for current use.

A fact in the memory DBMS should answer several questions:

-   What is being asserted?
-   What evidence supports the assertion?
-   When was the assertion true?
-   When was it verified?
-   How confident is the system that it is true?
-   How precise is the assertion?
-   Has it been contradicted, superseded, or made stale?

This allows the system to distinguish between a fact that is historically accurate and a fact that is currently actionable.

## 1.2. Claims

Claims are necessary because most institutional knowledge begins as an assertion rather than as a verified fact. A claim preserves the original meaning of a source without prematurely converting that source into institutional truth.

For example, an email may state:

> "I am installing the server today."

This statement should not immediately be stored as the fact:

> "The server was installed today."

The email supports a more limited claim:

> "The sender stated an intention or expectation to install the server on that day."

Additional evidence may later support a stronger fact. A completed ticket, server log, configuration record, monitoring entry, or follow-up message may allow the system to derive:

> "The server was installed on March 12, 2019."

The distinction is important because a claim may represent an intention, a belief, a plan, a report, a promise, a conclusion, or an observation. Not all claims should become facts. Some may remain unverified. Others may be contradicted. Others may be partially true but imprecise.

Claims should therefore include explicit references to their sources. A claim should be connected to one or more documents, messages, transcripts, logs, database records, or other evidentiary artifacts. These references should include enough detail to allow humans and AI agents to inspect the original basis for the claim, including document identifiers, timestamps, authorship, page numbers, message identifiers, transcript offsets, or line numbers where applicable (Buneman et al., 2001; Cheney et al., 2009; Moreau & Missier, 2013).

## 1.3. Memories

A memory is broader than a fact. A memory captures not only an assertion, but also its institutional meaning. Memories are contextual, temporal, relational, and often narrative. They represent events, decisions, discoveries, assumptions, lessons learned, procedural knowledge, incidents, warnings, preferences, dependencies, and changes that matter over time (Walsh & Ungson, 1991; Stein & Zwass, 1995; Hogan et al., 2021).

For example, the fact:

> "Server X was installed on March 12, 2019."

may contribute to a memory such as:

> "Server X was installed in March 2019 as part of the organization's migration from shared hosting to a dedicated application environment. The installation was performed to support Project Y, and later became the basis for the production deployment architecture used by the team."

The fact identifies what happened. The memory explains why it mattered, how it related to other activities, and how future users or AI agents may need to interpret it (Walsh & Ungson, 1991; Stein & Zwass, 1995).

Memories often combine multiple facts and claims. A memory may be derived from emails, chat conversations, tickets, meeting notes, source code commits, database records, and operational logs. It may also include relationships to people, teams, projects, systems, decisions, documents, and later outcomes. For this reason, the proposed architecture treats memories as first-class objects in the database, not merely as text summaries or vector embeddings (Walsh & Ungson, 1991; Stein & Zwass, 1995; Hogan et al., 2021).

The architecture also treats **episodic memory** as a first-class conceptual object type. In cognitive-science terminology, episodic memory refers to memory of specific experiences situated in time and context. In this DBMS proposal, the formal implementation term is **Episode Object**. The paper may also use **event memory** as a readable synonym, but the database object is an Episode Object: a remembered episode that records what happened, when it happened, who or what was involved, what evidence supports it, what context surrounded it, and why it later mattered. This preserves compatibility with AI-memory and cognitive-science terminology while keeping the database object model concrete and operational (Tulving, 2002; Squire, 2004; Park et al., 2023).

The paper therefore maps common memory terminology into DBMS terms as follows. Episodic memory corresponds to an Episode Object, also described informally as event memory: a specific remembered experience with time, actors, context, evidence, and outcomes. Semantic memory corresponds to facts, relationships, subject knowledge, and graph or index structures that represent what the organization knows. Procedural memory corresponds to retained how-to knowledge: the capability-oriented memory used to perform, adapt, validate, and repair repeatable work. A procedural memory may be supported by workflow traces, generalized workflows, examples, exceptions, tool choices, validation rules, and skills, but it is not identical to a workflow. Working memory corresponds to active memory pools, where humans and AI agents temporarily maintain and manipulate task context before selected outputs are promoted into long-term institutional memory (Baddeley, 1992; Cohen & Squire, 1980; Squire, 2004; Packer et al., 2023).

A memory may describe a specific Episode Object or episodic memory, such as a server installation, outage, deployment, meeting, migration, or customer interaction. It may also preserve the context around a decision, including why a technology, vendor, design, or policy was selected, or record a discovery such as a defect, risk, performance issue, or business insight. Some memories capture procedural knowledge: how a task is performed, adapted, validated, or repaired. Others preserve lessons learned, assumptions that guided prior work, relationships among systems or organizational actors, and changes that superseded earlier systems, processes, responsibilities, or facts.

Memories act on a timeline. They may describe what happened in the past, what was believed at a particular time, what changed later, and what remains valid now. This makes the temporal model of a memory DBMS central to its design (Kulkarni & Michels, 2012).

## 1.4. Episode Objects and Replay Semantics

An **Episode Object** is the concrete DBMS representation of a specific remembered episode. It is not merely a text summary, log entry, or embedding. It is a governed memory object that binds source evidence, extracted structure, temporal scope, relationships, security labels, and lifecycle state into an auditable unit.

At minimum, an Episode Object should have a stable identity, a version history, and enough typed structure to explain what kind of episode it represents. This includes an episode type, such as an installation, outage, decision meeting, migration, approval, incident, or customer interaction; subject anchors for the people, systems, assets, projects, teams, customers, or organizations involved; and a normalized action class that describes what happened. It should also contain a semantic frame with structured fields for purpose, rationale, outcome, constraints, environment, and other predicate-like details, along with a target or payload that identifies the affected systems, documents, services, artifacts, or remembered objects. Participants and agents should be represented explicitly so the system can distinguish humans, AI agents, services, vendors, approvers, verifiers, tools, and organizational teams.

The same object should also bind the episode to evidence, time, governance, and lifecycle state. It should link to verbatim source packages, documents, logs, tickets, transcripts, records, or API payloads; to the claims and accepted facts used to construct or revise the episode; and to relationship edges such as supports, contradicts, supersedes, derived_from, caused_by, depends_on, part_of, and related_to. When procedural information is available, the Episode Object should link to workflow traces containing observed steps, commands, approvals, checks, exceptions, and outcomes. Temporal fields should distinguish event time, source time, valid-time windows, transaction-time windows, verification time, and stale-after rules. The object should also carry confidence, precision, salience, lifecycle state, security labels, partition membership, subject and topic access constraints, and a derivation ledger entry showing how the episode was produced, revised, replayed, promoted, superseded, archived, or retired.

Replay semantics define how the DBMS reconstructs an episode for inspection, audit, retrieval, or reprocessing. A replay does not mean reproducing the original world event. It means reconstructing an authorized, time-aware view of the episode from the durable evidence and derivation records available to the DBMS.

An episode replay should be able to answer at least four questions: what happened according to the current accepted view, what evidence supports that view, what the DBMS believed at a prior transaction time, and what later sources or memories changed the interpretation. To do this, the replay operation should load the Episode Object, source links, claims, facts, relationship edges, workflow traces, temporal windows, supersession records, and derivation ledger entries under the requesting account's privileges. The result should identify which source packages were inspected, which derived objects were included or hidden by policy, which temporal mode was used, and whether the replay is historical, current-valid, transaction-time, or full bitemporal.

This replay model is important for future-proofing. If better extraction, summarization, causal analysis, or embedding models become available, the DBMS can replay selected episodes from their original evidence and derivation ledger without losing the historical record of how earlier interpretations were produced.

# 2. Confidence and Precision

Both facts and memories should include measures of **confidence** and **precision**, but these measures should be modeled separately. A statement may be highly precise but poorly supported, or well supported but imprecise. Precision should not be confused with accuracy. Accuracy concerns whether a statement is correct. Precision concerns how specific the statement is. Confidence concerns how strongly the system believes the statement is reliable based on available evidence.

For example:

> "The server was installed in 2019."

This statement may have high confidence but low temporal precision.

> "The server was installed on March 12, 2019 at 12:30 PM."

This statement has higher temporal precision, but it is not necessarily more accurate. If the only source is an email saying "I am installing the server today," then the exact time of 12:30 PM may have low confidence unless supported by stronger evidence such as a log entry or ticket timestamp.

Therefore, a memory DBMS should model at least two separate dimensions:

1.  **Confidence** --- how reliable the system believes the fact or memory is.

2.  **Precision** --- how specific the fact or memory is.

A useful principle is:

> A fact or memory may be precise without being reliable, reliable without being precise, historically true without being currently valid, and confidently extracted without being fully verified.

## 2.1. Confidence in Facts

The confidence of a fact represents the degree to which the system accepts the assertion as true. Fact confidence may be based on the number of supporting sources, the reliability of those sources, the consistency of the evidence, the presence or absence of contradictory evidence, and the verification rules of the system.

For example, a fact derived from a single informal chat message may have lower confidence than a fact supported by a ticket closure, a server log, a signed document, and a follow-up confirmation. Likewise, a fact extracted from a clear source statement may have higher extraction confidence than one inferred from vague or ambiguous language.

The proposed confidence methodology should use a weighted scoring model based on **Multi-Attribute Utility Theory (MAUT)**, which allows several independently evaluated attributes to contribute to a single normalized utility score (Keeney & Raiffa, 1976). In this model, fact confidence is represented as a score between 0 and 1, where each contributing category is also scored between 0 and 1 and assigned a policy-controlled weight. The five core categories are extraction, source, evidence, truth, and verification.

The total fact-confidence score can be expressed as:

```text
C_fact = (w_extraction * C_extraction) +
         (w_source * C_source) +
         (w_evidence * C_evidence) +
         (w_truth * C_truth) +
         (w_verification * C_verification)
```

where each category score is normalized between 0 and 1, each weight is greater than or equal to zero, and all weights sum to 1. Extraction confidence measures how reliably the system extracted the assertion from the source. Source confidence measures the reliability, authority, and governance status of the source. Evidence confidence measures the strength, quantity, independence, and consistency of supporting evidence. Truth confidence measures the likelihood that the assertion is actually true after considering contradictions, temporal context, known facts, and domain rules. Verification confidence measures whether the assertion has been confirmed by authoritative systems, independent sources, policy rules, or human review.

Weights should be configurable by schema, partition, source class, fact type, and organizational policy. A security event, financial approval, production incident, or informal project note may require different weighting. For example, an organization might assign greater weight to verification confidence for regulated workflows, greater weight to evidence confidence for operational troubleshooting, or greater weight to source confidence when an authoritative system of record is involved. The DBMS should store both the final score and the underlying category scores, weights, evaluators, model versions, policy versions, and supporting evidence so that confidence is auditable rather than opaque.

This separation is important because a fact may be clearly extracted from a source while still being uncertain as a statement about reality. The system may be highly confident that an email says "I installed the server today," but less confident that the server was actually installed unless corroborating evidence exists. A MAUT-style scoring model makes that distinction explicit: high extraction confidence does not automatically imply high truth confidence or high verification confidence.

## 2.2. Precision in Facts

The precision of a fact represents the specificity of the assertion. Precision may apply to time, entity identity, location, measurement, relationship, and semantic meaning.

Fact precision should also be scored using a MAUT-style weighted model. In this context, the score does not measure whether the assertion is true; it measures how specific the assertion is across the dimensions that matter for retrieval, reasoning, and operational reuse. A fact such as "the server was installed in 2019" has low temporal precision, while "the installation completed on March 12, 2019 at 12:34:17 PM CST" has high temporal precision. Similarly, "a server" has lower entity precision than "Server X with hostname app-prod-01," and "related to Project Y" has lower relationship precision than "installed specifically to host Project Y's production application."

The total fact-precision score can be expressed as:

```text
P_fact = (w_temporal * P_temporal) +
         (w_entity * P_entity) +
         (w_relationship * P_relationship) +
         (w_location * P_location) +
         (w_semantic * P_semantic) +
         (w_scope * P_scope)
```

where each precision score is normalized between 0 and 1, each weight is greater than or equal to zero, and all weights sum to 1. Temporal precision measures how exact the time reference is. Entity precision measures how specifically the subject or object is identified. Relationship precision measures how clearly the fact defines the connection between entities. Location precision measures the specificity of physical, logical, or organizational location. Semantic precision measures how clearly the action or state is described. Scope precision measures how narrowly the fact identifies the affected system, team, project, customer, environment, or use case.

Weights should be configurable by fact type, schema, partition, and query context. A compliance fact may require high temporal and scope precision, while an architectural lesson may rely more heavily on semantic and relationship precision. The DBMS should store the aggregate precision score, the category-level precision scores, and any categorical labels such as year-level, date-level, hostname-level, rack-level, project-level, or environment-level precision. A fact should therefore be able to carry both confidence and precision as separate MAUT-derived values. For example, the system may store a confidence score of 0.82 while also recording a fact-precision score of 0.74, with temporal precision at date-level and entity precision at hostname-level.

## 2.3. Confidence in Memories

The confidence of a memory represents the reliability of the memory as a contextual object. This is more complex than fact confidence because memories are often derived from multiple claims, facts, sources, and interpretations.

A memory may have high confidence if it is supported by several verified facts and consistent source materials. It may have lower confidence if it is based on incomplete evidence, inferred relationships, uncertain timelines, or ambiguous source language.

For example:

> "The team installed Server X in March 2019 to support Project Y's production migration."

This memory contains several components:

Server X was installed; the installation occurred in March 2019; the installation was related to Project Y; the purpose was to support a production migration; and the work was performed by or for a particular team.

Each component may have its own confidence level. The server installation date may be strongly supported by logs, while the purpose of the installation may be inferred from meeting notes and emails. The memory as a whole should therefore have an aggregate confidence score, but it should also preserve confidence at the component level.

Memory confidence should therefore use a MAUT-style aggregate score that combines the confidence of the supporting facts with the coherence of the memory as a contextual object. The score should account for the reliability of the facts used to construct the memory, the consistency of the underlying claims, the diversity and independence of supporting sources, the confidence of any inference required to connect the facts, the coherence of the timeline, the presence or absence of contradictions, and the degree to which the memory remains current or stale.

The total memory-confidence score can be expressed as:

```text
C_memory = (w_facts * C_supporting_facts) +
           (w_consistency * C_claim_consistency) +
           (w_diversity * C_source_diversity) +
           (w_inference * C_inference) +
           (w_temporal * C_temporal_coherence) +
           (w_contradiction * C_contradiction_status) +
           (w_staleness * C_staleness_status)
```

where each category score is normalized between 0 and 1, each weight is greater than or equal to zero, and all weights sum to 1. Supporting fact confidence represents the weighted reliability of the facts used to construct the memory. Claim consistency measures whether the underlying claims agree or conflict. Source diversity measures whether the memory is supported by independent source classes rather than repeated copies of the same assertion. Inference confidence measures how reliable the derived connection is when the memory includes meaning that was not directly stated. Temporal coherence measures whether the timeline of events is internally consistent. Contradiction status measures whether later evidence challenges the memory. Staleness status measures whether the memory remains operationally useful for current tasks.

The weights should vary by memory type and use case. A safety-critical procedure may place high weight on contradiction and staleness status, while historical recall may place greater weight on supporting fact confidence, temporal coherence, and source diversity. As with fact confidence, the DBMS should preserve the category scores, weights, policy versions, source evidence, and model or human evaluators used to compute the aggregate value.

This allows the memory DBMS to distinguish between a strongly verified memory, a plausible but uncertain memory, a disputed memory, and a memory that was once useful but is now stale.

## 2.4. Precision in Memories

The precision of a memory represents how specifically the memory describes the event, decision, discovery, or institutional context. A memory may be precise in some dimensions and imprecise in others.

For example:

"The infrastructure team installed a dedicated production server sometime in early 2019 because the shared hosting environment was no longer sufficient."

This memory may have moderate confidence and low-to-moderate precision. It identifies the general event, actor, period, and reason, but lacks an exact date, server identifier, and supporting details.

A more precise memory would be:

"On March 12, 2019, the infrastructure team installed Server X as the dedicated production host for Project Y after deciding that the shared hosting environment could not support expected application load."

This memory is more precise because it identifies the date, actor, server, project, purpose, and decision context. However, as with facts, greater precision does not automatically imply greater accuracy or confidence. The precision must be supported by evidence.

Memory precision should therefore be evaluated across several dimensions rather than treated as a single descriptive label. Temporal precision describes how specifically the memory places events on a timeline. Actor precision identifies how clearly the memory names the people, teams, systems, services, or AI agents involved. Entity precision captures whether the remembered objects, systems, documents, or resources are identified generically or specifically. Causal precision describes how clearly the memory explains why something happened, while decision precision captures what was decided, by whom, and under what conditions. Procedural precision describes how specifically the memory explains how work was performed, and contextual precision describes how clearly the memory connects the event to projects, goals, constraints, risks, dependencies, or outcomes.

This is especially important for AI agents. An agent may not need a perfect historical narrative to complete a task, but it may need highly precise operational guidance. Conversely, a human reviewing an old project may need broader contextual recall even when some operational details are no longer valid.

# 3. Memory Lifespan and Staleness

Unlike static records, memories have a lifespan. Some memories remain useful indefinitely as historical knowledge. Others become stale when systems change, people leave, policies evolve, projects end, or newer facts supersede older ones (Kulkarni & Michels, 2012).

The lifecycle state of a memory should therefore be explicit. A memory may be **current** when it remains valid for present use, or **historical** when it is true as a record of the past but no longer safe to treat as current guidance. It may become **stale** when it is potentially outdated, **superseded** when a newer memory, fact, policy, or decision replaces it, or **contradicted** when later evidence challenges it. Some memories may be **consolidated** into a higher-level summary, workflow, procedural memory, or skill, while others may become **decayed** when they are retained but assigned lower retrieval priority because they have not been reinforced. Older or lower-priority memories may be **archived** into lower-cost or lower-priority storage, **retired** from operational use while preserved for historical recall, or **pruned** only when policy, legal holds, provenance requirements, and audit rules permit deletion.

For example:

"Server X is the production server."

This memory may have been current in 2019. If production later moved to Server Z in 2022, the original memory remains historically meaningful but becomes stale or superseded as operational guidance.

A memory DBMS must therefore distinguish between:

"What was true then?"

and:

"What is valid now?"

This is one of the central requirements for human-AI knowledge sharing. AI agents must not simply retrieve the most semantically similar memory. They must retrieve memories with awareness of time, validity, confidence, precision, salience, and current applicability (Lewis et al., 2020; Liu et al., 2024; Packer et al., 2023).

Long-lived memory systems also need lifecycle management that goes beyond staleness. A memory may remain historically true while becoming less useful for routine recall. Other memories may be consolidated into a higher-level semantic summary, workflow, procedural memory, or skill. The DBMS should therefore support salience decay, reinforcement, consolidation, archival movement, and policy-governed pruning as explicit lifecycle operations rather than allowing the corpus to accumulate forever with equal retrieval weight (Park et al., 2023; Alqithami, 2025).

## 3.1. Bitemporal Valid-Time and Transaction-Time Model

The temporal model should be formalized as a **bitemporal** model that separates when a fact or memory was valid in the real world from when the DBMS recorded, accepted, corrected, or retired that knowledge. This distinction matters because institutional memory is often discovered late, corrected after the fact, or reinterpreted when additional source evidence becomes available (Kulkarni & Michels, 2012).

At minimum, governed facts, memories, workflows, procedural memories, and derived claims should carry temporal fields with distinct meanings. The `event_time` records when the remembered event or source event occurred, if known. The valid-time fields, `valid_time_start` and `valid_time_end`, define the period during which the assertion, memory, workflow, or procedural memory applies to the represented world. The transaction-time fields, `transaction_time_start` and `transaction_time_end`, define when the DBMS recorded, accepted, revised, or retired a specific version. The `source_time` preserves the timestamp supplied by the original source, such as an email, ticket, log, document, or API payload, while `verification_time` records when the system, policy, or human verifier accepted, rejected, or revised the object. Finally, `stale_after` identifies the time or condition after which the object should be reviewed before operational use.

This model allows the DBMS to answer different temporal questions without confusing them. A valid-time query asks what was true or applicable for the represented world at a given time, such as "Which server was production on March 24, 2023?" A transaction-time query asks what the DBMS believed or had recorded at a given time, such as "What did the system know on April 1, 2023?" A bitemporal query combines both, such as "As recorded on April 1, 2023, what did the organization believe was true on March 24, 2023?"

Corrections should not overwrite history. If a source later shows that all A100 GPUs were removed from the server room on March 24, 2023, the DBMS should close the valid-time window for any earlier fact claiming that three A100 GPUs were present after that removal. The prior fact remains transactionally visible as something the DBMS once recorded, but it should no longer be returned as current valid knowledge for dates after the removal event. This lets humans and AI agents distinguish "what happened," "what we believed at the time," and "what we now know was valid."

Operationally, bitemporal retrieval should expose explicit query modes rather than relying on implicit interpretation. A query may request current valid knowledge, historical valid knowledge, knowledge as recorded at a transaction time, or a full bitemporal audit trail. These modes should apply consistently across exact lookup, semantic search, workflow retrieval, active memory pool loading, and API access.

The component responsible for this behavior is the **Temporal Supersession Engine**. Its role is to apply valid-time and transaction-time rules whenever a claim becomes a fact, a fact changes, a memory is corrected, a workflow is replaced, or a procedural memory becomes unsafe to use. It should close validity windows, create supersession edges, preserve the prior transaction-time record, update staleness status, and notify derived artifacts such as embeddings, workflow summaries, active memory pools, and skill packages that they may need review or rebuild. The engine should never erase the prior state merely because a newer state is now preferred; it should make the supersession explicit and queryable.

# 4. Deriving Memories from Facts and Claims

Many memories will not be entered directly by humans. They will be derived from existing information. A memory DBMS must therefore support memory construction from documents, conversations, tickets, logs, databases, and other historical artifacts (Stein & Zwass, 1995; Buneman et al., 2001; Moreau & Missier, 2013).

Just as important, the system should preserve the original source artifact in verbatim form whenever policy, law, and retention constraints permit. A document, work order, ticket, transcript, or log capture should remain addressable as the authoritative source package from which later claims, facts, and memories can be derived. This allows every derived memory to be traced back to the exact source evidence that supported it, while also preserving the option to revisit the source later and extract additional information that was not considered important during the first ingestion pass (Buneman et al., 2001; Moreau & Missier, 2013; Consultative Committee for Space Data Systems, 2012).

The DBMS should record this chain in a **Derivation Ledger**. The ledger is the auditable lineage structure that links source packages to extracted claims, accepted facts, Episode Objects, workflow traces, generalized workflows, procedural memory objects, skill packages, embeddings, summaries, and model-generated artifacts. It should record which parser, model, prompt template, policy, verifier, or human action created or revised each derived object. This prevents derivation from becoming invisible application logic and gives the DBMS a durable way to replay, audit, or regenerate memory objects as models and schemas change.

In practice, derivation does not need to rely on one model type or one extraction strategy. Deterministic parsers, regular-expression or schema-driven extractors, embedding models, small local language models, larger LLMs, and custom domain models may all participate at different stages. Stable fields such as dates, identifiers, work-order numbers, and inventory counts may often be extracted or validated deterministically, while free-text descriptions of responsibility, approval, rationale, exception handling, or procedural detail may benefit from model-assisted extraction, classification, or summarization. All such model-generated outputs should remain provisional until they are linked to source evidence and, where necessary, verified by policy or human review.

For example, the system may ingest the following sources:

1.  An email saying, "I am installing the server today."

2.  A ticket marked "server installation completed."

3.  A log entry showing the server came online at 12:34 PM.

4.  A meeting note stating that the server was needed for Project Y.

5.  A later document describing the production migration.

From these sources, the system may extract several claims, verify some of them as facts, and derive a memory:

"Server X was installed on March 12, 2019 as part of the Project Y production migration. The installation was completed around 12:34 PM and later became part of the organization's production hosting architecture."

This memory should retain links to the underlying claims, facts, and sources. It should also indicate which parts of the memory are directly supported and which parts are inferred. For example, the timestamp may be supported by a system log, while the purpose of the installation may be inferred from meeting notes and project documents.

A second example is a work order or installation report confirming that a server installation was completed. The source document may contain several distinct assertions that can be separated and preserved individually. From one verbatim work order, the system might derive facts or strongly supported claims such as:

-   "The server was installed on March 24, 2014."

-   "The server was installed by ACME Installations."

-   "The server was installed by John Smith."

-   "Mary Jones verified and approved the installation."

Those same facts may later contribute to one or more memories, such as:

-   "The organization used ACME Installations to complete the March 24, 2014 server deployment."

-   "The installation required formal verification and approval before the server was accepted into service."

Not all claims, facts, or memories need to be derived at the moment the source first enters the system. An organization may initially extract only the date and completion status, then return later to the same verbatim work order to derive contractor involvement, individual responsibility, approval history, workflow steps, or compliance-relevant context when those details become operationally important. This deferred derivation model is one reason raw source preservation matters: it allows the system to deepen its understanding of past evidence without losing traceability to the original document (Consultative Committee for Space Data Systems, 2012; Sculley et al., 2015).

This same deferred model should also allow the DBMS to reopen earlier conclusions. A newly derived memory may reveal that an earlier fact should be contradicted, superseded, or given a narrower validity window. If later source interpretation shows that hardware was removed, access was revoked, a system was decommissioned, or an approval was reversed earlier than previously believed, the system should reevaluate the affected facts rather than leaving them permanently marked as current.

![Figure 1: Source-to-Memory Derivation Flow](Malu-Figure-4.0.pdf)

This derivation path is critical. Without it, a memory becomes an unsupported summary. With it, the memory becomes inspectable, verifiable, correctable, expandable, and reusable by both humans and AI agents (Buneman et al., 2001; Cheney et al., 2009; Moreau & Missier, 2013).

# 5. Organizing Memories: The Subject-Verb-Predicate-Object-Relationship Model

The database must organize memories in a way that supports efficient retrieval, long-term maintenance, scalability, and parallelism across hardware resources. Institutional memory may contain millions or billions of memories, facts, claims, documents, conversations, workflow traces, decisions, events, and relationships. If every query requires a global semantic search across all stored memories, the system becomes expensive, slow, and difficult to govern. A scalable memory DBMS therefore requires an organizational structure that can compartmentalize resource-intensive search operations, distribute work across available CPU, GPU, storage, and network resources, and still allow memories to be connected across people, projects, systems, time periods, and domains (Walsh & Ungson, 1991; Argote & Ingram, 2000; Ko et al., 2007; Galan, 2023).

We propose a hybrid organizational model inspired by English grammar and extended with database-native relationship structures:

Subject\
└── Verb\
└── Predicate\
└── Object\
↔ Relationship

This structure provides a semantic frame for organizing memories. It does not replace relational indexes, vector indexes, document storage, time-series structures, or graph traversal. Instead, it acts as an organizing layer that helps the DBMS route queries into smaller, meaningful compartments before applying more expensive retrieval operations such as vector similarity search, relationship traversal, temporal analysis, or document reconstruction.

The same grammar-inspired structure should also participate directly in vector embedding and semantic search. When a memory chunk, source excerpt, workflow trace, or derived summary is embedded, the DBMS should prepend or otherwise include the relevant subject, verb, predicate, and object frame in the text submitted to the embedding model. Query-time retrieval should use the same structure: when the system can extract or infer the subject, verb, predicate, object, topic, time window, or relationship hints from a user prompt, those fields should be included in the search vector or coordinated retrieval package. Aligning the embedded memory chunks and the query vectors around the same semantic frame should improve retrieval precision, reduce ambiguous matches, and help ensure that the most relevant authorized vectors are ranked ahead of textually similar but contextually unrelated memories.

## 5.1. Subject

The **Subject** represents the primary entity around which a memory is organized. A subject may be a person, company, project, organization, customer, department, system, server, location, document, product, AI agent, or other identifiable thing.

Examples of subjects include:

| Subject Type | Example |
| --- | --- |
| Person | Alice Johnson |
| Project | Project Y |
| System | Server X |
| Organization | Kinetic Seas Incorporated |
| Customer | Customer A |
| Place | Chicago data center |
| AI Agent | Infrastructure support agent |
| Document | Migration plan v2 |
| Application | Billing platform |

The subject provides the first level of compartmentalization. A query about Server X should not initially require searching all memories in the enterprise. The DBMS can first restrict the search to memories where Server X is the primary subject, a related subject, or an object connected to another subject.

For example:

"What do we know about Server X?"

can be routed first to the Server X subject compartment, where the system can then search facts, claims, events, decisions, workflows, documents, and relationships connected to that subject.

## 5.2. Verb

The **Verb** represents the type of action, state, or activity associated with the subject. It describes what happened, what was done, what was discussed, what changed, or what was learned.

Examples of memory verbs include actions and states such as `discussed`, `decided`, `discovered`, `planned`, `installed`, `configured`, `purchased`, `reviewed`, `changed`, `failed`, `approved`, `rejected`, `migrated`, `learned`, and `verified`. These verbs let the DBMS classify memories by the type of activity or assertion they represent: a decision, a discovery, a system change, a failure, an approval, a migration, a lesson, or a verification event.

The verb level allows the system to narrow the search by action class. For example:

"What decisions were made about Project Y?"

can be routed to:

| Field | Value |
| --- | --- |
| Subject | Project Y |
| Verb | decided |

This is more efficient than searching every memory related to Project Y. It also improves retrieval quality because the search is scoped to decision memories rather than all documents, messages, and events.

A search that contains both a subject and a verb will usually be more efficient than a search by verb alone because the subject narrows the candidate set before the DBMS evaluates action class, predicate, relationship, time, and semantic similarity. A verb-only search such as "show all failed deployments" is still important, but it can be much broader and more expensive in a large enterprise corpus. The architecture should therefore explore additional strategies for efficient verb-oriented retrieval, including verb-level indexes, subject-type partitions, time-window constraints, compact routing indexes, workflow-family filters, and reranking methods that can identify the most relevant action-class memories without requiring an undifferentiated global search.

## 5.3. Predicate

The **Predicate** represents the structured meaning around the verb. In grammar, the predicate says something about the subject. In the memory DBMS, the predicate provides a normalized semantic frame that describes the assertion, event, decision, discovery, or relationship being remembered.

For example, in the memory:

"On March 12, 2019, the infrastructure team installed Server X as the dedicated production host for Project Y after deciding that the shared hosting environment could not support expected application load."

The subject may be:

Subject: Server X

The verb may be:

Verb: installed

The predicate may capture structured meaning such as:

| Field | Value |
| --- | --- |
| installed\_as | dedicated production host |
| installed\_for | Project Y |
| installed\_by | infrastructure team |
| reason | shared hosting environment could not support expected load |
| event\_date | March 12, 2019 |
| outcome | successful |

The predicate is important because it makes memories more than text. It provides a structured representation that can be queried, indexed, validated, and connected to facts, claims, evidence, workflows, and timelines.

The predicate may include the role a system or person played, the purpose of an action, the actor responsible for it, the reason it occurred, the outcome it produced, the relevant time, the evidence supporting it, and the confidence, precision, and validity status assigned to that interpretation. In the server-installation example, the predicate may say that Server X was installed as a dedicated production host, for Project Y, by the infrastructure team, because shared hosting could not support expected load, with a successful outcome on March 12, 2019, supported by tickets, email, logs, and meeting notes.

Search by predicate without a subject and verb will usually be very inefficient. Predicate elements such as purpose, reason, outcome, actor, role, and validity occur across many unrelated memories, projects, systems, and workflows. A query that asks only for a predicate such as "reason: performance" or "outcome: successful" may require broad scans, large vector searches, or heavy reranking unless it is constrained by subject, verb, partition, time window, workflow family, or security scope. The predicate is therefore most powerful when used as part of a structured retrieval path, such as subject plus verb plus predicate, rather than as an isolated global search field.

In this sense, the predicate is the bridge between natural-language memory and structured database representation.

## 5.4. Object

The **Object** represents the thing being acted on, remembered, described, or affected. The object may be another entity, a document, a system, a decision, a configuration, a workflow, a claim, a fact, or the memory payload itself.

In some cases, the subject and object are both entities:

| Field | Value |
| --- | --- |
| Subject | Infrastructure Team |
| Verb | installed |
| Object | Server X |

In other cases, the subject is the main entity being remembered:

| Field | Value |
| --- | --- |
| Subject | Server X |
| Verb | installed |
| Object | Dedicated production host role |

The object level may also contain or link to the memory payload. However, the payload should not be treated as an unstructured blob only. A memory payload may combine a natural-language summary with structured fields such as extracted entities, dates, actors, outcomes, source references, supporting claims, supporting facts, confidence, precision, validity period, staleness status, workflow traces, and embedding vectors. In this sense, objects can behave like documents in a document database because they may carry rich payloads, metadata, and source references, while also behaving like nodes in a graph database because they can participate in typed relationships across the larger memory structure.

Those object relationships do not need to remain inside a single subject or verb path. An object may connect memories across subjects, verbs, projects, time periods, workflows, skills, and security partitions when the relationship is meaningful and authorized. The object layer may also include durable pointers back to raw source documents, tickets, transcripts, logs, database records, or archive files, allowing derived memories to remain traceable to the evidence from which they were created.

The object layer is where the DBMS can attach the actual memory content, but the system should also preserve structured fields and relationships so that the memory can be maintained and retrieved efficiently.

### 5.4.1. Memory Detail Objects and Recursive Detail

Not every useful memory exists at the same level of granularity. A memory such as "installed an Ubuntu 24.04 server on Proxmox" may be useful as a concise historical statement, but it may also contain operational details such as downloading the ISO, uploading it to Proxmox, creating a virtual machine, assigning CPU and memory, configuring storage, selecting a network bridge, installing the operating system, hardening access, and validating the result. Each of those details may contain still more detail, including URLs, commands, configuration choices, tool versions, screenshots, log excerpts, approvals, validation checks, and exceptions.

The DBMS should therefore support **Memory Detail Objects** as addressable child or linked objects beneath a memory, Episode Object, workflow trace, procedural memory, or skill. A Memory Detail Object may represent a step, substep, parameter, command, configuration value, source excerpt, decision point, validation check, exception, warning, artifact, or evidence item. It should not be stored only as paragraph text inside the parent memory, because humans and AI agents may need to retrieve, inspect, cite, reorder, reuse, or validate the detail independently.

This leads to the concept of **recursive detail**. A Memory Detail Object should be able to contain or link to additional Memory Detail Objects, allowing the DBMS to expand a memory only to the level required by the task. A broad historical question may need only the parent memory. A troubleshooting question may need the relevant step and its exception history. A skill execution may need the complete ordered procedure, including commands, parameters, validation checks, fallback paths, and source evidence.

For example:

```text
Episode Object: installed Ubuntu 24.04 server on Proxmox
  Detail: downloaded Ubuntu Server ISO
    Detail: source URL
    Detail: checksum or signature verification
    Detail: selected release and architecture
  Detail: uploaded ISO to Proxmox
    Detail: target storage location
    Detail: upload method
  Detail: created new virtual machine
    Detail: CPU, memory, disk, and firmware settings
    Detail: network bridge and VLAN settings
    Detail: boot order and installation media
```

This structure uses document-database ideas because each memory may contain nested, richly described detail. It also requires graph-database behavior because a detail should be reusable across memories, workflows, skills, subjects, and projects. The detail "uploaded ISO to Proxmox" might appear in Ubuntu installations, Debian installations, disaster-recovery drills, VM migration workflows, and troubleshooting memories. It should therefore be represented as an addressable object with relationship edges, not only as a nested string.

Retrieval should support progressive expansion. A query or API call may ask for the parent memory only, for the first level of details, for all details to a specified depth, for only details related to a subject or exception, or for the evidence behind a particular step. This allows the DBMS to keep ordinary recall concise while still supporting the deep operational detail needed to build and execute procedural memories, skills, and competencies.

## 5.5. Relationship

The **Relationship** layer connects memories, Memory Detail Objects, facts, claims, subjects, objects, workflows, documents, and time periods. This is where the grammar metaphor should be extended into a database-native graph model.

The term **Relationship** is preferable to **Preposition** because the DBMS must support more than natural-language prepositional links. Prepositions such as **about**, **before**, **after**, **inside**, **from**, **toward**, and **with** are useful, but enterprise memory also requires relationships such as **supports**, **contradicts**, **supersedes**, **derived\_from**, **verified\_by**, **caused\_by**, **depends\_on**, and **part\_of**.

The relationship model should distinguish explicitly between broad association and causality. A `related_to` edge means two objects are connected or worth retrieving together. A `caused_by` edge should mean that the system, a policy, or a human reviewer has accepted a stronger causal assertion with evidence, mechanism, confidence, and scope. This distinction prevents the graph from implying that events caused one another merely because they appeared in the same project, time window, incident, or workflow trace.

Relationship types should cover the major ways institutional knowledge connects across the system. A memory may be `about` a subject, object, or topic; occur `before` or `after` another memory or event; arise `because_of` another fact, decision, or condition; or be generally `related_to` another object when the system only needs a broad semantic association. Other relationships describe containment, origin, and participation, such as `inside` a project, department, system, or place; `from` a source artifact; `with` a person, team, system, or organization; or `has_detail` and `contains` relationships that connect parent memories to recursive detail objects. Governance-oriented relationships should also be available, including `supports`, `contradicts`, `supersedes`, `derived_from`, and `verified_by`, so claims, facts, memories, workflows, and skills can be traced, challenged, promoted, or replaced over time. Operational relationships such as `depends_on`, `caused_by`, and `part_of` allow the system to represent process structure, causal assertions, and membership in a larger project, workflow, incident, or timeline.

Causal relationships should carry additional metadata, such as causal mechanism, evidence links, confidence, temporal ordering, candidate counterfactual, reviewer or model source, and whether the causal edge is asserted, inferred, disputed, or verified. The initial DBMS may store and query these causal edges without claiming to solve full causal inference. Later versions may add a causal reasoning subsystem that derives candidate causal chains, evaluates interventions, or answers counterfactual questions using the stored evidence and typed graph (Jaimini & Sheth, 2022; Zellinger et al., 2024).

Relationships allow the memory DBMS to move beyond hierarchical storage. A memory may be stored primarily under one subject, verb, predicate, and object, but it may also be connected to many other memories through relationship edges. This prevents the system from becoming trapped in a rigid folder structure.

For example, the server installation memory may be organized as:

| Field | Value |
| --- | --- |
| Subject | Server X |
| Verb | installed |
| Predicate: installed\_as | dedicated production host |
| Predicate: installed\_for | Project Y |
| Predicate: installed\_by | infrastructure team |
| Predicate: reason | shared hosting capacity limitation |
| Predicate: outcome | successful |
| Object: memory\_payload | server installation event |
| Object: workflow\_trace | server installation steps |
| Object: recursive\_details | ISO download, VM creation, network configuration, validation checks |
| Object: source\_documents | email, ticket, log, meeting note |
| Relationship: after | decision to leave shared hosting |
| Relationship: because\_of | expected application load |
| Relationship: with | infrastructure team |
| Relationship: inside | Project Y |
| Relationship: from | ticket \#9021 |
| Relationship: verified\_by | deployment log |
| Relationship: related\_to | production migration |
| Relationship: supports | server installation workflow |

This makes the memory useful for multiple kinds of retrieval:

-   historical recall;

-   operational troubleshooting;

-   project timeline reconstruction;

-   recursive detail expansion;

-   workflow derivation;

-   evidence inspection;

-   AI-agent guidance;

-   edge-case reasoning.

# 6. Implications for a Memory DBMS

The distinction between facts, claims, and memories has direct implications for database design. A memory DBMS should not merely store text chunks and embeddings. It should provide native structures for representing assertions, evidence, confidence, precision, time, provenance, relationships, and staleness (Moreau & Missier, 2013; Kulkarni & Michels, 2012; Hogan et al., 2021).

At minimum, the system should preserve the full evidentiary path from extracted assertions to accepted knowledge. It should store claims separately from facts, manage accepted facts with their supporting evidence, and represent memory objects as contextual records of events, decisions, discoveries, procedures, warnings, assumptions, and lessons. Because these objects differ in reliability and specificity, the DBMS should model confidence and precision separately for claims, facts, and memories. It should also preserve provenance links back to original sources, support bitemporal validity so the system can distinguish historical truth from current operational guidance and prior DBMS belief, detect staleness, track contradictions, and version knowledge objects as the organization learns more.

The same foundation should extend beyond recall into operational learning. Detailed memories should support workflow derivation, and similar traces should be consolidated into generalized workflows without severing links to the underlying source evidence. Generalized workflows and supporting memories can then contribute to procedural memory formation, skill formation, and competency packaging for reuse by humans and AI agents. The system should also track outcomes, distinguish successful, failed, partial, and obsolete executions, retrieve edge cases that help resolve unusual current situations, and detect when workflows or procedural memories may no longer be valid because tools, systems, policies, or operating conditions have changed.

This design allows the system to answer questions that conventional databases, vector stores, and document repositories do not answer well:

-   What do we know?

-   How do we know it?

-   How confident are we?

-   How precise is the knowledge?

-   When was it true?

-   Is it still valid?

-   What evidence supports it?

-   What claims contradict it?

-   Which memory replaced it?
-   Can an AI agent safely use it for a current task?

# 7. Deriving Workflows, Procedural Memories, Skills, and Competencies from Memories

A memory DBMS should not only preserve what happened; it should also preserve and derive knowledge about how outcomes were achieved. That operational knowledge has several layers. A **workflow trace** records the observed sequence of actions in one event. A **generalized workflow** abstracts repeated patterns across similar traces. A **procedural memory** records retained how-to knowledge about how to perform, adapt, validate, and repair a task. A **skill** or **competency** packages validated procedural memory for reuse by humans or AI agents (Cohen & Squire, 1980; Squire, 2004; van der Aalst et al., 2004; IEEE Task Force on Process Mining, 2012).

Many institutional memories describe completed events that resulted from a sequence of actions. If the memory is sufficiently detailed, the system should be able to link the event to the step-by-step workflow trace that produced it (van der Aalst et al., 2004; IEEE Task Force on Process Mining, 2012).

For example, the memory:

"On March 12, 2019, the infrastructure team installed Server X as the dedicated production host for Project Y after deciding that the shared hosting environment could not support expected application load."

describes an event, but it may also imply or link to a workflow trace. The installation of Server X may have required selecting hardware or a virtual machine, installing the operating system, configuring networking, applying security patches, creating user accounts, installing application dependencies, configuring services, deploying application code, validating connectivity, and transferring production traffic. If these steps are preserved or reconstructed from tickets, commands, logs, documentation, and conversations, the memory becomes more than a historical record. It becomes one execution instance that can support both a generalized workflow and procedural memory.

This creates an important distinction among **Episode Object**, **workflow trace**, **generalized workflow**, and **procedural memory**. An Episode Object, corresponding conceptually to episodic memory, records that something happened as a specific episode in time and context. A workflow trace records the steps observed in that episode. A generalized workflow describes a repeatable process pattern derived from one or more traces. A procedural memory is not the workflow itself; it is capability-oriented memory about how to perform and adapt the work, including prerequisites, judgment points, exceptions, validation checks, tool choices, safety constraints, and recovery tactics. A workflow can support procedural memory, but it does not exhaust it.

A memory DBMS should therefore support the ability to derive candidate workflow traces from individual memories, consolidate generalized workflows across many similar memories, and form procedural memories that preserve reusable know-how without discarding context (Cohen & Squire, 1980; van der Aalst et al., 2004; IEEE Task Force on Process Mining, 2012). A single server installation may provide one example of a process. Many server installation memories, performed by different people across different projects and environments, may allow the system to identify a candidate generalized workflow for installing a server. The system should be able to compare similar events, extract common steps, identify variations, distinguish required steps from optional steps, and preserve known exceptions.

Workflow traces should be built from addressable Memory Detail Objects rather than only from narrative text. A top-level Episode Object may say that an Ubuntu 24.04 server was installed on Proxmox, while its recursive details record how the ISO was obtained, where it was uploaded, how the virtual machine was configured, which commands were run, which validation checks passed, and which exceptions occurred. This allows the same memory to support multiple uses: concise historical recall, detailed troubleshooting, generalized workflow extraction, and skill execution. The DBMS can expand the trace only as far as the task requires, rather than forcing every user or agent to retrieve the entire procedural tree every time.

For example, if the system contains fifty memories involving successful server installations, it may discover that most of them include the following recurring pattern:

1.  define the application or business requirement;

2.  select the target server, virtual machine, or cloud instance;

3.  install or provision the operating system;

4.  configure networking, DNS, firewall, and access controls;

5.  install required runtime dependencies;

6.  configure application services;

7.  deploy application code or database components;

8.  validate connectivity, security, and performance;

9.  document the server and link it to the responsible project;

10. transition the system into operational use.

Recurring patterns like this can be converted into **skills** or **competencies** when they are sufficiently supported, validated, and current. These skills make it possible for both humans and AI agents to perform tasks based on proven workflows while still accessing the supporting memories behind the workflow. That access matters because real operational work often depends on exceptions and edge cases: a human or AI agent using the skill should be able to inspect how similar problems were handled before, which variations succeeded or failed, and which evidence supports the recommended path. Recursive detail makes that inspection practical because the agent can begin with a generalized workflow, expand one uncertain step, retrieve the substeps and exceptions for that step, and then return to the higher-level skill without flooding the active context with unrelated detail.

The resulting workflow should not replace the original memories. Instead, it should be stored as a derived knowledge object linked back to the source memories, facts, claims, and evidence from which it was constructed. This preserves provenance and allows future humans or AI agents to inspect why the workflow exists, which cases support it, which cases contradict it, and under what conditions it should be applied (Buneman et al., 2001; Moreau & Missier, 2013).

This design allows the memory DBMS to support both **specific recall** and **generalized operational knowledge**. Specific recall answers questions such as:

"When was Server X installed, who installed it, and why?"

Generalized workflow knowledge answers questions such as:

"What is the organization's proven process for installing a production server?"

Procedural memory answers a different question:

"How does the organization know how to install, adapt, validate, and troubleshoot production servers in real environments?"

The difference is important. A memory describes an instance. A workflow trace describes the observed steps in an instance. A generalized workflow describes a repeatable pattern derived from one or more instances. A procedural memory describes retained know-how for applying, adapting, validating, and repairing those patterns. A skill or competency is a governed procedural memory packaged for reuse by humans or AI agents.

## 7.1. Using the Organization Model for Workflow Formation

The Subject-Verb-Predicate-Object-Relationship model also supports workflow formation because it gives the DBMS a consistent way to group similar Episode Objects before attempting process discovery. Repeated Episode Objects can be grouped by subject type, verb, object type, predicate structure, outcome, relationship, and available workflow trace. This makes the organization model more than a storage convention: it becomes part of the learning mechanism that helps the DBMS identify reusable operational patterns (van der Aalst et al., 2004; IEEE Task Force on Process Mining, 2012).

For example, the DBMS may identify many memories with the following pattern:

| Field | Value |
| --- | --- |
| Subject Type | Server |
| Verb | installed |
| Object Type | production host |
| Outcome | successful |
| Relationship | inside project |
| Workflow Trace | available |

From these memories, the system can consolidate a candidate generalized workflow or skill, such as installing and validating a production application server. The workflow may be derived from fifty successful server-installation memories and may identify common steps such as provisioning the host, configuring networking, installing dependencies, deploying the application, validating the service, and documenting the environment. It should also preserve known exceptions, such as firewall-rule conflicts, DNS delays, package-version conflicts, or insufficient disk allocation, because those edge cases often determine whether the workflow is practically useful. The resulting workflow should carry confidence, precision, current-validity, review status, and provenance metadata so humans and AI agents can understand how strongly the workflow is supported and whether it remains safe to use under current operating-system versions, security policies, and infrastructure standards.

![Workflow Formation from the Organizational Model](Malu-Figure-2.pdf)

*Figure 2 - Workflow Formation from the Organizational Model.*

This allows the memory DBMS to help transform repeated institutional events into reusable procedural memories, skills, and competencies. The organizational structure is therefore not only a storage model; it is also a learning model.

## 7.2. Workflow and Procedural Memory Confidence and Precision

As with facts and memories, workflows should have confidence and precision. A workflow may be highly detailed but poorly supported, or broadly supported but imprecise.

**Workflow confidence** represents how reliable the system believes the workflow is. It should be based on the number of supporting memories, the quality of the outcomes those memories describe, the diversity of supporting sources across people, teams, systems, and projects, and the consistency of the steps observed across multiple executions. The DBMS should also consider evidence strength, including whether the workflow steps are supported by logs, tickets, documents, commands, or direct observation; contradiction status, including whether other memories show that the workflow failed or is incomplete; and current validity, including whether the workflow remains appropriate for present systems, policies, tools, and operating environments.

**Workflow precision** represents how specifically the workflow describes the process. A low-precision workflow may say:

"Provision the server, configure the application, and validate the deployment."

A higher-precision workflow may say:

"Provision an Ubuntu 24.04 virtual machine with 8 GB RAM, assign a static IP address, configure DNS, install Apache, PHP, and MariaDB, harden SSH access, enable the firewall, deploy the application to /var/www/html, configure Apache virtual hosts, test HTTPS, and document the server in the project inventory."

The second workflow is more precise, but it is not necessarily more reliable unless supported by evidence. Therefore, workflow precision should be modeled separately from workflow confidence.

Procedural memories should also have confidence and precision, but those values should not be copied mechanically from the related workflow. A workflow may be precise because it lists exact steps, while the procedural memory may still be weak if it lacks validated exception handling, current tool compatibility, recovery guidance, or evidence that the process works across real environments. Procedural-memory confidence should consider supporting outcomes, operational diversity, exception coverage, validation checks, and current applicability. Procedural-memory precision should describe how specifically the DBMS can apply the know-how, including preconditions, decision branches, tool versions, safety limits, fallback paths, and evidence links.

## 7.3. Procedural Memory, Skills, and Competencies

When a generalized workflow has sufficient evidence, successful outcomes, and current operational validity, it may contribute to a **procedural memory**. Procedural memory is distinct from a workflow. A workflow is an explicit process artifact; procedural memory is the durable how-to knowledge that allows a human or AI agent to perform the task, recognize when the standard workflow applies, adapt when the situation differs, validate the outcome, and recover from known failure modes.

When procedural memory is sufficiently validated and packaged for reuse, it may become a **skill** or **competency**. In this context, a skill is not merely a document or checklist. It is a governed, evidence-backed procedural memory that can guide humans or AI agents through a repeatable task while preserving the provenance of the events, traces, and workflows from which it was learned.

A procedural memory or skill should describe what the skill is intended to accomplish, when it applies, what must be true before execution begins, and which inputs, credentials, tools, or resources are required. It should connect to related standard, variant, and fallback workflows; preserve ordered steps, recursive detail objects, and decision points; define validation checks and expected outputs; and identify known exceptions, failure modes, recovery tactics, supporting memories, and staleness indicators that may make the skill unsafe or outdated.

This is especially important for AI agents. An AI agent should be able to retrieve a memory to answer a historical question, a workflow to understand a process pattern, and a procedural memory or skill to perform a current task. When a procedural memory has high confidence and high precision, fewer additional calls to the memory DBMS or external sources should be required because the skill already packages the relevant purpose, preconditions, steps, validation checks, exception guidance, supporting memories, and current-applicability constraints. Lower-confidence or lower-precision procedural memories should have the opposite effect: they should cause the agent to retrieve additional memories, expand recursive details, inspect edge-case histories, consult external sources the agent is authorized to use, or request human review before acting.

For example, an AI infrastructure agent asked to install a new server may use the server installation procedural memory or skill as its primary guide. If it encounters an unusual network configuration, it may retrieve prior memories involving similar network exceptions. In this way, memories support both routine execution and edge-case reasoning.

## 7.4. From Episode Objects to Operational Knowledge

Modern AI tools are changing how operational knowledge must be packaged for AI agents, and the requirements vary across tools, model families, agent frameworks, and deployment contexts. For that reason, the rules used to convert Episode Objects into operational knowledge should be configurable within the DBMS rather than hard-coded into a single extraction pipeline. The system should support policy-governed transformation templates and system prompts that define how an LLM or task-specific model should derive workflow traces, procedural memories, skills, exception guidance, and validation rules from Episode Objects. This is one of the areas where LLMs with system prompts designed for the task should be employed, with all generated outputs remaining linked to source evidence, confidence, precision, and review policy.

Recent work on Recursive Language Models suggests that LLMs can process long and information-dense inputs more effectively when context is treated as an external environment that can be inspected, decomposed, and recursively queried rather than flattened into a single prompt. For a memory DBMS, the parallel design principle is that operational memory should remain addressable as recursive, source-linked detail objects. Humans and AI agents can then expand only the details needed for the current task, skill execution, verification step, or exception path instead of loading an entire historical corpus into context (Zhang et al., 2026).

The transformation from memory to procedural capability can be understood as a layered process:

| Layer | Description |
| --- | --- |
| **Source evidence** | Emails, tickets, logs, documents, transcripts, commands |
| **Claims** | Assertions extracted from the source evidence |
| **Facts** | Verified or accepted assertions |
| **Episode Objects** | Contextual records of what happened |
| **Workflow traces** | Step-by-step sequences linked to specific Episode Objects |
| **Generalized workflows** | Consolidated processes derived from multiple similar events |
| **Procedural memories** | Capability-oriented how-to knowledge supported by workflows, exceptions, validation rules, and outcomes |
| **Skills or competencies** | Governed procedural memories packaged and validated for reuse by humans or AI agents |

This layered model allows the system to move from raw institutional artifacts to reusable operational intelligence without losing provenance. Each higher-level object remains connected to the lower-level evidence that supports it.

A key design principle follows:

A memory DBMS should preserve individual events as historical memories while also allowing repeated events to be consolidated into workflows, procedural memories, skills, and competencies that guide future human and AI action.

## 7.5. Expanded Conceptual Model

The proposed database system expands traditional database systems by producing functional outputs, including workflow traces, generalized workflows, procedural memories, skill packages, competencies, and the ability to design autonomous or semi-autonomous agents based on defined skills, agent goals, policy constraints, and real-time access to related memories. In this model, the DBMS is not only a repository that stores records; it is a governed knowledge system that can transform historical evidence into operational guidance while preserving the lineage, confidence, precision, and validity rules needed for safe reuse.

The expanded conceptual model therefore begins with sources such as emails, documents, chats, logs, and tickets. Sources produce claims, claims may become accepted facts, and related facts and claims may become contextual memories or Episode Objects. From sufficiently detailed Episode Objects, the DBMS can derive workflow traces that record how specific work was performed, consolidate similar traces into generalized workflows, and then form procedural memories that explain how to perform, adapt, validate, and troubleshoot the work. When procedural memories are sufficiently validated, they can be packaged into skill packages or competencies for humans and AI agents. Evidence, provenance, and staleness remain attached throughout this chain so that a skill such as "install and validate a production application server" remains traceable back to the source records, claims, facts, memories, and workflow traces that justify it, and can be marked for review when operating systems, security policies, tools, or organizational practices change.

## 7.6. Workflow Extraction Engine

Workflow extraction should be implemented as a required governed engine rather than as an informal summarization step or optional future benefit. The engine's purpose is to transform Episode Objects and source evidence into inspectable workflow traces, candidate generalized workflows, and supporting material for procedural-memory formation. This is an architectural proposal informed by process-mining methods, and it should remain evidence-linked and reviewable rather than presented as automatic truth (van der Aalst et al., 2004; IEEE Task Force on Process Mining, 2012).

A practical workflow extraction engine should begin by selecting the project, subject, action class, time window, source types, and security domain to analyze. It should then group events into candidate cases, such as one server installation, incident response, deployment, or approval workflow, and extract the relevant steps from tickets, logs, transcripts, commands, documents, and Episode Objects with source links preserved. The engine should normalize different wording into common action classes, objects, actors, tools, and outcomes; order the steps using event time, source time, and transaction time while preserving uncertainty; and construct per-case workflow traces with actors, steps, inputs, outputs, evidence, confidence, and exceptions. Similar traces can then be clustered by subject type, action class, outcome, environment, tool stack, and exception pattern so that recurring sequences and branches can be proposed as candidate generalized workflows. Before promotion, the engine should perform causal analysis, conformance review against existing workflows, and human, policy, or model-assisted validation so that candidate workflows do not influence procedural memory or skills until they have appropriate provenance and review status.

![Workflow Extraction Engine Process](Malu-Figure-3.psd)

*Figure 3: Workflow Extraction Engine Process.*

The engine should preserve both positive and negative evidence. Successful traces may support a generalized workflow, while failed or partial traces may identify missing prerequisites, unsafe assumptions, or known exception paths. The output of the workflow extraction engine should therefore include not only a proposed step sequence, but also supporting traces, contradictory traces, confidence and precision scores, applicability conditions, known variants, and links back to the source claims, facts, and Episode Objects.

The workflow extraction engine should also separate temporal sequence from causality. A trace may show that a configuration change happened before an outage, but a causal finding should require additional support such as a mechanistic explanation, repeated pattern, diagnostic evidence, human verification, or counterfactual analysis. For example, "the server failed after the network change" is a sequential or correlational relationship, while "the network change caused the server failure because it removed the failover route" is a causal claim that should be represented, reviewed, and queried differently. Causal knowledge graph research shows that causal edges and counterfactual reasoning require richer structure than ordinary graph adjacency, so this paper treats causal reasoning as an architectural hook and future capability rather than as an automatic inference guarantee (Heindorf et al., 2020; Jaimini & Sheth, 2022; Zellinger et al., 2024).

The workflow extraction engine should not directly overwrite an existing workflow or procedural memory. Instead, it should create candidate workflow versions with provenance and review status. A candidate may be accepted, rejected, merged, retained as a variant, marked environment-specific, or used only as an exception record. This keeps process mining aligned with the DBMS governance model.

## 7.7. Skill Derivation Schema and Runtime

Skills and competencies should have a concrete schema and runtime behavior. A skill is a governed package of procedural memory, workflows, evidence, and execution policy. It should be usable by humans and AI agents, but it should remain traceable to the memories, workflow traces, generalized workflows, and procedural memories from which it was derived.

A skill object should have a stable identifier and version, a human-readable name and purpose, links to the procedural memories and workflows that justify it, and explicit applicability conditions such as project, subject, environment, technology stack, time window, and governing policy. It should define preconditions, required and optional inputs, expected outputs, created artifacts, ordered steps, decision points, validation checks, known exceptions, recovery paths, escalation conditions, and unsafe states. It should also carry security policy, model policy, confidence and precision scores, bitemporal validity, evidence and provenance links, and staleness rules that force review when tools, policies, failed executions, or model dependencies change.

At runtime, a skill should execute as a governed state machine rather than as a free-form prompt, but the concrete runtime format may vary based on the LLM, agent framework, and orchestration harness used to manage skill utilization. Some skills may be materialized as system prompts or prompt templates, others as Markdown files that describe operational procedures, and others as application plugins, MCP-accessible tools, or workflow definitions that can be invoked by a runtime. Regardless of packaging, the runtime should bind the current user or agent account, active memory pool, task objective, authorized partitions, and relevant source context. It should check preconditions, load only authorized memories and workflows, present or execute steps, record observations, evaluate validation checks, branch on known exceptions, and emit new claims, workflow traces, or procedural-memory updates when the execution produces useful new knowledge.

This runtime distinction is important for safety. A human may use the skill as guided operational documentation. An AI agent may use it as an executable procedure only if its account, tools, model policies, and target partitions permit that use. In either case, every skill execution should produce an auditable execution record that can later support workflow conformance, procedural-memory refinement, staleness review, or incident investigation.

# 8. Hybrid Structure: Hierarchy Plus Graph

The proposed model should not be implemented as a simple tree. Institutional memory is too interconnected for a strict hierarchy, but it is also too large and operationally sensitive for every query to begin as an unconstrained graph traversal or global vector search (Hogan et al., 2021). The DBMS should therefore use a **hybrid structure** that combines several searchable representations, each optimized for a different part of memory recall. The hierarchy narrows the search space, the graph follows meaningful relationships, recursive detail expands only the level of operational detail required, temporal indexes constrain time and validity, vector indexes recover semantic similarity, and relational/catalog indexes enforce governance and exact filtering. The DBMS may use any combination of these structures to satisfy a query, and users, applications, or AI agents should be able to guide the search process when they have additional insight into the use case, desired answer type, or most appropriate initial search path.

**Semantic hierarchy for organization:**

This path routes from the main entity being remembered, to the action or state, to the structured meaning, and finally to the memory payload or target object.

**Subject → Verb → Predicate → Object**

Search role: route queries into meaningful compartments before expensive retrieval begins. A query can first identify the subject, action class, predicate frame, or object type, then search within that narrowed compartment instead of scanning the entire memory store.\
\
**Graph relationships for traversal:**

This mechanism uses typed graph edges to connect a memory to related memories, facts, claims, source evidence, workflows, and entities beyond its primary hierarchy location.

**Memory ↔ Relationship ↔ Memory / Fact / Claim / Source / Workflow / Entity**

Search role: follow typed edges such as `supports`, `contradicts`, `supersedes`, `derived_from`, `depends_on`, `caused_by`, `part_of`, and `related_to`. Graph search is used when the user or agent needs connected evidence, causal chains, workflow dependencies, source lineage, or cross-project relationships.\
\
**Recursive detail for expansion:**

This structure opens a memory into progressively deeper operational detail, from a parent memory to detail objects, subdetails, evidence, steps, and exceptions.

**Memory -> Detail -> Subdetail -> Evidence / Step / Exception**

Search role: expand a parent memory only to the level of detail required by the task. A high-level recall query may stop at the parent memory, while a skill execution or troubleshooting query may expand into steps, substeps, commands, parameters, validation checks, evidence, and exceptions.\
\
**Temporal indexes for time:**

This index family searches by when something happened, when it was asserted, when it was verified, when it was valid, and when it may become stale.

**event\_time, assertion\_time, verification\_time, validity\_period, stale\_after**

Search role: restrict retrieval by when something happened, when it was recorded, when it was verified, when it was valid, and whether it may now be stale. Temporal search allows the DBMS to answer historical, current-valid, transaction-time, and bitemporal questions without confusing past truth with present guidance.\
\
**Vector indexes for semantic search:**

This mechanism organizes embedding spaces by meaningful scopes so semantic search can run within authorized compartments instead of across the entire database.

**embeddings scoped by subject, verb, predicate, object, project, or time**

Search role: recover semantically similar memories, details, workflows, source excerpts, and skills when exact terms are unknown or incomplete. Vector search should normally operate inside authorized, structured scopes rather than across the full enterprise corpus.\
\
**Relational/catalog indexes for governance:**

This control layer uses exact metadata fields to identify objects, enforce policy, filter results, and keep retrieval auditable.

**entity IDs, source IDs, claim IDs, fact IDs, workflow IDs, confidence, precision, status**

Search role: apply exact filters, identity resolution, authorization, status checks, confidence thresholds, precision thresholds, partition membership, model version constraints, and object-type restrictions. Catalog search gives the DBMS a reliable control layer around semantic and graph retrieval.

The hierarchy provides compartments. Recursive detail provides controllable depth. The graph provides connections and reuse across memories, workflows, and skills. The temporal model provides sequence and validity. The vector index provides semantic recall. The relational catalog provides consistency, governance, and query discipline (Kulkarni & Michels, 2012; Hogan et al., 2021).

![Hybrid Memory Structure Diagram](Malu-Figure-4.pdf)

*Figure 4 - Hybrid Memory Structure Diagram.*

In practice, the DBMS should combine these mechanisms rather than choose only one. For example, a query about why a project decision was made may begin with the semantic hierarchy to identify `Project Y` and the action class `decided`, use temporal indexes to restrict the search to 2019, apply catalog indexes to enforce authorization and confidence thresholds, follow graph relationships such as `because_of` or `supports`, and then use vector search only inside the remaining authorized compartment to find semantically similar memories or source excerpts. The result is not simply "vector search with metadata." It is a coordinated search path that uses each structure where it is strongest.

The same design supports progressive detail retrieval. A user or AI agent may begin with a parent memory or generalized workflow, then expand only the relevant Memory Detail Objects. For example, the system may retrieve the parent memory "installed Ubuntu 24.04 on Proxmox," expand the step "created new virtual machine," then follow graph edges to related memories about network bridge selection, failed boot-order settings, or successful validation checks. This combines document-like nested detail with graph-like reuse and avoids treating every operational instruction as either a flat document paragraph or a globally searched vector chunk.

# 9. Compartmentalized Search

A major purpose of the organizational model is to reduce the cost and ambiguity of search. In a large enterprise database containing memories, search should occur in stages.

A query such as "Why was Server X installed?" should be routed as a scoped recall path rather than as an undifferentiated semantic search. The retrieval process begins by identifying the subject, expands the likely verb space, selects predicate targets that indicate what kind of answer is being requested, searches only the relevant memory compartments, traverses relationships, retrieves supporting evidence, and returns an answer with confidence, precision, and provenance.

![Compartmentalized Recall for an Installation-Reason Query](Malu-Figure-5.pdf)

*Figure 5 - Compartmentalized Recall for an Installation-Reason Query.*

A query such as "What is our process for installing a production server?" follows a different path because the user is asking for reusable operational knowledge rather than a reason for a specific past event. The DBMS should identify the object type and verb class, retrieve similar Episode Objects, extract workflow traces, compare common and exceptional paths, and return a generalized workflow with supporting memories and confidence.

![Compartmentalized Workflow Retrieval for a Production-Server Installation Query](Malu-Figure-6.pdf)

*Figure 6 - Compartmentalized Workflow Retrieval for a Production-Server Installation Query.*

This routing model allows the DBMS to select the appropriate retrieval strategy rather than treating every query as a generic semantic search (Barnett et al., 2024; Khattab & Zaharia, 2020; Thakur et al., 2021).

## 9.1. Search Path Selection and Query Hints

A generic user query should not force the DBMS to begin with a single default retrieval method. The same natural-language request may require a different search path depending on whether the user wants a historical memory, a current-valid fact, a supporting source, a workflow, a procedural skill, a causal explanation, a recursive detail expansion, or an exploratory semantic search. The DBMS should therefore treat the initial query as a request to build a retrieval plan rather than as text to send directly to a vector index.

The search path selection process should begin by creating a retrieval envelope. The envelope includes the authenticated account, roles, active memory pool, application context, agent delegation chain, allowed partitions, current task, query text, and any structured fields supplied by the caller. The DBMS should then extract candidate retrieval cues from the query, including subject, verb or action class, object type, topic, time range, relationship hints, desired answer shape, evidence requirement, confidence threshold, precision requirement, and recursive detail depth. These cues should be treated as candidates rather than as final truth, because generic language may be incomplete or ambiguous.

After cue extraction, the DBMS should classify the likely intent of the query and choose an initial path. A specific recall question should usually begin with subject, verb, temporal, and catalog filters before graph or vector expansion. A "why" or "how do we know" question should begin with the derivation ledger, evidence links, source references, and supporting or contradicting relationships. A "how do we perform this task" question should begin with workflow traces, generalized workflows, procedural memories, skills, and recursive detail objects. A date-sensitive question should begin with temporal indexes and bitemporal validity. A dependency, cause, or relationship question should begin with graph traversal. A broad exploratory question may begin with a scoped semantic search, but even then the DBMS should apply authorization, partition, subject, topic, and time filters before expanding into larger vector search.

The selected path should be a plan, not a rigid single step. The retrieval coordinator may start with the cheapest or most precise indexes, gather candidate memories and facts, then expand into graph traversal, recursive detail, vector similarity, source inspection, or workflow lookup when the first path is insufficient. The final result should be assembled only after access control, confidence, precision, validity, provenance, and evidence requirements have been checked. This makes generic search more reliable because the system can adaptively choose the right search structure while still preserving deterministic governance boundaries.

The DBMS should also support an optional **query hint** structure. Hints allow a user, application, AI agent, or MCP server to suggest the initial search path, expected result shape, or depth of detail when the use case is known. Hints are useful because a human or application may know that the query is asking for a workflow, a source-backed answer, a current-valid fact, or a detailed procedural expansion even when the natural-language prompt is short. A hint should guide planning, but it should not override security, access control, validity rules, provenance requirements, or DBMS safeguards. The system should be free to ignore or downgrade a hint when it conflicts with policy, missing evidence, or the query's inferred intent.

For example, a procedural query may include a hint package like this:

```json
{
  "query": "What should I know before installing this server?",
  "hints": {
    "intent": "procedural_guidance",
    "preferred_paths": ["workflow", "skill", "recursive_detail"],
    "subject": "Server X",
    "object_type": "production server",
    "time_mode": "current_valid",
    "detail_depth": 2,
    "evidence": "summaries_with_source_links",
    "minimum_confidence": 0.75,
    "partitions": ["Project Y", "Infrastructure"]
  }
}
```

In this example, the hint tells the DBMS to begin with workflow and skill retrieval rather than open-ended semantic recall. The `detail_depth` value asks the DBMS to expand the procedure to major steps and substeps, but not necessarily to every command, log excerpt, and source artifact unless the agent later asks for deeper evidence. This pattern supports both human usability and machine precision: casual users can ask ordinary questions, while applications and AI agents can provide structured hints when they know the operational context.

## 9.2. Pre-Loaded or Cached Search

Compartmentalized search should also support **pre-loaded search contexts** for high-speed work. Before a human or AI-agent task begins, the DBMS should be able to build a subject-scoped memory buffer pool that brings relevant memories, facts, workflows, procedural memories, skills, source references, and recent active context close to the agents that will use them. This can be initiated by a user prompt, application request, API call, or MCP server command, such as "load authentication memories and skills for Project Y" or "prepare the production-server installation context for this deployment task."

The formal conceptual object for this pattern is an **Active Memory Pool**, while a cache or buffer pool is the performance mechanism used to make selected content faster to retrieve. The preload operation should use the same governed retrieval model as ordinary search: it should identify the subject, allowed topics, relevant verbs, workflow families, source classes, confidence thresholds, validity windows, and security boundaries before materializing any shared working set. The resulting cached pool should not be an opaque shortcut around the DBMS. It should preserve provenance, source pointers, confidence, precision, staleness status, and access-control labels so agents can inspect why a memory was loaded and whether it is safe to use.

Subject-based loading is especially useful because it reduces repeated broad searches during active work. If several agents are collaborating on authentication for a PHP system, the system can preload authentication-related memories, login workflows, OAuth or OIDC decisions, token-handling edge cases, prior incident records, approved skills, and relevant raw-source pointers into the active pool. Agents connected to that pool can then retrieve high-value context with low latency instead of issuing repeated semantic searches against the full enterprise memory space.

Real-time shared memories are an extension of this pattern. Multiple agents and humans may read from and write to the same active pool simultaneously while they perform a task, record actions, raise questions, add observations, and propose claims. The pool becomes a shared operational memory surface rather than a static search result. Because these active memories may later influence decisions or be promoted into durable institutional memory, every write should remain attributable to a user account, AI-agent account, MCP server, tool, or application and should be governed by the same permissions, retention rules, and promotion policies used elsewhere in the DBMS.

![Pre-Loaded Subject Memory Pool for Real-Time Agent Collaboration](Malu-Figure-7.pdf)

*Figure 7: Pre-Loaded Subject Memory Pool for Real-Time Agent Collaboration.*

# 10. Implementation Requirements

This section intentionally limits the implementation discussion to requirements the proposed DBMS must satisfy. The purpose is not to prescribe a complete storage-engine design or product implementation. Instead, it defines the architectural commitments needed for the system to operate as a database of memories rather than as an application assembled from unrelated search, graph, document, vector, and time-series tools. The detailed engineering choices, benchmark targets, and internal algorithms should be handled in a separate implementation specification.

A memory DBMS should be evaluated against four requirements: it must preserve the evidence and provenance behind memories, support retrieval across structured and semantic representations, enforce security and temporal validity during recall, and allow derived artifacts to be rebuilt or improved as models and indexes evolve. These requirements distinguish the proposed system from a conventional vector store, document database, knowledge graph, or workflow tool (Moreau & Missier, 2013; Kulkarni & Michels, 2012; Hogan et al., 2021; PostgreSQL Global Development Group, n.d.).

The implementation should be organized around three architectural domains. The **Enterprise Memory Core** is the durable system of record for source archives, claims, facts, Episode Objects, memories, workflows, procedural memories, skills, relationships, partitions, indexes, catalog metadata, audit records, users, roles, privileges, and model lifecycle metadata. **Local Memory Nodes** allow humans, applications, and AI agents to stage or operate with local subsets of memory near the point of work, then synchronize selected claims, memories, or source packages back to the enterprise system. **Active Memory Pools** provide scoped working-memory spaces where humans and AI agents can share loaded memories, skills, pending observations, and task state in real time.

![Implementation Requirements Overview](Malu-Figure-8.pdf)

*Figure 8: Implementation Requirements Overview.*

## 10.1. Core System Requirements

The Enterprise Memory Core must provide a governed object model for source packages, claims, facts, Episode Objects, memories, Memory Detail Objects, workflows, procedural memories, skills, relationships, and derived artifacts. These objects should not be treated as loose application records. They should be first-class database objects with identifiers, provenance, temporal fields, confidence and precision metadata, lifecycle status, access controls, and links back to supporting evidence. The core should support recursive detail so memories, workflow traces, procedural memories, and skills can expand from concise summaries into source-linked steps, substeps, parameters, commands, validations, exceptions, and evidence when the task requires it. When legal and retention policies permit, the core must also preserve a **Verbatim Source Archive** of timestamped raw inputs because future extraction models, embedding models, summarizers, and workflow-mining methods may make it valuable to re-ingest the original material rather than relying only on derived records.

The system catalog must serve as the central metadata repository for the DBMS. It should describe users, roles, privileges, schemas, partitions, object types, relationship types, retention policies, verification policies, source types, model versions, index definitions, active memory pools, local nodes, archive tiers, rebuild state, and operational statistics. Catalog information should be readable through SQL and governed APIs so administrators, applications, authorized users, and agents can inspect the structure, health, usage, and performance of the system. At the system level, the catalog should also expose statistics about predefined slices such as subjects, verbs, topics, projects, security domains, active pools, and time partitions.

The core must provide bitemporal behavior. Every governed memory object should distinguish when an event happened or was valid from when the database learned, recorded, modified, or superseded it. New memories may invalidate or supersede prior facts without erasing the older historical record. This requirement is central to institutional memory because a statement may be true for a past period, false for the present, and still important for understanding how a decision or workflow evolved.

The core must include a derivation ledger. The ledger should trace how sources become claims, how claims become facts, how facts and claims become memories, and how repeated memories become workflows, procedural memories, skills, or competencies. This ledger is the basis for auditability, review, model reprocessing, evidence inspection, and downstream invalidation when a source, model, fact, or workflow changes.

The implementation must define transaction semantics for multi-model writes. A single operation may update object metadata, source links, graph relationships, temporal windows, full-text indexes, vector indexes, workflow records, and audit logs. The proposed architecture does not require a particular storage engine, but the DBMS must provide an internal transaction boundary that prevents partially committed memory operations. A unified logical write-ahead log, global commit sequence, MVCC strategy, or equivalent mechanism should make crash recovery able to restore all internal stores to a consistent state. Critical promotion and supersession operations should have stronger isolation guarantees than low-risk maintenance tasks.

## 10.2. Retrieval, Embedding, and Model Requirements

The enterprise memory DBMS must include native support for semantic search and vector embedding, but embedding and search must be interchangeable components that can evolve as AI technologies improve. Structure-aware memory organization should improve retrieval reliability regardless of which vector engine or embedding model is used. When practical, the system should incorporate the subject, verb, predicate, object, relationship, topic, temporal scope, and recursive detail path into both embedded chunks and query-time retrieval packages. This prompt-plus approach allows the DBMS to avoid treating natural-language prompts as opaque strings and reduces the need for low-precision global search (Lewis et al., 2020; Thakur et al., 2021; Khattab & Zaharia, 2020).

Semantic retrieval must be security-aware. Authorization should be evaluated before candidate expansion, during index selection, and again when results are assembled. The system should avoid exposing unauthorized information through vector similarity, graph traversal, summaries, relationship expansion, or active memory pool loading. This means access policy cannot be bolted onto the final answer only after retrieval. It must be part of retrieval planning, candidate filtering, reranking, evidence expansion, and result packaging.

The retrieval engine should support hybrid recall across structured filters, full-text search, graph traversal, temporal constraints, workflow lookup, source inspection, recursive detail expansion, and vector search. A query such as "What should I know before installing this server?" may need current environment facts, prior server installation Episode Objects, known edge cases, generalized workflows, procedural memories, relevant skills, source evidence, and active pool state. A memory DBMS should therefore return retrieval packages rather than only ranked text chunks. Those packages may include memories, facts, source references, workflow steps, recursive detail depth, confidence scores, validity status, permissions, and links to evidence. The retrieval engine should also support search-path planning and optional query hints so users, applications, AI agents, and MCP servers can suggest whether the initial path should emphasize facts, evidence, workflows, skills, graph traversal, temporal validity, recursive detail, or semantic exploration.

Model services should be treated as replaceable and governed components. Embedding models, rerankers, extraction models, summarizers, workflow-mining models, validation models, and small maintenance language models may run locally on CPU, locally with GPU acceleration, or through approved cloud services. Local execution may improve data locality, latency, and governance control, while cloud execution may offer elasticity, managed operations, and stronger models. The architecture should support both, with model choice controlled by policy, workload, data sensitivity, cost, compliance, and infrastructure availability (Shi et al., 2016; Satyanarayanan, 2017; Kwon et al., 2023; Johnson et al., 2019).

The model registry must track model identity, version, dimensions, embedding space, prompt or template policy, evaluation status, rollout state, and derived artifacts. When embedding models change, the system must support migration strategies that avoid forcing long downtime. Acceptable strategies include blue-green indexes, dual-space query routing during transition, adapter-based alignment between embedding spaces, and staged background re-embedding. The registry should record which memories, indexes, summaries, workflows, and skills were produced by which models so that upgrades can be audited, rolled back, compared, or used to trigger targeted reprocessing (Sculley et al., 2015; Mitchell et al., 2019; Gebru et al., 2021).

The system should also manage embedding write amplification. Multi-view embedding can improve precision, but embedding every memory through every possible scope at ingestion time may create unnecessary storage, cost, and rebuild burden. The DBMS should support embedding budget policies, lazy or deferred embedding, compact routing vectors, partition-specific embedding scopes, and rebuild prioritization. High-value active partitions may receive richer embeddings, while cold historical partitions may rely on fewer embeddings until queried, restored, or selected for reprocessing.

## 10.3. Security, Governance, and Enterprise Operation Requirements

Every actor that connects to the DBMS should have an account. This includes human users, applications, service accounts, AI agents, MCP servers, local memory nodes, and administrative processes. The system should support direct user privileges, role-based privileges, API keys linked to individual accounts, service-account controls, and administrative role creation. Built-in role patterns such as CONNECT, RESOURCE, and DBA may provide a familiar baseline, but the system should also support custom roles aligned to projects, subjects, topics, workflows, source domains, and governance responsibilities.

Privileges must apply to memory-specific objects and semantic slices, not only to tables or collections. A user may be allowed to read a topic such as authentication in PHP systems without receiving full access to every project where authentication work occurred. The authorization model should therefore support grants on subjects, verbs, topics, projects, partitions, source types, workflow objects, active memory pools, and relationships among those objects. Grammar-based organization is not only a retrieval optimization; it also becomes a way to express access control over institutional knowledge at a level more meaningful than a table, file, or vector namespace.

Logging, auditing, and observability are required enterprise features. The DBMS should record who accessed, changed, verified, promoted, synchronized, exported, deleted, or derived governed objects. It should also record ingestion jobs, model execution, prompt or template policy, model version, index rebuilds, archive recalls, synchronization events, recovery operations, quality signals, performance metrics, and traces. These records support security review, operational monitoring, regulatory obligations, model lifecycle governance, and debugging when retrieved memories or derived skills are questioned (NIST, 2006; NIST, 2020).

Backup and recovery should prioritize durable assets that cannot be cheaply regenerated. Verbatim source packages, catalog metadata, access-control state, human verification decisions, derivation ledgers, transaction logs, memory objects, facts, claims, and governance records are core recovery assets. Embeddings, vector indexes, summaries, routing structures, and other derived artifacts should be recoverable through rebuild procedures when source data and metadata remain intact. For large deployments, this strategy can reduce total cost of ownership by avoiding unnecessary full backup of every rebuildable semantic artifact, while still requiring tested rebuild ordering, restore validation, and audit records showing which models, source snapshots, templates, and index definitions were used during recovery.

The DBMS must expose stable client access surfaces. At minimum, it should support common enterprise programming environments such as Python, Node.js, PHP, and other languages through drivers, APIs, or protocol bindings. It should also support a query surface that combines structured requests, SQL-like inspection where appropriate, and prompt-plus retrieval commands for memory-oriented recall. Applications should be able to retrieve memories, sources, workflows, skills, and active-pool context without depending on a single proprietary client workflow.

## 10.4. Partitioning, Local Nodes, and Active Memory Requirements

Partitioning is required for performance, governance, lifecycle management, and future distribution. The system should support partitions by subject, project, security domain, time range, workflow, source domain, retention policy, and active pool. Cross-partition queries must be explicit about authorization and result assembly, because memories often reference facts, sources, or workflows outside their primary project. As the system grows, partitioning also becomes the first step toward a distributed memory database that can operate across equipment in a network, even though high availability and active-active replication remain outside the initial scope.

Local Memory Nodes should support offline, private, edge, or personal work without becoming independent ungoverned databases. A local node may hold selected memories, pending claims, source snippets, task context, local observations, and synchronization metadata. When reconnected, it should submit new claims, Episode Objects, source packages, conflict records, deletions or tombstones, workflow updates, and promotion candidates to the Enterprise Memory Core. The enterprise system remains authoritative, but local nodes make memory capture possible closer to where work occurs.

Active Memory Pools should support live task work. A pool may be created for a project, incident, deployment, investigation, design session, coding task, support case, or multi-agent collaboration. The pool can be loaded from a prompt, API command, MCP server request, or structured query that identifies a subject, topic, workflow, time range, or task. It should contain relevant memories, facts, workflows, skills, source references, pending claims, task observations, and collaboration state. The purpose is to bring the right governed context close to the agents and humans doing the work, reducing repeated retrieval overhead and enabling shared situational awareness.

Agent-to-agent shared memory channels should extend active pools when multiple AI agents or humans need simultaneous access to the same working context. These channels should support scoped membership, publish-and-subscribe updates, pending observations, retrieved memory references, conflict markers, promotion candidates, and audit logs. Access may be implemented through APIs, WebSockets, direct TCP/IP, or other transports, with WebSockets or direct TCP/IP likely offering the lowest latency for real-time collaboration. Regardless of transport, the channel must remain governed by the same identity, privilege, partition, provenance, and promotion rules as the rest of the DBMS.

Active memories should not automatically become permanent memories. The system should support a promotion path from active observations to pending claims, verified facts, Episode Objects, workflow traces, generalized workflow updates, procedural memory updates, and skill refinements. It should also support consolidation and lifecycle management so old episodic traces do not remain equally prominent forever. Salience, reinforcement, decay, archive movement, legal hold, and retention policy should determine whether memories remain in active retrieval paths, become consolidated into higher-level procedural knowledge, move to cold storage, or are eventually pruned when policy permits.

## 10.5. Deferred Implementation Decisions

Several choices are important but should not be over-specified in this white paper. The paper does not require a particular storage engine, vector index implementation, graph engine, programming language runtime, model provider, or exact optimizer design. It also does not require high availability, active-active replication, or a distributed consensus layer in the initial version. Those capabilities may become important later, but treating them as first-version requirements could distract from the harder foundation: defining durable memory objects, provenance, temporal truth, security-aware retrieval, model lifecycle governance, and internal consistency.

The C-core design should decide the exact transaction model, storage layout, WAL format, MVCC strategy, concurrency rules, index visibility policy, recovery sequence, and model-execution boundary before implementation begins. This white paper commits to the requirements those mechanisms must satisfy: governed memory objects must not be partially written, derived artifacts must be traceable and rebuildable, retrieval must respect security policy, model outputs must be versioned and auditable, and the system must remain capable of improving its semantic layer as AI models and indexing methods advance.

The result is a deliberately constrained implementation architecture. The proposed DBMS is not merely a vector database with metadata, a graph database with embeddings, or a document store with an LLM attached. It is a database system whose required implementation properties are organized around memories: where they came from, when they were valid, who may access them, how they can be retrieved, how they become workflows and skills, and how the system can preserve them long enough to benefit from future advances in AI.

# Appendix A. Proposed Terminology

This appendix provides a consistent vocabulary for the paper.

  **Grammar-Inspired Term**   **Formal DBMS Term**      **Recommended Use**
  --------------------------- ------------------------- ------------------------------------------------------------------------------
  Subject                     Entity Anchor             Primary organizing entity
  Verb                        Action Class              What happened or what is being remembered
  Predicate                   Semantic Frame            Structured meaning of the memory
  Object                      Target / Memory Payload   Thing acted on or remembered
  Preposition                 Relationship Edge         Connection between memories, facts, claims, sources, workflows, and entities

The model may still be described as grammar-inspired, but the formal architecture should use **Relationship** rather than **Preposition**. Prepositions provide useful human-readable labels for many relationships, but institutional memory requires a broader graph structure that can represent temporal, causal, evidentiary, procedural, hierarchical, and semantic connections. Therefore, the paper uses the term Relationship to describe typed edges between memories, facts, claims, sources, workflows, entities, and timelines.

The proposed memory organization model is:

Subject → Verb → Predicate → Object ↔ Relationship

The paper also maps common memory-theory terms to implementation-oriented DBMS terms:

  **Conceptual Memory Term**   **DBMS Representation**                       **Recommended Use**
  ---------------------------- --------------------------------------------- ------------------------------------------------------------------------------
  Episodic Memory              Episode Object                                 First-class DBMS object representing a specific remembered episode; "event memory" may be used only as an informal synonym
  Semantic Memory              Facts, relationships, and semantic indexes     Stable knowledge about entities, concepts, meanings, and relationships
  Procedural Memory            Procedural Memory Object                       Retained how-to knowledge for performing, adapting, validating, and repairing repeatable work
  Working Memory               Active Memory Pool                             Temporary shared task context for humans and AI agents

In this terminology, **episodic memory** is first-class at the conceptual level, while **Episode Object** is the concrete DBMS object type used to store, query, govern, replay, and trace a specific remembered episode. **Event memory** may remain a readable synonym, but the formal schema name should be Episode Object. **Procedural memory** is also first-class conceptually, but it should not be collapsed into a workflow. A workflow is an explicit process representation; procedural memory is the broader retained know-how that may use workflows, examples, exceptions, validations, and skills.

The following canonical implementation terms should be used consistently:

  **Canonical Term**              **Meaning**
  ------------------------------- ------------------------------------------------------------------------------------------------------
  **Episode Object**              Concrete DBMS object for a specific remembered episode, including evidence, time, actors, relationships, and replay metadata
  **Memory Detail Object**        Addressable child or linked object that represents a step, substep, parameter, command, validation, exception, source excerpt, or evidence item within a memory or skill
  **Recursive Detail**            The ability to expand Memory Detail Objects into additional governed details only to the depth required by a task, query, review, or skill execution
  **Workflow Trace**              Observed step sequence from one Episode Object or case
  **Generalized Workflow**        Repeatable process pattern derived from multiple traces or validated examples
  **Procedural Memory Object**    Capability-oriented how-to knowledge for performing, adapting, validating, and repairing work
  **Skill Package**               Governed procedural memory packaged for reuse by humans or AI agents with execution policy and audit records
  **Active Memory Pool**          Scoped working-memory space for a task, project, incident, agent group, or collaboration session
  **Derivation Ledger**           Auditable lineage from source package through claims, facts, memories, workflows, procedural memories, skills, and derived artifacts
  **Temporal Supersession Engine** DBMS subsystem that manages valid-time windows, transaction-time history, corrections, supersession, staleness, and current validity
  **Security-Aware Retrieval Coordinator** Subsystem that builds authorization context, prefilters retrieval, post-validates candidates, and assembles policy-aware result packages
  **Transaction and WAL Manager** Subsystem that provides MVCC, global commit sequencing, isolation policy, crash recovery, and internal multi-model atomicity

# Appendix B. References

Adwant, G. (2026). *vectormigrate: Python-first tooling for safe embedding-model migration across vector retrieval systems.* PyPI. [Link](https://pypi.org/project/vectormigrate/)

Alqithami, S. (2025). "Forgetful but Faithful: A Cognitive Memory Architecture and Benchmark for Privacy-Aware Generative Agents." arXiv:2512.12856. [Link](https://arxiv.org/abs/2512.12856)

Apache Avro. (n.d.). *Apache Avro Documentation.* [Link](https://avro.apache.org/docs/)

Apache NiFi. (n.d.). *Apache NiFi User Guide.* [Link](https://nifi.apache.org/docs/nifi-docs/html/user-guide.html)

Argote, L., & Ingram, P. (2000). "Knowledge Transfer: A Basis for Competitive Advantage in Firms." *Organizational Behavior and Human Decision Processes*, 82(1), 150-169. [Link](https://www.sciencedirect.com/science/article/pii/S0749597800928930)

Asai, A., Wu, Z., Wang, Y., Sil, A., & Hajishirzi, H. (2024). "Self-RAG: Learning to Retrieve, Generate, and Critique through Self-Reflection." *ICLR 2024*. arXiv:2310.11511. [Link](https://arxiv.org/abs/2310.11511)

Baddeley, A. (1992). "Working Memory: The Interface between Memory and Cognition." *Journal of Cognitive Neuroscience*, 4(3), 281-288. [Link](https://pubmed.ncbi.nlm.nih.gov/23964884/)

Barnett, S., Kurniawan, S., Thudumu, S., Brannelly, Z., & Abdelrazek, M. (2024). "Seven Failure Points When Engineering a Retrieval Augmented Generation System." *CAIN 2024*. arXiv:2401.05856. [Link](https://arxiv.org/abs/2401.05856)

Buneman, P., Khanna, S., & Tan, W.-C. (2001). "Why and Where: A Characterization of Data Provenance." *ICDT 2001*. [Link](https://homepages.inf.ed.ac.uk/opb/papers/ICDT2001.pdf)

Cheney, J., Chiticariu, L., & Tan, W.-C. (2009). "Provenance in Databases: Why, How, and Where." *Foundations and Trends in Databases*, 1(4), 379-474. [Link](https://research.ibm.com/publications/provenance-in-databases-why-how-and-where)

Cohen, N. J., & Squire, L. R. (1980). "Preserved learning and retention of pattern-analyzing skill in amnesia: Dissociation of knowing how and knowing that." *Science*, 210(4466), 207-210. [Link](https://pubmed.ncbi.nlm.nih.gov/7414331/)

Consultative Committee for Space Data Systems. (2012). *Reference Model for an Open Archival Information System (OAIS).* ISO 14721 / CCSDS 650.0-M-2. [Link](https://www.iso.org/standard/57284.html)

Galan, N. (2023). "Knowledge loss induced by organizational member turnover: a review of empirical literature, synthesis and future research directions (Part I)." *The Learning Organization*, 30(2), 117-136. [Link](https://doi.org/10.1108/TLO-09-2022-0107)

Gebru, T., Morgenstern, J., Vecchione, B., Vaughan, J. W., Wallach, H., Daume III, H., & Crawford, K. (2021). "Datasheets for Datasets." *Communications of the ACM*, 64(12), 86-92. [Link](https://dl.acm.org/doi/10.1145/3458723)

Gray, J., & Reuter, A. (1992). *Transaction Processing: Concepts and Techniques.* Morgan Kaufmann. [Link](https://dl.acm.org/doi/10.5555/573304)

GustyCube. (n.d.). *membrane/pkg/decay package documentation.* Go Packages. [Link](https://pkg.go.dev/github.com/GustyCube/membrane/pkg/decay)

Heindorf, S., Scholten, Y., Wachsmuth, H., Ngonga Ngomo, A.-C., & Potthast, M. (2020). "CauseNet: Towards a Causality Graph Extracted from the Web." *CIKM 2020*. [Link](https://causenet.org/)

Hogan, A., Blomqvist, E., Cochez, M., D'Amato, C., de Melo, G., Gutierrez, C., et al. (2021). "Knowledge Graphs." *ACM Computing Surveys*, 54(4), Article 71. [Link](https://dl.acm.org/doi/10.1145/3447772)

IEEE Task Force on Process Mining. (2012). "Process Mining Manifesto." *Business Process Management Workshops*, LNBIP 99, 169-194. [Link](https://www.tf-pm.org/upload/1580737614108.pdf)

Jaimini, U., & Sheth, A. (2022). "CausalKG: Causal Knowledge Graph Explainability Using Interventional and Counterfactual Reasoning." *IEEE Internet Computing*, 26(1), 43-50. [Link](https://doi.org/10.1109/MIC.2021.3133551)

Johnson, J., Douze, M., & Jegou, H. (2021). "Billion-scale similarity search with GPUs." *IEEE Transactions on Big Data*. [Link](https://arxiv.org/abs/1702.08734)

Kent, K., & Souppaya, M. (2006). *Guide to Computer Security Log Management.* NIST SP 800-92. [Link](https://csrc.nist.gov/pubs/sp/800/92/final)

Keeney, R. L., & Raiffa, H. (1976). *Decisions with Multiple Objectives: Preferences and Value Tradeoffs.* Wiley.

Khattab, O., & Zaharia, M. (2020). "ColBERT: Efficient and Effective Passage Search via Contextualized Late Interaction over BERT." *SIGIR 2020*, 39-48. [Link](https://arxiv.org/abs/2004.12832)

Ko, A. J., DeLine, R., & Venolia, G. (2007). "Information Needs in Collocated Software Development Teams." *ICSE 2007*, 344-353. [Link](https://doi.org/10.1109/ICSE.2007.45)

Kulkarni, K., & Michels, J.-E. (2012). "Temporal Features in SQL:2011." *SIGMOD Record*. [Link](https://dl.acm.org/doi/10.1145/2380776.2380786)

Lewis, P., Perez, E., Piktus, A., Petroni, F., Karpukhin, V., Goyal, N., et al. (2020). "Retrieval-Augmented Generation for Knowledge-Intensive NLP Tasks." *NeurIPS 2020*. [Link](https://proceedings.neurips.cc/paper/2020/hash/6b493230205f780e1bc26945df7481e5-Abstract.html)

Liu, N. F., Lin, K., Hewitt, J., Paranjape, A., Bevilacqua, M., Petroni, F., & Liang, P. (2024). "Lost in the Middle: How Language Models Use Long Contexts." *Transactions of the Association for Computational Linguistics*, 12, 157-173. [Link](https://aclanthology.org/2024.tacl-1.9/)

LMDB Project. (n.d.). *LMDB documentation.* [Link](https://lmdb.readthedocs.io/en/latest/)

Maystre, L., Ortega Gonzalez, A., Park, C., Dolga, R., Berariu, T., Zhao, Y., & Ciosek, K. (2025). "When Embedding Models Meet: Procrustes Bounds and Applications." OpenReview submission. [Link](https://openreview.net/forum?id=DLEzSo1DIk)

Mitchell, M., Wu, S., Zaldivar, A., Barnes, P., Vasserman, L., Hutchinson, B., et al. (2019). "Model Cards for Model Reporting." *FAT 2019*. [Link](https://arxiv.org/abs/1810.03993)

Moreau, L., & Missier, P. (eds.). (2013). "PROV-DM: The PROV Data Model." W3C Recommendation. [Link](https://www.w3.org/TR/prov-dm/)

NIST. (2023). *Artificial Intelligence Risk Management Framework (AI RMF 1.0).* [Link](https://www.nist.gov/itl/ai-risk-management-framework)

Packer, C., Wooders, S., Lin, K., Fang, V., Patil, S. G., Stoica, I., & Gonzalez, J. E. (2023). "MemGPT: Towards LLMs as Operating Systems." arXiv:2310.08560. [Link](https://arxiv.org/abs/2310.08560)

Park, J. S., O'Brien, J. C., Cai, C. J., Morris, M. R., Liang, P., & Bernstein, M. S. (2023). "Generative Agents: Interactive Simulacra of Human Behavior." *UIST 2023*. arXiv:2304.03442. [Link](https://arxiv.org/abs/2304.03442)

PostgreSQL Global Development Group. (n.d.). *PostgreSQL Documentation.* [Link](https://www.postgresql.org/docs/current/)

Qdrant. (n.d.). "Migrate to a New Embedding Model with Zero Downtime in Qdrant." [Link](https://qdrant.tech/documentation/tutorials-operations/embedding-model-migration/)

Satyanarayanan, M. (2017). "The Emergence of Edge Computing." *Computer*, 50(1), 30-39. [Link](https://elijah.cs.cmu.edu/DOCS/satya-edge2016.pdf)

Sculley, D., Holt, G., Golovin, D., Davydov, E., Phillips, T., Ebner, D., et al. (2015). "Hidden Technical Debt in Machine Learning Systems." *NeurIPS 2015*. [Link](https://papers.nips.cc/paper/5656-hidden-technical-debt-in-machine-learning-systems)

Squire, L. R. (2004). "Memory Systems of the Brain: A Brief History and Current Perspective." *Neurobiology of Learning and Memory*, 82(3), 171-177. [Link](https://pubmed.ncbi.nlm.nih.gov/15464402/)

Stein, E. W., & Zwass, V. (1995). "Actualizing Organizational Memory with Information Systems." *Information Systems Research*, 6(2), 85-117. [Link](https://pubsonline.informs.org/doi/10.1287/isre.6.2.85)

TensorFlow. (n.d.). *ML Metadata.* [Link](https://www.tensorflow.org/tfx/guide/mlmd)

Thakur, N., Reimers, N., Ruckle, A., Srivastava, A., & Gurevych, I. (2021). "BEIR: A Heterogeneous Benchmark for Zero-shot Evaluation of Information Retrieval Models." *NeurIPS Datasets and Benchmarks*. arXiv:2104.08663. [Link](https://arxiv.org/abs/2104.08663)

TidesDB. (n.d.). *TidesDB C API Reference.* [Link](https://tidesdb.com/reference/c/)

Trivedi, H., Balasubramanian, N., Khot, T., & Sabharwal, A. (2023). "Interleaving Retrieval with Chain-of-Thought Reasoning for Knowledge-Intensive Multi-Step Questions." *ACL 2023*. arXiv:2212.10509. [Link](https://arxiv.org/abs/2212.10509)

Tulving, E. (2002). "Episodic Memory: From Mind to Brain." *Annual Review of Psychology*, 53, 1-25. [Link](https://www.annualreviews.org/doi/10.1146/annurev.psych.53.100901.135114)

Unstructured. (n.d.). *Unstructured open source overview.* [Link](https://docs.unstructured.io/open-source/introduction/overview)

van der Aalst, W. M. P., Weijters, A. J. M. M., & Maruster, L. (2004). "Workflow Mining: Discovering Process Models from Event Logs." *IEEE Transactions on Knowledge and Data Engineering*, 16(9), 1128-1142. [Link](https://www.vdaalst.com/publications/p245.pdf)

Vejendla, H. (2025). "Drift-Adapter: A Practical Approach to Near Zero-Downtime Embedding Model Upgrades in Vector Databases." *Proceedings of EMNLP 2025*, 15938-15949. [Link](https://aclanthology.org/2025.emnlp-main.805/)

Walsh, J. P., & Ungson, G. R. (1991). "Organizational Memory." *Academy of Management Review*, 16(1), 57-91. [Link](https://www.jstor.org/stable/258607)

Weaviate. (n.d.). "Collection aliases." [Link](https://docs.weaviate.io/weaviate/manage-collections/collection-aliases)

XTDB. (n.d.). "What is XTDB?" [Link](https://docs.xtdb.com/intro/what-is-xtdb.html)

Yoon, J., Sinha, R., Arik, S. O., & Pfister, T. (2024). "Matryoshka-Adaptor: Unsupervised and Supervised Tuning for Smaller Embedding Dimensions." *EMNLP 2024*, 10318-10336. [Link](https://aclanthology.org/2024.emnlp-main.576/)

Zellinger, L., Stephan, A., & Roth, B. (2024). "Counterfactual Reasoning with Knowledge Graph Embeddings." *EACL 2024*, 2753-2772. [Link](https://aclanthology.org/2024.eacl-long.168/)

Zhang, A. L., Kraska, T., & Khattab, O. (2026). "Recursive Language Models." arXiv:2512.24601. [Link](https://arxiv.org/abs/2512.24601)
