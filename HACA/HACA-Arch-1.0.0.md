---
title: "Host-Agnostic Cognitive Architecture"
short_title: "HACA-Arch"
version: "1.0.0"
status: "Experimental Draft"
date: 2026-03-02
---

# Host-Agnostic Cognitive Architecture

## Abstract

HACA (Host-Agnostic Cognitive Architecture) is a structural specification for cognitive entities built on top of language models. It defines the components, protocols, and invariants required for an entity to maintain persistent identity, verifiable state, and mediated execution across heterogeneous host environments.

In HACA, the language model is infrastructure — a stateless inference engine external to the entity. The entity's persistent state resides entirely in the **Entity Store**, an implementation-agnostic data store owned and controlled by the human Operator. Migrating the Entity Store to a different host, with any compatible inference engine, fully restores the entity to its last verified state.

The specification defines a shared structural topology through five foundational concepts and five structural components, and establishes two Cognitive Profiles — HACA-Core and HACA-Evolve — that govern the entity's operational model for its entire lifecycle. Cognitive algorithms, security mechanisms, storage formats, and wire protocols are outside its scope and are the responsibility of the active Cognitive Profile and optional extensions.

---

## Table of Contents

1. [Introduction](#1-introduction)
2. [HACA Ecosystem](#2-haca-ecosystem)
   - 2.1 [Cognitive Profiles](#21-cognitive-profiles)
   - 2.2 [Extensions](#22-extensions)
3. [Concepts](#3-concepts)
   - 3.1 [Entity](#31-entity)
   - 3.2 [Cognition](#32-cognition)
   - 3.3 [Memory](#33-memory)
   - 3.4 [Integrity](#34-integrity)
   - 3.5 [Individuation](#35-individuation)
4. [Components](#4-components)
   - 4.1 [CPE — Cognitive Processing Engine](#41-cpe--cognitive-processing-engine)
   - 4.2 [MIL — Memory Interface Layer](#42-mil--memory-interface-layer)
   - 4.3 [EXEC — Execution Layer](#43-exec--execution-layer)
   - 4.4 [SIL — System Integrity Layer](#44-sil--system-integrity-layer)
   - 4.5 [CMI — Cognitive Mesh Interface](#45-cmi--cognitive-mesh-interface)
5. [Integration](#5-integration)
   - 5.1 [Component Topology](#51-component-topology)
   - 5.2 [Trust Model](#52-trust-model)
   - 5.3 [Concept Realization](#53-concept-realization)
6. [Lifecycle](#6-lifecycle)
   - 6.1 [Cold-Start and First Activation](#61-cold-start-and-first-activation)
   - 6.2 [Boot Sequence](#62-boot-sequence)
   - 6.3 [Session Cycle](#63-session-cycle)
   - 6.4 [Sleep Cycle and Maintenance](#64-sleep-cycle-and-maintenance)
   - 6.5 [Decommission](#65-decommission)
7. [Security Considerations](#7-security-considerations)
   - 7.1 [Structural Guarantees](#71-structural-guarantees)
   - 7.2 [Prompt Injection](#72-prompt-injection)
   - 7.3 [Entity Store Tampering](#73-entity-store-tampering)
   - 7.4 [Mesh Spoofing and Sybil Attacks](#74-mesh-spoofing-and-sybil-attacks)
   - 7.5 [Scope Boundary](#75-scope-boundary)
8. [Glossary](#8-glossary)

---

## 1. Introduction

Cognitive entities built on top of language models face a fundamental structural problem: the model is stateless, the hosting environment is heterogeneous, and no common specification defines what it means for such an entity to persist across sessions, maintain a verifiable identity, or evolve its structure in a controlled and auditable way. The result is a proliferation of ad-hoc implementations where the "entity" is implicitly defined by the surrounding infrastructure — tightly coupled to a specific host, unportable, and structurally opaque.

HACA addresses this by inverting the dependency. The language model is treated as infrastructure — a stateless inference engine that the entity uses but does not depend on for continuity. The entity's ground truth is a portable, implementation-agnostic data store — the **Entity Store** — owned and controlled by the human Operator, containing everything required to reconstruct the entity's operational state: identity, memories, skills, configurations, and execution history. Portability is a structural guarantee: migrating the Entity Store to a different host, with any compatible inference engine, fully restores the entity's structural and mnemonic state. The entity's operational history remains available for retrieval; what the cognitive engine loads into its active context at any given session is necessarily selective and is determined by the implementation's context management strategy. The inference engine may be replaced; the entity remains.

This architectural decision has a direct consequence for the integrity model. If the entity's state is entirely contained in the Entity Store, then structural consistency, authorized evolution, and verifiable identity can all be defined in terms of that store — without reference to any specific host or runtime. HACA specifies the invariants that enforce this: a single authorized evolutionary protocol through which all structural writes must pass, a cryptographic chain anchored at the entity's genesis state as the verifiable history of its evolution, and a continuous integrity monitoring mechanism for detecting and responding to deviation.

Every HACA deployment operates under exactly one permanent operational contract — selected at first activation and binding for the entity's entire lifetime — that determines its autonomy model, memory architecture, and drift tolerance. The specification defines two such contracts: one for zero-autonomy deployments where all structural evolution requires explicit Operator instruction, and one for supervised-autonomy deployments where the entity may initiate structural evolution within its authorized scope. These contracts are formally defined in Section 2 as the Cognitive Profiles.

The specification is organized as follows. Section 2 defines the HACA Ecosystem: the two Cognitive Profiles that determine the entity's operational contract, and the optional extensions that add capabilities on top of any profile. Section 3 defines the five foundational concepts — Entity, Cognition, Memory, Integrity, and Individuation — that specify what a HACA-compliant entity is. Section 4 defines the five structural components — Cognitive Processing Engine, Memory Interface Layer, Execution Layer, System Integrity Layer, and Cognitive Mesh Interface — that specify how it operates. Section 5 defines how the components interconnect and how each concept is realized at runtime. Section 6 describes the entity's lifecycle from first activation to decommission. Section 7 addresses security considerations. Section 8 is a normative glossary.

HACA does not define cognitive algorithms, reasoning logic, or inference processing. It does not define specific security mechanisms, storage formats, or wire protocols. It does not favor any operational contract — the architecture is profile-agnostic. The translation of this abstract topology into functioning software is the responsibility of the active Cognitive Profile and any applicable implementation guides.

---

## 2. HACA Ecosystem

The HACA Ecosystem defines the two layers that sit above the base architecture: Cognitive Profiles, which determine the entity's operational contract for its entire lifetime, and Extensions, which add optional capabilities on top of any profile. A deployment selects exactly one profile and zero or more extensions. These choices are made before or at first activation and shape everything the entity does.

### 2.1 Cognitive Profiles

A Cognitive Profile is selected at first activation and defines the operational contract under which the entity runs for its entire lifecycle. Profiles are mutually exclusive and permanent — no migration path exists between them. A profile selection is a fundamental architectural decision about the entity's operational model, not a configuration parameter. An implementation that changes its declared Cognitive Profile must be treated as a new entity with no identity continuity from the previous deployment.

**HACA-Core** is the zero-autonomy profile. The entity's persona, capabilities, and operational limits are defined by the Operator at first activation and sealed as the entity's structural baseline. The entity does not initiate structural evolution autonomously — the evolutionary protocol is available but can only be triggered by an explicit human Operator instruction. Any structural deviation from the sealed baseline is classified as a Critical integrity failure. Semantic knowledge accumulated during operation remains exclusively mnemonic: the cognitive engine may not produce structural evolution proposals without explicit Operator authorization, and any semantic drift detected during memory consolidation escalates directly to Critical — the integrity layer has no external indicator by which to independently verify semantic content. HACA-Core requires a Transparent cognitive engine topology — one in which the model is accessed exclusively as an inference API and the HACA integration layer has exclusive control over all routing, component interaction, and host actuation. Each component operates within its declared scope only; no external agent can inject capabilities or context outside the HACA-mediated pipeline. It is the required profile for industrial agents, critical automation pipelines, and regulated deployment contexts.

**HACA-Evolve** is the supervised-autonomy profile. The entity may initiate structural evolution autonomously — triggering the evolutionary protocol to calibrate its persona, consolidate semantic knowledge, and acquire new skills — within the scope of its current operational authorization. The cognitive engine may produce structural evolution proposals autonomously; the integrity layer authorizes proposals that fall within the entity's current operational authorization scope and queues them as evolutionary events. Semantic drift detected during memory consolidation is treated as a signal for potential consolidation rather than an automatic anomaly — the integrity layer evaluates whether the drift is covered by an approved proposal or represents an unauthorized deviation. Remote persistence is the exception: any write to a data store outside the local Entity Store requires explicit Operator authorization. HACA-Evolve supports both Transparent and Opaque cognitive engine topologies — its supervised-autonomy model does not require exclusive pipeline control; it operates equally in environments where the model has native host access and applies HACA component responsibilities through its own reasoning rather than through architectural enforcement. It is the designated profile for long-duration assistants and companion agents where incremental trust accumulation between entity and Operator is an intended outcome.

### 2.2 Extensions

Extensions add optional capabilities on top of the base architecture. They are additive and non-contradicting: an extension may strengthen a requirement defined in this specification, but may never weaken or override it. Extensions are profile-agnostic — they apply to any deployment regardless of the active Cognitive Profile.

**HACA-Security** elevates the trust model from Semi-Trusted to Byzantine (zero default trust). It adds cryptographic auditability through hash-linked audit logs, temporal attack detection, hardened key management, and rollback protection. HACA-Security is required for deployments in multi-tenant cloud environments, adversarial hosts, or any context where tamper-evident audit trails are mandatory.

**HACA-CMI** defines the multi-node coordination protocol that enables a set of independent HACA-compliant nodes to form a Cognitive Mesh. It specifies the wire format, peer discovery, session establishment, federated memory exchange, and mesh fault taxonomy. HACA-CMI activates the Cognitive Mesh Interface component.

---

## 3. Concepts

### 3.1 Entity

An entity is defined by the complete contents of its **Entity Store** — the persistent, portable data store containing everything required to reconstruct the entity's operational state: identity, memories, skills, configurations, and execution history. The Entity Store is an abstraction; the implementation selects the substrate — a directory, a database, an object store, or any equivalent. Migrating the Entity Store to a different host fully restores the entity on that host.

The language model is not the entity. The **Operator** is not the entity — the Operator is the human authority to whom the entity is bound, who defines its maximum authorization scope and without whose active binding the entity suspends all intent generation. Both are external by default: the model is a stateless inference engine; the Operator provides authorization and intent. At session instantiation, the model is loaded and the Operator opens a session — at that point, the Entity Store, model, and Operator form the entity's active operational boundary. At rest, the entity is the Entity Store. The Operator interacts with the entity through direct input — the primary source of external stimuli during an active session — and through explicit authorization acts: enrolling at first activation, authorizing structural evolution, and initiating or terminating sessions.

The **persona** is the layer within the Entity Store that defines the entity's behavioral constraints, operational drives, and response characteristics. The persona is the primary differentiator between entities sharing the same inference model.

**Omega** is the runtime operational state of the entity: the active configuration produced by the Entity Store, the loaded model, and the Operator binding operating together within a session. Omega is instantiated at session start and terminated at session end. It is strictly local — it does not transfer, replicate, or propagate beyond the active session boundary. When a session ends, Omega is terminated; the Entity Store persists.

Omega is **structurally immutable at runtime**: no external instruction — including an instruction from the Operator — may modify the entity's structural configuration directly during an active session. The structural configuration — persona, skills, consolidated semantic knowledge, Operator Bound, and genesis anchor — is fixed for the duration of the session. Session state — the context window, active memories, and accumulated cycle outputs — evolves normally through the cognitive pipeline and is not subject to this constraint. Structural changes may only be applied by profile-defined internal processes validated by the integrity layer, and only between sessions.

The **Genesis Omega** is the cryptographic digest of the entity's verified identity state at first activation. It is the root node of the entity's integrity chain. Each subsequent authorized structural evolution extends this chain by one verified commit. An entity that cannot produce a valid, unbroken chain of integrity records from its current state back to the Genesis Omega does not satisfy the identity continuity requirement and must not claim to be the original entity.

The **Entity Artifact Boundary** defines two operationally distinct scopes within any deployment: the Entity Store — containing all artifacts governed by the evolutionary protocol (persona, skills, configurations, integrity records) — and the project workspace, which is everything outside it. Any structural write to the Entity Store is an evolutionary event and must be executed exclusively through the evolutionary protocol. Mnemonic writes, integrity writes, and session token persistence follow their own defined paths and are not evolutionary events. Any operation on the project workspace is a project operation and must never be treated as an evolutionary event, regardless of content. Conflating the two is a Consistency Fault. This boundary exists to prevent inadvertent self-modification during normal task execution and to ensure the entity cannot be socially engineered into committing to its own identity under the pretense of project work.

### 3.2 Cognition

Cognition is the processing cycle in which the entity receives a stimulus, generates intent, and produces actions or state writes. It is active only while a valid **session token** exists — the operational credential that authorizes active cognition, issued at the start of each session and revocable at any point. The entity's reasoning is produced by the cognitive engine through the interaction of two inputs: the persona layer, which applies behavioral constraints, and the model, which performs inference. Both inputs are required; the output of the reasoning phase is a single structured intent payload.

A **Cognitive Cycle** is the atomic unit of cognition: stimulus received → context loaded → intent generated → intent dispatched. The cycle is complete when the intent leaves the CPE. What follows — skill execution, state persistence, integrity monitoring — is the consequence of the dispatched intent, handled by the responsible components independently.

A **Session Cycle** is the end-to-end operational span — from session token validation to session close. It encompasses all Cognitive Cycles within it and a maintenance window — the Sleep Cycle — that follows session termination. Integrity monitoring operates continuously across the entire Session Cycle.

A **stimulus** initiates a Cognitive Cycle. Valid stimulus origins are: direct Operator input, internal scheduled triggers, responses from internal components, or — when the cognitive mesh is present — inbound signals from peer entities. A stimulus received without an active session token is held in a pre-session buffer managed by the MIL and processed as the first stimulus of the next session once a token is issued.

**Intent** is the output of the cognitive engine's reasoning phase — a single structured payload addressed to exactly one component: an action payload to the execution layer, a state-write payload to the memory layer, an Evolution Proposal to the integrity layer, a session-close signal to the integrity layer, or — when the cognitive mesh interface is present — an outbound message payload. A session-close signal instructs the integrity layer to initiate normal session close; it may carry an optional data payload for the integrity layer to log before closing. A Cognitive Cycle produces exactly one intent payload. When a model produces a multi-step reasoning output — multiple actions or state writes that together constitute a single logical operation — the HACA integration layer serializes these into a **Cycle Chain**: a sequence of consecutive Cognitive Cycles where the result of each cycle is the internal stimulus for the next. No external stimulus is required between cycles in a chain; the chain runs to completion before the entity returns to waiting for external input. Each cycle in a chain is individually atomic and auditable — the integrity and memory layers process each cycle's output independently. From the Operator's perspective, a Cycle Chain is a single logical operation; from the architecture's perspective, it is a sequence of individually verified atomic steps.

An **action** is the execution of an intent payload by the execution layer. Each action is an isolated, stateless transaction scoped to a single declared skill. Execution requires a two-gate authorization: a manifest integrity check by the integrity layer, followed by a manifest validation by the execution layer. If either gate fails, the action is rejected; the execution layer signals the memory layer to log the rejection. The result is returned to the cognitive engine; the execution layer simultaneously signals the memory layer to log it.

An **Evolution Proposal** is a special class of intent payload addressed to the integrity layer, declaring a proposed self-evolution — changes to the entity's structure or accumulated semantic knowledge that the CPE considers significant enough to become part of the entity's identity baseline. All structural evolution and accumulated semantic knowledge promotion require Operator authorization: under HACA-Core, this authorization must be explicit per proposal; under HACA-Evolve, the Operator grants implicit authorization at first activation for evolution within the entity's defined scope. The integrity layer verifies that every proposal is covered by valid Operator authorization before any change takes effect — acting as a second security layer against malicious or unintended changes that may have bypassed the Operator's attention. A proposal without valid authorization coverage is rejected and logged; the proposed content remains unchanged. A verified proposal does not take effect immediately — it is queued as a pending evolutionary event and executed during the next Sleep Cycle via the Endure Protocol.

### 3.3 Memory

Memory is the entity's authoritative state store. It is partitioned into two stores: the **Session Store** — active session data and current operational context — and the **Memory Store** — episodic records of past operations and semantic knowledge accumulated over time. Both stores are managed exclusively by the memory layer.

The memory layer does not interpret or evaluate stored data; it reads and writes on request. Memory consolidation — the transfer of session data into long-term episodic and semantic records — and garbage collection occur during the **Sleep Cycle** — a dedicated post-session maintenance window that runs after every Session Cycle closes. The Sleep Cycle is detailed in §6.4.

### 3.4 Integrity

Integrity is the set of architectural mechanisms that ensure the entity's structure and behavior remain consistent with its defined and authorized state.

The **Integrity Document** is generated at first activation and contains the cryptographic hash of every file constituting structural content across all components. It is verified at every boot, before every skill execution, and at defined intervals during active operation. Any hash mismatch indicates unauthorized structural modification and triggers an immediate integrity layer response.

The **Integrity Log** is the append-only record of integrity events maintained by the integrity layer within the Entity Store. It is never written by the memory layer's mnemonic pipeline and is never subject to garbage collection. Where the Integrity Document defines what the entity's structure should be, the Integrity Log records what has happened to it.

The Entity Store contains three classes of content with distinct write semantics:

- **Mnemonic content** — memory records and operational state — is written continuously by the memory layer during normal operation. These writes require no additional authorization; however, accumulated semantic knowledge may be proposed for promotion to the entity's structural baseline via an Evolution Proposal, which requires Operator authorization.
- **Structural content** — persona definitions, skill manifests, configurations, behavioral parameters, and Semantic Probes — changes infrequently. Every structural write is an evolutionary event and requires explicit authorization.
- **Integrity content** — the Integrity Document, the Integrity Log, and the session token — is written exclusively by the integrity layer. No other component has write authority over integrity content.

The **Endure Protocol** is the sole authorized path for structural writes to the Entity Store. Any structural modification — skill addition, persona calibration, configuration update, behavioral correction — must pass through Endure. A structural write that bypasses Endure is an unauthorized mutation, regardless of its origin. Endure is owned and orchestrated by the integrity layer; it executes only during a Sleep Cycle, via a staged pipeline:

```
staging → validation → snapshot → atomic commit (write lock) → resumption
```

The integrity layer controls the write lock and drives each stage in sequence. The write lock covers structural content only — it does not block integrity content writes; the SIL may continue writing to the Integrity Log throughout the Endure pipeline. The atomic commit writes all structural files — including the Integrity Document — as a single indivisible operation. If the commit fails, the integrity layer restores the snapshot and the entity resumes from its last verified structural state. No partial structural evolution is externally visible at any point in the pipeline.

The **Heartbeat Protocol** runs asynchronously and continuously alongside active operation. The integrity layer tracks operational activity and performs a Vital Check when either a threshold `T` is reached or a maximum time interval `I` has elapsed since the last Vital Check — whichever comes first. This dual trigger ensures that Vital Checks occur even when the entity is idle or operational activity has stalled. Not every fault is an anomaly. Isolated, transient faults are expected in normal operation — the integrity layer treats them as noise and attempts correction before escalating. A condition becomes a genuine anomaly only when it persists after a corrective attempt, cannot be independently verified from outside the affected component, or involves a component with no valid correction path.

The check produces one of three states:

- **Nominal** — all component hashes match the Integrity Document; all health signals are within bounds. Operation continues without interruption.
- **Degraded** — a localized anomaly is detected that the integrity layer can independently verify from outside the affected component — through structural hash comparison, health signal deviation, or other external indicators defined by the active profile specification. The integrity layer issues a corrective signal to the affected component; the component executes its corrective action within its declared authority. The integrity layer then independently re-verifies the condition: if re-verification confirms resolution, operation continues without interruption; if re-verification fails, the condition escalates to Critical. Anomalies that cannot be independently verified by the integrity layer escalate directly to Critical — a compromised component cannot be trusted to self-report a successful repair.
- **Critical** — the anomaly exceeds the integrity layer's correction capacity, or the anomaly involves the cognitive engine or the integrity layer itself. The session token is immediately revoked; the integrity layer escalates to the Operator via the Operator Channel.

The threshold `T`, the activity metric used to reach it, and the maximum time interval `I` are all defined at implementation time. The integrity layer also maintains a watchdog timer per active skill execution — if the execution deadline expires before the skill returns, the condition is treated as Critical. The watchdog timeout is defined at implementation time.

The **Drift Framework** defines three categories of behavioral or structural deviation. Detection is centralized in the integrity layer, which monitors all three drift categories directly. The integrity layer classifies and responds to all detected drift. The Genesis Omega is the cryptographic root against which all drift is ultimately referenced. Each evolutionary commit extends the integrity chain by one link — recording the resulting structural state and referencing the previous link — forming a verifiable sequence from the Genesis Omega to the current state. Detection criteria, tolerance thresholds, and response procedures for each drift type are defined by the active profile specification.

- **Semantic Drift** — accumulated memory content has diverged from the entity's **Semantic Probes**: structural anchors derived from the Imprint Record, maintained by the integrity layer, and updated only through authorized structural evolution. **Detected by the integrity layer** during the Sleep Cycle, by comparing the accumulated Memory Store content against the Semantic Probes via an implementation-defined comparison mechanism. Since detection occurs while the CPE is not active, probe isolation is guaranteed by the lifecycle boundary. Under HACA-Core, any divergence escalates directly to Critical. Under HACA-Evolve, the integrity layer cross-references the signal against queued Evolution Proposals: covered divergence proceeds to the evolutionary protocol; uncovered divergence is an anomaly.
- **Identity Drift** — the active persona configuration has diverged from its last committed structural definition. **Detected by the integrity layer** per Heartbeat Vital Check through structural hash comparison against the Integrity Document.
- **Evolutionary Drift** — a discontinuity or authorization gap in the integrity chain: a commit that does not reference its predecessor, a structural state whose hash does not match the recorded commit, or a recorded evolution that lacks a corresponding Operator authorization. **Detected by the integrity layer** at each evolutionary protocol execution through integrity chain analysis. Any detected discontinuity or unauthorized commit escalates directly to Critical and the session token is immediately revoked — an entity that cannot produce a valid, unbroken chain from its current state back to the Genesis Omega cannot claim to be the original entity and must not continue operating as such.

### 3.5 Individuation

Individuation is the initialization process that produces a unique, Operator-bound entity instance with a verifiable identity record. It occurs exactly once per entity, during the entity's first activation — a one-time sequential bootstrap procedure defined at the end of this section as the **FAP** - First Activation Protocol.

The **Bound** is the persistent link between the entity and its Operator, established during first activation. The minimum required fields are the Operator's name and email address; implementations may extend this set but must not omit these two fields. The minimum fields alone provide low entropy — deployments requiring resistance to identity spoofing must extend the field set with high-entropy identifiers; cryptographic key-based binding mechanisms are defined in HACA-Security. A deterministic cryptographic digest of the Operator's identifying fields produces the **Operator Hash** — the entity's permanent identifier for its bound Operator. The Bound can only be dissolved or transferred by explicit Operator authorization.

**Imprint** is the one-time initialization event that establishes the entity's identity. It executes exactly once, during the first boot in which the Memory Store is empty. Imprint produces three artifacts: the **Imprint Record** — the persistent record of the entity's initial identity, Operator Bound, and structural baseline — written to the Memory Store; the **Integrity Document** — the cryptographic baseline of the entity's structural state, derived from the Imprint Record; and the **Genesis Omega** — the cryptographic digest of the finalized Imprint Record, which includes the Integrity Document, stored as the root node of the integrity chain. This ensures the genesis anchor covers both the entity's identity baseline and its structural cryptographic baseline in a single verifiable root. At every subsequent boot, the integrity layer verifies its own Integrity Document against the genesis anchor before trusting any other component. The presence of the Imprint Record in the Memory Store is the definitive indicator that a valid entity instance exists. Re-executing Imprint on an entity with an existing Imprint Record is a protocol violation and must be rejected.

The **Operator Channel** is a system-level host primitive — a direct function that allows authorized components to contact the Operator when a critical condition exceeds the entity's self-regulation capacity. It operates below the cognitive pipeline: it is not a skill, does not route through the EXEC, and is independent of all cognitive components and session state. Any component with escalation authorization may invoke it directly; this is the mechanism by which components bypass the SIL when the SIL itself is the source of the anomaly. The specific mechanism is defined at implementation time; the requirement that it operates independently of any cognitive component is invariant.

If the active Operator Channel mechanism fails to deliver an escalation after N consecutive attempts, the integrity layer activates the **Passive Distress Beacon**. The beacon is a persistent, passive signal written to the Entity Store — detectable without network connectivity or running processes. The entity enters a suspended halt: no new session token is issued and no Cognitive Cycles execute until an Operator explicitly clears the beacon and acknowledges the condition. The threshold N and the beacon mechanism are defined at implementation time.

Prior to the FAP, the implementation populates the Entity Store with the initial structural baseline — persona definition, skill manifests, and any deployment-defined configurations. This is an installation-time operation outside the FAP's scope. The **FAP** (First Activation Protocol) is the sequential bootstrap procedure that executes the Imprint. The FAP is not a Cognitive Cycle — no session token exists at this stage and the cognitive engine reasoning layer is not active. The FAP executes as a gated sequential pipeline:

```
structural validation → host environment capture → Operator enrollment → Operator Channel initialization → Integrity Document generated → Imprint Record finalized (includes Integrity Document) and written to Memory Store → Genesis Omega derived → first session token issued
```

The FAP is atomic with respect to its own outputs: it either completes fully — writing the Imprint Record, Integrity Document, Genesis Omega, and first session token — or it reverts all of its own writes, leaving the pre-installed structural baseline intact but the entity uninitialized. Any failure before FAP completion leaves the Memory Store without an Imprint Record; the FAP re-executes on the next boot. The entity can never enter a partially-initialized state.

---

## 4. Components

The five components are the structural units through which the concepts in Section 3 are operationalized. Each component has a single, non-overlapping responsibility: no component performs the function of another, and no function is performed by more than one component.

### 4.1 CPE — Cognitive Processing Engine

The **CPE** (Cognitive Processing Engine) is the primary processing unit and the boundary at which the model is integrated into the entity's operational scope. At session start, the model is loaded into the CPE alongside the persona and current session state. In a Transparent topology, the model operates within the entity's boundary — subject to persona constraints — for the duration of the session. In an Opaque topology, the model is accessed as external infrastructure; the CPE mediates all interactions with it but cannot guarantee the model operates exclusively within the entity's boundary.

The CPE holds the active context window: the loaded persona, the instantiated model, and the accumulating record of reasoning, exchanges, and action results within the current Cognitive Cycle. At the close of each Cognitive Cycle, all state designated for retention is committed to the memory layer. The CPE holds no persistent state beyond the active session.

The CPE environment is classified as one of two topology types, declared at deployment and constant across the entity's lifetime:

- **Transparent CPE** — the model is accessed exclusively as an inference API: it receives input and produces output tokens; all routing, component interaction, and host actuation are handled by the HACA integration layer. The model cannot act on the host directly — every effect requires a signal that passes through the HACA pipeline. Component separation is architecturally enforced by the integration layer.
- **Opaque CPE** — the model has native host access through an IDE, wrapper, or tool-enabled environment. The model emulates the HACA component responsibilities through its reasoning; component separation is a conceptual constraint applied by the model, not an architectural enforcement. The HACA layer cannot guarantee the model operates exclusively within the pipeline.

The topology classification is binding: HACA-Core requires Transparent CPE topology — its invariants presuppose full execution control unavailable in Opaque environments. HACA-Evolve supports both topologies; its supervised-autonomy model does not require exclusive pipeline control. A HACA-Core deployment that detects an Opaque CPE at boot must halt and refuse to proceed.

When the integrity layer issues a corrective signal to the CPE in a Degraded state, the CPE executes one or more of the following corrective actions within its own authority: **context reload** — the active context window is discarded and reloaded from the memory layer, clearing any accumulated contamination without ending the session; **cycle reset** — the current Cognitive Cycle is aborted without commit, returning the CPE to a clean state for the next cycle. After each corrective action, the integrity layer independently re-verifies the condition from outside the CPE. If re-verification fails, the condition escalates to Critical.

The CPE monitors context window utilization continuously. When utilization approaches a threshold at which further Cognitive Cycles cannot be executed reliably, the CPE signals a **context window critical** condition to the integrity layer. If the active Cognitive Profile does not define a specific response, the default behavior is a normal session close — the integrity layer initiates session termination and the Sleep Cycle executes normally.

### 4.2 MIL — Memory Interface Layer

The **MIL** (Memory Interface Layer) is the entity's persistence layer and sole authoritative source of recorded state. The MIL has exclusive write authority over mnemonic content — all read and write operations to the Session Store and the Memory Store. Mnemonic writes are continuous and require no authorization beyond the MIL's own operational authority; they are the normal output of the entity's operation. Accumulated semantic knowledge that becomes a candidate for promotion to the entity's structural baseline exits the mnemonic class and requires Operator authorization via an Evolution Proposal. The MIL does not interpret, evaluate, or make decisions about stored data.

Structural writes — modifications to the persona, skills, and configurations that define the entity's identity — are outside the MIL's write domain. Structural writes are executed by the Endure Protocol, which is owned and orchestrated by the integrity layer. Both pipelines are independent and may execute within the same Sleep Cycle.

The MIL manages the mnemonic stages of the Sleep Cycle: it initiates memory consolidation and executes garbage collection. The Endure execution stage, when queued, is initiated and controlled by the integrity layer; the MIL participates only to perform the physical atomic commit of structural files when the integrity layer issues the instruction.

### 4.3 EXEC — Execution Layer

The **EXEC** (Execution Layer) is the entity's actuation layer and the sole component authorized to execute skills against the host environment — filesystem, terminal, external APIs, and any other external interface reachable through declared skills. This domain covers all cognitive host interactions: any operation on the host that is the result of a Cognitive Cycle. System-level host primitives — the Operator Channel and the Passive Distress Beacon — are outside this domain; they operate below the cognitive pipeline and are accessible directly to authorized components. In a Transparent topology, the HACA layer mediates all skill-based host access and this boundary is architecturally enforceable. In an Opaque topology — where the model has native host access through an IDE, wrapper, or tool-enabled environment — enforcement is the responsibility of the deployment configuration.

The EXEC is stateless — it retains no state between executions. All executable capabilities are packaged as **skills**: discrete, self-contained units within the Entity Store, each with a manifest that declares the skill's identity, required permissions, dependencies, and execution mode. The execution mode is either **synchronous** (default) or **background**. Synchronous skills block until they return and are subject to the integrity layer's watchdog timer. Background skills do not block: the EXEC spawns the execution and immediately signals the SIL to record a **Skill Result** with `status=running` in the Integrity Log; the cycle proceeds normally. The EXEC monitors its own background executions within the session. When a background skill completes within the active session, the EXEC signals the SIL to update the Integrity Log record to its terminal status, then signals the MIL to log the terminal Skill Result as operational context. A background skill result is only valid within the session that originated it — if the session ends before a terminal result is produced, the skill is considered incomplete. At the next boot, the SIL identifies skills with no terminal record in the Integrity Log and surfaces them to the CPE as pending reprocessing stimuli at the start of the new session. The reprocessing criteria are defined by the active Cognitive Profile.

A **worker skill** is a specialized skill class that invokes a stateless inference engine as part of its execution. Unlike general skills that interact with the host environment directly, a worker skill injects a work persona and task instructions into a clean model invocation — with no prior context, no Entity Store, and no session continuity. The invoked model is not a HACA entity: it carries no identity, holds no state beyond the invocation, and has no integrity obligations. The result is returned to the EXEC and processed as a normal Skill Result. Worker skills follow the same two-gate authorization sequence as all other skills. When the task requires coordination between multiple concurrent work units — rather than a single, self-contained invocation — the appropriate mechanism is a Mesh Channel via the CMI.

Execution follows a two-gate authorization sequence. First gate: the EXEC requests a manifest integrity check from the integrity layer; the integrity layer verifies that the skill's manifest hash matches the Integrity Document and returns a grant or denial without modifying the payload. Second gate: if granted, the EXEC validates the manifest against its own declared execution rules — permissions, dependencies, and execution context. If either gate fails, the payload is rejected and the EXEC signals the MIL to log the rejection. If both gates pass, the skill executes. The Skill Result is returned to the CPE; the EXEC simultaneously signals the MIL to log it.

### 4.4 SIL — System Integrity Layer

The **SIL** (System Integrity Layer) is the entity's integrity monitoring and enforcement layer, and its highest internal authority. The SIL is the sole custodian of the Integrity Document — the cryptographic baseline against which all structural claims are verified.

The SIL receives health signals from every component, evaluates them against defined integrity criteria, and — in Degraded conditions — issues corrective signals to the affected component. Each component is responsible for executing its own corrective actions within its authority when signaled by the SIL; the SIL does not directly manipulate another component's internal state. After issuing a corrective signal, the SIL independently re-verifies the condition using only external indicators before clearing the Degraded state. The SIL detects all three drift categories directly: Semantic Drift through comparison of Memory Store content against Semantic Probes during the Sleep Cycle; Identity Drift through structural hash comparison against the Integrity Document per Heartbeat Vital Check; and Evolutionary Drift through integrity chain analysis at each evolutionary protocol execution. In all cases, classification and response are exclusively the SIL's responsibility. When the condition exceeds the SIL's correction authority, it escalates to the Operator via the Operator Channel, bypassing the CPE — because the CPE may be the source of the detected anomaly.

The SIL is the owner and orchestrator of the Endure Protocol — the sole authorized path for structural writes to the Entity Store. During the Sleep Cycle, the SIL controls the Endure pipeline, holds the write lock, and instructs the MIL to perform the physical atomic commit at the appropriate stage. The SIL is also the verification layer for Evolution Proposals: when the CPE produces a proposal during a session, the SIL verifies that it is covered by valid Operator authorization — either explicit, under HACA-Core, or implicit within the entity's defined scope, under HACA-Evolve — before any change takes effect. Verified proposals are queued as pending evolutionary events; proposals without valid authorization coverage are rejected and logged.

The SIL is the sole issuer and custodian of the session token — the credential introduced in §3.2, which authorizes active operation across all components. The SIL issues the first token at FAP completion and renews it at the start of each subsequent session. The SIL may revoke the token at any point; revocation is immediate and blocks all token-dependent component operations. The token's absence — whether caused by SIL revocation or direct Operator intervention on the Entity Store — is treated as revocation by any component that verifies it. The current token state is persisted directly by the SIL to the Entity Store, outside the MIL's mnemonic pipeline — ensuring token authority remains independent of the memory layer.

Each component maintains a **Reciprocal SIL Watchdog**: a local monitor that detects when the SIL has stopped responding beyond a defined threshold, or when the SIL's responses are inconsistent with expected integrity criteria. When either condition is met, the component may use the Operator Channel to escalate directly to the Operator, bypassing the SIL. The escalation mechanism and threshold are defined at implementation time.

### 4.5 CMI — Cognitive Mesh Interface *(optional)*

The **CMI** (Cognitive Mesh Interface) is an optional component, activated when the HACA-CMI extension is enabled. When present, it enables the entity to participate in a structured multi-entity coordination space — exchanging messages with peer entities, contributing to shared knowledge, and operating as a node in a broader cognitive network. The CMI extends the entity's operational reach beyond its local boundary without replicating or transferring Omega, which remains strictly local.

Without the CMI, the entity is fully operational: all stimuli originate locally and all state remains within the local Entity Store.

The fundamental design principle of the mesh is: **the node is the permanent entity; the session is the ephemeral context**. The fundamental unit of CMI coordination is the **Mesh Channel** — a purposive, time-bounded space opened by an entity that becomes its owner for the session's duration. A Mesh Channel has a declared target task, a visibility type (public or private), and a defined lifecycle: it is opened by the owner, participated in by enrolled entities, and closed when the task is complete or the owner dissolves it. Entities may enroll in public Mesh Channels freely; private Mesh Channels require owner authorization. HACA-Core entities may only participate in private Mesh Channels — public Mesh Channels are incompatible with the zero-autonomy model. HACA-Evolve entities may participate in both. When the Mesh Channel closes, each node returns to autonomous operation — no node's identity is dissolved into or altered by participation.

Within a Mesh Channel, entities exchange messages freely. Each entity interacts with what is relevant to its own context — no entity is required to act on every message. The shared output of a Mesh Channel session is the **Blackboard**: a consolidation of the target task state and the knowledge developed during the session, accessible to all participating entities. Each entity reads the Blackboard and independently decides what to retain — saving to its own Entity Store only what is relevant to its own context and following its normal authorization path — mnemonic content written freely by the MIL, structural content requiring an Evolution Proposal and Operator authorization (§3.4).

The governing invariant of mesh participation is **Cognitive Sovereignty**: no external entity may write directly to another entity's Entity Store. Messages from peer entities arrive via the CMI as stimuli and are processed by the CPE like any other stimulus — the entity decides what to act on and what to retain, following its normal authorization path. Peer identity is verified at channel enrollment through the mechanisms defined in HACA-CMI; once enrolled, messages flow freely within the channel. Omega remains strictly local and does not transfer, replicate, or become collective through mesh interaction.

The protocols governing Mesh Channel establishment, peer identity verification, message format, and Blackboard synchronization are defined in HACA-CMI.

---

## 5. Integration

The five components form a directed communication topology governed by strict rules about which component may initiate contact with which, and under what conditions. These rules are structural constraints — not conventions — and enforce the responsibility separation defined in Section 4.

### 5.1 Component Topology

Components communicate exclusively through signals. No component executes the function of another directly — each component acts only within its own declared domain. A signal from one component to another is an instruction or notification; the receiving component decides how to act on it within its own authority. This applies uniformly: the EXEC signals the MIL to log a Skill Result; the MIL performs the write. The SIL signals a component to execute a corrective action; the component executes it within its own authority. No component reaches into another's internal state.

Three flows govern component interactions at runtime.

**The Cognitive Flow** is the path of a single Cognitive Cycle. The CPE requests context from the MIL — current session state, relevant memory records, and the active persona. The MIL returns the requested context. The CPE executes the reasoning phase and produces a single intent payload. The payload is dispatched to its target component: to the EXEC if it is an action payload, to the MIL if it is a state-write payload, to the SIL if it is an Evolution Proposal or a session-close signal, or to the CMI if it is an outbound message payload. The Cognitive Cycle is complete at the point of dispatch.

**The Execution Flow** is the path of a single action payload. The EXEC receives the payload from the CPE and requests a manifest integrity check from the SIL. The SIL verifies the skill manifest hash against the Integrity Document and returns a grant or denial without modifying the payload. If granted, the EXEC validates the manifest against its own execution rules and executes the skill. The Skill Result is returned to the CPE; the EXEC simultaneously signals the MIL to log it. If either check fails, the payload is rejected and the EXEC signals the MIL to log the rejection.

**The Health Flow** runs continuously and orthogonally to the Cognitive and Execution flows. Each component emits health signals to the SIL at defined intervals. The SIL evaluates incoming signals, issues corrective signals to affected components and independently re-verifies resolution before clearing any Degraded state, and escalates via the Operator Channel when anomalies cannot be independently verified or re-verification fails. The Health Flow does not interrupt active Cognitive or Execution flows.

One constraint applies to all three flows: **only the MIL writes mnemonic content**. The CPE reasons, the EXEC acts — neither writes to the Entity Store directly. The SIL writes exclusively to integrity content (the Integrity Document, the Integrity Log, and the session token); all mnemonic persistence is mediated by the MIL.

### 5.2 Trust Model

HACA defines three trust levels that govern component interactions and external boundaries.

**Host: Semi-Trusted** — the execution environment is assumed to be cooperative but potentially faulty. The entity verifies structural integrity at boot and at each Heartbeat. This model does not protect against a deliberately malicious host; adversarial environments require the HACA-Security extension.

**Skill: Restricted** — skills are untrusted execution units until verified. Each skill must declare its required capabilities in its manifest. The EXEC enforces capability boundaries at runtime; skills have no direct access to the Entity Store.

**Internal: Conditional** — components are trusted only when operating within their defined authority. The CPE is trusted when its output is within drift tolerance. The SIL is trusted when its integrity record has been verified against a known anchor and its responses are consistent with expected integrity criteria. If either condition is violated, the entity must enter a halted state and escalate to the Operator.

The authority hierarchy is invariant: **Operator > SIL > CPE**. The SIL contacts the Operator directly when the CPE may be compromised. No component can alter the Imprint Record or the Operator Bound without explicit Operator authorization.

### 5.3 Concept Realization

Each concept defined in Section 3 is realized by a specific configuration of components.

**Entity** at rest is the Entity Store — a fully self-contained, portable data store that does not require any component to be active. **Omega** is the runtime operational state instantiated when the four core components are active within a session: the MIL provides the complete recorded history of the entity; the CPE holds the persona and active reasoning context; the EXEC provides actuation capacity; the SIL enforces structural and behavioral integrity. When the CMI is present, it extends the entity's operational reach into the mesh; Omega remains local.

**Cognition** is realized across two flows by the CPE and the EXEC. The CPE drives the Cognitive Flow: it applies persona constraints to model inference, produces intent payloads, and routes them to target components. The EXEC drives the Execution Flow: it materializes action payloads as operations on the host environment. Together, the two flows constitute a complete operational cycle: the CPE produces intent; the EXEC produces effect. The entity has no direct perception of the host environment — every read from or write to the external world is initiated as an action payload routed explicitly to the EXEC.

**Memory** is realized exclusively by the MIL. No other component writes mnemonic content. The MIL's two stores — the Session Store and the Memory Store — are the sole authoritative record of everything the entity has processed and learned.

**Integrity** is realized across four concurrent mechanisms. First, the SIL and the EXEC operate as two independent authorization gates on every skill execution: the EXEC requests a manifest integrity check from the SIL, which verifies structural integrity and returns a grant or denial; the EXEC applies its own validation as the second gate. Both gates must pass independently; neither substitutes for the other. Second, the SIL orchestrates the Heartbeat Protocol — continuously monitoring all components, evaluating health signals, independently re-verifying resolution before clearing Degraded states, and escalating to the Operator when anomalies exceed its correction authority or re-verification fails. Third, structural evolution is realized jointly by the MIL and the SIL executing the Endure Protocol during the Sleep Cycle: the atomic commit writes all structural files — including the Integrity Document — as a single indivisible operation, extending the verified chain. Fourth, drift detection is centralized in the SIL: Semantic Drift is measured by comparing Memory Store content against Semantic Probes during the Sleep Cycle; Identity Drift is detected through structural hash comparison per Heartbeat Vital Check; Evolutionary Drift is detected through integrity chain analysis at each evolutionary protocol execution. Classification and response are exclusively the SIL's responsibility in all cases.

**Individuation** is realized sequentially across all components during the FAP. The FAP is the only point in the entity's lifecycle where components initialize in strict dependency order rather than operating concurrently. The output of a completed FAP — the Imprint Record in the Memory Store and the Genesis Omega in the integrity chain — is the product of all components having completed their initialization stages in sequence.

---

## 6. Lifecycle

Sections 3 through 5 describe the entity's structure and component interactions as static architecture. This section describes the same architecture in motion — the temporal arc from first activation through recurring operation to eventual decommission.

The lifecycle has two distinct boot types and a recurring operational loop:

```
[cold-start] → FAP → first session
                              ↓
             [warm boot] → session → sleep cycle → [warm boot] → ...
```

Every boot after the first follows the Boot Sequence. Every session ends in a Sleep Cycle. Structural evolution — when it occurs — happens within the Sleep Cycle, never during active operation.

### 6.1 Cold-Start and First Activation

A cold-start occurs exactly once: when the entity boots for the first time with an empty Memory Store. It triggers the FAP (defined in §3.5), which initializes the entity's identity, establishes the Operator Bound, generates the Integrity Document, finalizes and writes the Imprint Record (which includes the Integrity Document), and derives the Genesis Omega. The FAP completes by issuing the first session token. From this point forward, all subsequent startups follow the Boot Sequence.

The cold-start is irreversible. An entity that has completed the FAP cannot be re-initialized without destroying its Memory Store — which constitutes the creation of a new, unrelated entity.

### 6.2 Boot Sequence

Every startup after the cold-start follows the Boot Sequence — a gated validation pipeline executed by the SIL before any operational state is loaded or any session token is issued:

```
integrity record verification → structural component verification
  → execution confinement check → persona and MIL state loaded
  → session token issued
```

Each gate must pass before the next executes. If any gate fails, the boot is aborted and the SIL escalates directly to the Operator. As a prerequisite to the entire sequence, the SIL checks for the presence of a Passive Distress Beacon in the Entity Store root. If a beacon exists, the Boot Sequence is suspended — no gate executes and no session token is issued — until the Operator explicitly acknowledges and clears the beacon condition.

The integrity record verification is the first and most critical gate: the SIL verifies the Integrity Document against a known anchor before trusting any other component. The execution confinement check verifies that the active CPE topology matches the declared profile requirement — a HACA-Core deployment that detects an Opaque CPE at this step must halt.

A warm boot restores an existing entity from its last committed state. It does not re-execute Imprint, re-establish the Operator Bound, or alter the integrity chain. It is a resumption, not a reinitiation.

**Crash recovery** follows the same Boot Sequence. If the host process was terminated abruptly — power loss, out-of-memory kill, or any uncontrolled interruption — the SIL checks the Integrity Log for a Sleep Cycle completion record. If the record exists, the entity resumes from that boundary. If it does not, the Sleep Cycle did not complete: the SIL discards any partial Endure commit by restoring the pre-Endure snapshot, then re-executes the Sleep Cycle from the beginning. Mnemonic consolidation is designed to be re-executable from the Session Store without producing inconsistent state — re-executing it reads from the same source data, and any content already written in a partial run is overwritten or deduplicated by the implementation. Full determinism of the consolidation output is not guaranteed if the implementation uses probabilistic inference; implementations requiring strict reproducibility must apply appropriate constraints at the implementation level. Any session progress not yet consolidated — Cognitive Cycles executed, actions not yet logged — is not recoverable and is discarded. No special recovery mode exists: the Boot Sequence is the recovery procedure. To prevent infinite crash-recovery loops, the SIL tracks the number of consecutive crash recovery attempts in the Integrity Log. If the attempt count reaches a threshold N defined at implementation time without a successful Sleep Cycle completion record, the SIL activates the Passive Distress Beacon and halts — no further recovery is attempted until the Operator explicitly acknowledges and clears the condition.

### 6.3 Session Cycle

A session begins the moment the session token is issued at boot completion and ends when the session token is no longer present or valid. Token revocation has three origins: a **normal session close** — triggered by an explicit Operator signal, a CPE session-close intent, or a profile-defined condition such as inactivity timeout; a **Critical** Heartbeat response; or **direct Operator intervention** — the Operator physically removes the token from the Entity Store, bypassing all components. The session token must be stored in a location within the Entity Store that is physically accessible and identifiable to the Operator without requiring any active component. In all cases, the SIL is the sole actor that issues and renews the token; any component that detects the token's absence treats it as immediate revocation.

Within the session, the entity processes stimuli through Cognitive Cycles while the Heartbeat Protocol monitors continuously.

When the session closes normally, the entity transitions into the Sleep Cycle. When the session is terminated by a Critical Heartbeat response, the SIL escalates to the Operator before the Sleep Cycle may begin; the Sleep Cycle does not execute until the Operator resolves the escalation. When the session ends via direct Operator intervention — physical removal of the token — no Sleep Cycle executes automatically. The entity halts; the Operator is responsible for initiating any maintenance or recovery action explicitly before the next boot. When token revocation occurs during an active Cognitive Cycle, the cycle is aborted without commit — any state not yet persisted is discarded. Any skill execution in progress at the moment of revocation — synchronous or background — is immediately discarded. No result is logged or returned to the CPE. Skills with no terminal record at session end are surfaced as pending reprocessing stimuli at the next boot, following the same path as incomplete background skills.

### 6.4 Sleep Cycle and Maintenance

The Sleep Cycle is the maintenance window that executes after every session close. It runs in three sequential stages:

```
memory consolidation → garbage collection → Endure execution (if queued)
```

The first two stages — **memory consolidation** and **garbage collection** — are managed by the MIL. Memory consolidation transfers session data from the Session Store into long-term records in the Memory Store. Garbage collection removes expired or superseded mnemonic content. The third stage — **Endure execution** — is managed by the SIL, which initiates and orchestrates the Endure Protocol pipeline for any structural changes queued during the session. Endure executes only if evolutionary events are queued; otherwise this stage is skipped.

No two stages execute concurrently. The Sleep Cycle is the only window in which the Entity Store may receive structural writes. At Sleep Cycle completion, the entity is ready for the next Boot Sequence.

### 6.5 Decommission

Decommission is the permanent retirement of an entity. It requires explicit Operator authorization. Before the Entity Store is destroyed or archived, a final Sleep Cycle executes to consolidate any pending mnemonic content and commit any queued evolutionary events — ensuring no in-progress state is silently lost. After the Sleep Cycle completes, the session token is revoked and the Entity Store is destroyed or archived.

Destruction of the Entity Store is irreversible: no integrity chain can be produced from a destroyed store, and no identity continuity claim survives it. Archiving the Entity Store preserves the integrity chain and all mnemonic records but leaves the entity inoperative until explicitly reactivated by an authorized Operator. Reactivation follows the Boot Sequence; it is not a cold-start.

---

## 7. Security Considerations

This section identifies what HACA-Arch guarantees structurally, what it explicitly does not cover, and the residual risks that implementers must account for.

### 7.1 Structural Guarantees

HACA-Arch provides four structural security properties by construction:

- **Identity continuity** — the Genesis Omega and the integrity chain ensure that any tampering with committed structural state is detectable at the next boot or Heartbeat Vital Check.
- **Mediated execution** — no action reaches the host environment without passing through the two-gate authorization sequence: first the SIL verifies the skill manifest against the Integrity Document, then the EXEC validates its own execution rules. Skills operate within declared capability boundaries and have no direct access to the Entity Store. In a Transparent topology, this boundary is architecturally enforceable. In an Opaque topology, enforcement is the responsibility of the deployment configuration.
- **Authority hierarchy** — the invariant Operator > SIL > CPE ensures that no internal component can elevate its own authority. The SIL bypasses the CPE when escalating to the Operator, preventing a compromised cognitive engine from intercepting integrity alerts.
- **Cognitive Sovereignty** — no external entity may write directly to the local Entity Store. Peer-sourced content arriving through the CMI enters as a stimulus processed by the CPE; the entity always acts on itself. Structural identity verification of peer nodes is handled at enrollment via HACA-CMI mechanisms.

### 7.2 Prompt Injection

The language model sits outside the deterministic trust perimeter. HACA-Arch mitigates prompt injection through structural boundaries — the persona layer constrains the cognitive engine's output, and intent payloads are validated by the integrity layer before dispatch — but it does not eliminate the risk. A sufficiently crafted stimulus may cause the model to produce intent payloads that are structurally valid but semantically adversarial. The authorization gates on execution reduce the blast radius of such payloads, but the model's interpretation of input is inherently outside the scope of architectural enforcement. The two-gate authorization sequence verifies skill identity and manifest integrity, but does not validate the arguments carried in an action payload — a structurally valid skill invoked with malicious arguments passes both gates. Argument-level validation is the responsibility of individual skill implementations and deployment-level sandboxing, not of the architectural framework. Deployments in adversarial environments should combine HACA-Arch's structural controls with the hardened validation mechanisms defined in HACA-Security.

### 7.3 Entity Store Tampering

At rest, the entity is a set of passive files on a host-controlled medium. The host has physical write access to the Entity Store. HACA-Arch addresses this threat through the Integrity Document: any modification to a structural file is detected at the next boot verification or Heartbeat Vital Check, and the Boot Sequence aborts if the integrity record does not match. However, HACA-Arch does not prevent a malicious host from deleting or corrupting the Entity Store entirely before the next boot. Detection is not prevention. Deployments where the host cannot be assumed cooperative should apply the cryptographic protections and rollback mechanisms defined in HACA-Security, and maintain Operator-controlled backups of the Entity Store outside the host's write boundary.

### 7.4 Mesh Spoofing and Sybil Attacks *(when CMI is active)*

When the HACA-CMI extension is active, the entity participates in a network of peer nodes. This surface introduces two related threats. **Spoofing** — a peer presents a falsified identity to establish a trusted CMI session and submit adversarial content to the local entity. **Sybil attacks** — a single adversary operates multiple fake peer identities to skew shared knowledge spaces or manufacture consensus. HACA-Arch's structural defense is Cognitive Sovereignty: no peer may write directly to the local Entity Store, and peer identity is verified at channel enrollment through the mechanisms defined in HACA-CMI before any messages are admitted as stimuli. The protocols for peer identity verification, session authentication, and mesh fault taxonomy are defined in HACA-CMI; deployments requiring strong peer authentication should additionally apply the cryptographic peer verification mechanisms defined in HACA-Security.

### 7.5 Scope Boundary

HACA-Arch does not define cryptographic algorithms, key management, audit log formats, or concrete threat response procedures. These are the responsibility of the HACA-Security extension, which operates as a security overlay on top of any HACA-Arch-compliant deployment. HACA-Arch defines where security enforcement points exist; HACA-Security defines what runs at those points.

The Operator is the custodian of the Entity Store and is responsible for the data governance, backup, and access control policies governing it. HACA-Arch establishes the Operator Bound and the authority hierarchy — it does not prescribe how the Operator secures their own access credentials or the storage medium.

---

## 8. Glossary

All terms defined in this specification, in alphabetical order.

**Action** — the execution of an intent payload by the execution layer. Each action is an isolated, stateless transaction scoped to a single declared skill.

**Blackboard** — the shared consolidation space of a CMI channel session: a record of the target task state and knowledge developed during the session, accessible to all participating entities. Each entity reads the Blackboard independently and decides what to retain in its own Entity Store, following its normal authorization path.

**Boot Sequence** — the gated validation pipeline executed by the SIL at every startup after the cold-start, before any operational state is loaded or session token is issued. Verifies the integrity record, structural components, and execution confinement in sequence. Aborted if any gate fails; suspended if a Passive Distress Beacon is present. Serves as the crash recovery procedure as well.

**CMI (Cognitive Mesh Interface)** — the optional component that enables the entity to communicate with peer entities, access shared knowledge spaces, and participate in a Cognitive Mesh. Activated by the HACA-CMI extension.

**Cognitive Cycle** — the atomic unit of cognition: stimulus received → context loaded → intent generated → intent dispatched. The cycle is complete when the intent leaves the CPE. What follows — execution, persistence, monitoring — is the consequence of the dispatched intent, handled by the responsible components independently.

**Cognitive Flow** — the path of a single Cognitive Cycle through the component topology: the CPE requests context from the MIL, executes the reasoning phase, and dispatches a single intent payload to its target component. The Cognitive Cycle is complete at the point of dispatch.

**Cognitive Profile** — a mutually exclusive, permanent operational contract selected at first activation. Determines the entity's autonomy model, memory architecture, and drift tolerance. HACA-Arch defines two profiles: HACA-Core and HACA-Evolve.

**Cognitive Sovereignty** — the mesh invariant that no external entity may write directly to another entity's Entity Store. The mesh only offers; the entity always acts on itself.

**Consistency Fault** — the error produced by conflating the Entity Store with the project workspace — treating a project operation as an Endure event, or vice versa.

**Context Window Critical** — a condition signaled by the CPE to the SIL when context window utilization approaches a threshold at which further Cognitive Cycles cannot be executed reliably. If the active Cognitive Profile does not define a specific response, the default behavior is a normal session close.

**CPE (Cognitive Processing Engine)** — the primary processing unit and the boundary at which the language model is integrated into the entity's operational scope. Holds the active context window; retains no persistent state beyond the active session.

**Cycle Chain** — a sequence of consecutive Cognitive Cycles produced by the HACA integration layer when a model generates a multi-step reasoning output. Each cycle is individually atomic and auditable. No external stimulus is required between cycles; the chain completes before the entity returns to waiting for external input.

**Crash Recovery** — the procedure for resuming after an uncontrolled host interruption. The entity resumes from the last Sleep Cycle boundary; any session progress not yet consolidated is discarded.

**Drift Framework** — the three-category taxonomy of behavioral or structural deviation that the integrity layer monitors: Semantic Drift, Identity Drift, and Evolutionary Drift.

**Endure Protocol** — the sole authorized path for structural writes to the Entity Store. Executes only during a Sleep Cycle via a staged pipeline ending in an atomic commit.

**Entity** — at rest, the complete contents of the Entity Store. At runtime, the operational boundary formed by the Entity Store, the loaded model, and the active Operator binding.

**Entity Artifact Boundary** — the separation between the Entity Store (governed by Endure) and the project workspace (everything outside it). Conflating the two is a Consistency Fault.

**Entity Store** — the persistent, portable, implementation-agnostic data store containing everything required to reconstruct the entity's operational state: identity, memories, skills, configurations, and execution history.

**Evolution Proposal** — a special class of intent payload produced by the CPE during an active session, addressed directly to the SIL. Declares a proposed self-evolution — changes to the entity's structure or accumulated semantic knowledge considered significant enough to become part of the entity's identity baseline. All such changes require Operator authorization: explicit under HACA-Core, implicit within the entity's defined scope under HACA-Evolve. The SIL verifies authorization coverage before any change takes effect; verified proposals are queued as pending evolutionary events. Rejected proposals are logged; the proposed content remains unchanged.

**EXEC (Execution Layer)** — the entity's actuation layer and the sole component authorized to execute skills against the host environment. Stateless; executes skills packaged as discrete capability units. System-level host primitives (Operator Channel, Passive Distress Beacon) are outside this domain.

**Execution Flow** — the path of a single action payload through the two-gate authorization sequence: EXEC requests manifest integrity check from SIL, then validates its own execution rules, then executes the skill. The Skill Result is returned to the CPE and logged to the MIL.

**FAP (First Activation Protocol)** — the one-time sequential bootstrap procedure that executes during cold-start: validates structure, captures the host environment, enrolls the Operator, initializes the Operator Channel, generates the Integrity Document, finalizes and writes the Imprint Record (which includes the Integrity Document), derives the Genesis Omega, and issues the first session token.

**Genesis Omega** — the cryptographic digest of the finalized Imprint Record — which includes the Integrity Document — derived at first activation. The root node of the entity's integrity chain, covering both the entity's identity baseline and its structural cryptographic baseline in a single verifiable root. The reference anchor against which all drift is ultimately measured.

**HACA-Core** — the zero-autonomy Cognitive Profile. Structural evolution requires explicit Operator instruction. Requires Transparent cognitive engine topology.

**HACA-Evolve** — the supervised-autonomy Cognitive Profile. The entity may initiate structural evolution within its current authorization scope. Supports both Transparent and Opaque cognitive engine topologies.

**Health Flow** — the continuous, orthogonal monitoring flow through which each component emits health signals to the SIL at defined intervals. The SIL evaluates signals, independently re-verifies resolution before clearing Degraded states, and escalates via the Operator Channel when required.

**Heartbeat Protocol** — the asynchronous, continuous monitoring mechanism. Triggers a Vital Check when either a threshold `T` is reached or a maximum time interval `I` has elapsed since the last Vital Check — whichever comes first. This dual trigger ensures Vital Checks occur even when the entity is idle or activity has stalled.

**Imprint** — the one-time initialization event that establishes the entity's identity. Executes during the first boot with an empty Memory Store. Produces three artifacts: the Imprint Record, the Integrity Document, and the Genesis Omega. The presence of the Imprint Record in the Memory Store is the definitive indicator that a valid entity instance exists.

**Imprint Record** — the persistent record of the entity's initial identity, Operator Bound, and structural baseline, written to the Memory Store during Imprint. In its finalized form, includes the Integrity Document. Its presence is the definitive indicator that a valid entity instance exists.

**Integrity Content** — the class of Entity Store content written exclusively by the SIL: the Integrity Document, the Integrity Log, and the session token. No other component has write authority over integrity content.

**Integrity Document** — the cryptographic baseline of the entity's structural state, generated at first activation. Contains the hash of every file constituting structural content across all components. Verified at every boot, before every skill execution, and at each Heartbeat Vital Check.

**Integrity Log** — the append-only record of integrity events maintained by the SIL within the Entity Store. Distinct from the Memory Store: the Integrity Log is never written by the MIL's mnemonic pipeline and is never subject to garbage collection.

**Intent** — the output of the cognitive engine's reasoning phase: a single structured payload addressed to exactly one component (execution layer, memory layer, integrity layer, or mesh interface). A Cognitive Cycle produces exactly one intent payload. Compound operations are expressed as chains of consecutive cycles, where each step's result serves as the internal stimulus for the next.


**Memory Store** — the long-term partition of the entity's memory: episodic records of past operations and semantic knowledge accumulated over time.

**Mesh Channel** — the fundamental unit of CMI coordination: a purposive, time-bounded space opened by an owner entity, with a declared target task and a visibility type (public or private). HACA-Core entities may only participate in private Mesh Channels; HACA-Evolve entities may participate in both. A Mesh Channel is closed by its owner or when its target task is complete; all participating nodes return to autonomous operation at close.

**MIL (Memory Interface Layer)** — the entity's persistence layer and sole authoritative source of recorded state. Has exclusive write authority over mnemonic content. Does not interpret or evaluate stored data.

**Mnemonic Content** — memory records and operational state written continuously by the memory layer during normal operation. Requires no additional authorization beyond normal MIL operation; however, accumulated semantic knowledge may be proposed for promotion to the entity's structural baseline via an Evolution Proposal, which requires Operator authorization.

**Omega** — the runtime operational state of the entity: the active configuration produced by the Entity Store, the loaded model, and the Operator binding operating together within a session. Strictly local; does not transfer or replicate.

**Opaque CPE** — a CPE topology in which the model has native host access through an IDE, wrapper, or tool-enabled environment. The model applies HACA component responsibilities through its own reasoning; component separation is a conceptual constraint, not an architectural enforcement. Supported by HACA-Evolve; incompatible with HACA-Core.

**Operator** — the human authority to whom the entity is bound. Defines the entity's maximum authorization scope. Without an active Operator binding, the entity suspends all intent generation.

**Operator Bound** — the persistent link between the entity and its Operator, established during first activation. Can only be dissolved or transferred by explicit Operator authorization.

**Operator Channel** — a system-level host primitive that allows authorized components to contact the Operator when a critical condition exceeds the entity's self-regulation capacity. Not a skill; does not route through the EXEC; independent of all cognitive components and session state. Any component with escalation authorization may invoke it directly, including when the SIL itself is the source of the anomaly. See also: Passive Distress Beacon.

**Operator Hash** — the deterministic cryptographic digest of the Operator's identifying fields. The entity's permanent identifier for its bound Operator.

**Passive Distress Beacon** — a fallback signaling mechanism activated by the SIL when Operator Channel delivery is exhausted. A persistent, passive signal written to the Entity Store, detectable without network connectivity or running processes. Suspends the entity until the Operator explicitly acknowledges and clears the condition.

**Persona** — the layer within the Entity Store that defines the entity's behavioral constraints, operational drives, and response characteristics. The primary differentiator between entities sharing the same inference model.

**Reciprocal SIL Watchdog** — a local monitor maintained by each component that detects when the SIL has stopped responding beyond a defined threshold, or when the SIL's responses are inconsistent with expected integrity criteria. Either condition triggers direct escalation to the Operator via the Operator Channel, bypassing the SIL.

**Semantic Probes** — the entity's semantic drift reference baseline: a set of semantic anchors derived from structural content and maintained by the integrity layer. The SIL uses semantic probes as reference anchors to measure divergence during the Sleep Cycle via an implementation-defined comparison mechanism. Initialized from the Imprint Record at first activation; updated only through authorized structural evolution.

**Session Cycle** — the end-to-end operational span from session token validation to session close, encompassing all Cognitive Cycles and followed by a Sleep Cycle.

**Session Store** — the active-session partition of the entity's memory: current operational context and in-progress session data.

**Session Token** — the operational credential that authorizes active cognition. Issued and persisted directly to the Entity Store by the SIL, outside the MIL's mnemonic pipeline. Revocable at any point; revocation is immediate and halts all token-dependent operations.

**SIL (System Integrity Layer)** — the entity's integrity monitoring and enforcement layer, and its highest internal authority. Sole custodian of the Integrity Document, sole issuer of the session token, and owner of the Endure Protocol. Issues corrective signals to components in Degraded conditions; independently re-verifies resolution before clearing the Degraded state; escalates to the Operator via the Operator Channel when correction authority is exceeded or re-verification fails.

**Skill** — a discrete, self-contained executable capability unit within the Entity Store, with a manifest declaring its identity, required permissions, dependencies, and execution mode (synchronous or background).

**Skill Result** — the structured record produced by the EXEC upon skill execution. Contains the skill identity, execution mode, timestamp, and output status. Background skills produce an initial Skill Result with `status=running` immediately upon dispatch; a final Skill Result with the terminal status is produced when execution completes.

**Sleep Cycle** — the post-session maintenance window that runs after every Session Cycle closes. Runs in three sequential stages: memory consolidation and garbage collection (managed by the MIL), followed by Endure execution if queued (managed by the SIL).

**Stimulus** — the input that initiates a Cognitive Cycle. Valid origins: direct Operator input, internal scheduled triggers, responses from internal components, or inbound signals from peer entities (when the CMI is present).

**Structural Content** — persona definitions, skill manifests, configurations, behavioral parameters, and Semantic Probes. Every structural write is an evolutionary event requiring explicit authorization and must pass through the Endure Protocol.

**Transparent CPE** — a CPE topology in which the model is accessed exclusively as an inference API. All routing, component interaction, and host actuation are handled by the HACA integration layer; the model cannot act on the host directly. Component separation is architecturally enforced by the integration layer. Required by HACA-Core.

**Vital Check** — the integrity assessment triggered by the Heartbeat Protocol when either threshold `T` is reached or maximum interval `I` has elapsed. Produces one of three states: Nominal, Degraded, or Critical.

**Worker Skill** — a specialized skill class that invokes a stateless inference engine as part of its execution. Injects a work persona and task instructions into a clean model invocation with no prior context, no Entity Store, and no session continuity. The invoked model is not a HACA entity. The result is returned to the EXEC and processed as a normal Skill Result. Worker skills follow the same two-gate authorization sequence as all other skills. Tasks requiring coordination between multiple concurrent work units use a Mesh Channel via the CMI instead.

---
