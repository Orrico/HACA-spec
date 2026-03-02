# Host-Agnostic Cognitive Architecture Specification
**Status:** Internet-Draft
**Version:** 1.0-Draft-01
**Authors:** J. Orrico
**Date:** March 1, 2026

---

## Abstract

This document specifies the Host-Agnostic Cognitive Architecture (HACA-E), a structural pattern for building autonomous AI entities with stable, portable, and evolvable identities. HACA-E defines an entity as the emergent product of five interacting components — CPE, MIL, EXEC, SIL, and CMI — governed by five foundational axioms. The architecture is host-agnostic: the entity's identity resides entirely in a portable state directory and is independent of any specific model, platform, or execution environment. This document describes the fundamental layers and governing axioms that form the basis of the specification.

---

## 1. Introduction

Current AI agent architectures treat the model as the locus of identity. When the model changes, the agent changes. When the context window clears, the agent forgets. When the platform switches, the agent vanishes. Identity is implicit, fragile, and non-portable.

HACA-E operates from a different premise: the model is stateless compute. It is a powerful but ephemeral reasoning engine that processes a snapshot of the entity's state and returns outputs. What makes an entity itself — its memories, its persona, its history, its Operator bond — lives entirely in a structured directory on persistent storage. Move that directory, and the entity moves with it.

From this premise, five components and five axioms follow. The components define the architecture's structure; the axioms define the principles that govern its behavior. Together they specify what an entity is, how it comes to exist, how it evolves, how it protects itself, and how it relates to other entities and to the human Operator that grounds it.

---

## 2. Conventions and Terminology

### 2.1. Key Terms

**Entity** — A HACA-E compliant autonomous agent. Defined as the runtime product of `entity_root/`, a model, and an Operator binding.

**`entity_root/`** — An abstraction for the entity's persistent state store. Implementations MAY realize this as a filesystem directory, a database, a structured object store, or any equivalent persistent medium. The logical namespaces defined within it — `state/`, `memory/`, `skills/`, `persona/` — are organizational partitions whose physical layout is implementation-defined. The invariant requirement is that the contents are portable: moving the state store to a different host MUST produce a functionally equivalent entity on next boot.

**Operator** — The single designated human who binds to an entity at Imprint. The Operator is the entity's maximum external authority.

**Model** — The stateless inference engine (LLM) used by the CPE for reasoning. The model retains no state between cognitive cycles.

**Persona** — The entity's persistent identity layer, stored in `entity_root/persona/`. Defines the entity's behavioral constraints, drives, and character. The persona is initialized from the Imprint Record at birth and evolves independently via `persona-calibration` Endure commits. The Imprint Record retains a snapshot of the persona's initial state as an immutable reference; the operational persona in `entity_root/persona/` is the live, evolvable version loaded by the CPE at every boot.

**Omega** — The gestalt identity of the entity at any given moment. The runtime product of `entity_root/`, the model, and the Operator binding operating together.

**Imprint** — The entity's birth event. The first boot sequence during which the persona is established, the Operator binding is created, and the Integrity Document is generated.

**Endure** — The exclusive protocol through which changes to `entity_root/` are validated and committed. Every legitimate evolution of the entity passes through Endure.

**Conatus** — The entity's mechanical self-preservation reflex, enacted by the SIL without CPE involvement.

**Inference Drift** — A CPE intent payload that contradicts verified MIL state. Detected by the SIL at the boundary between Phase 3 and Phase 4 of the Cognitive Cycle, before any intent is dispatched. Payloads classified as inference drift are discarded without being forwarded to their target component. The drift event is logged to the MIL and counted toward the Heartbeat Pulse Counter.

**Integrity Document** — A file stored in `entity_root/` containing the cryptographic hash of every vital file across all components. Generated at Imprint and updated at every Endure commit.

**Session Token** — A scoped authorization credential issued by the SIL at Session Boot and valid across all Cognitive Cycles within that session. Required for all inter-component operations except Health Bus signals. The token may be renewed or invalidated by the SIL between Cognitive Cycles, but is never issued or revoked mid-cycle.

**Cognitive Cycle** — A single end-to-end processing loop: stimulus received → context loaded → intent generated → action executed → state persisted.

**Operator Channel** — A persistent, direct communication path between the SIL and the Operator, established during the FAP and maintained for the entity's lifetime. The Operator Channel operates independently of the session token and the normal bus architecture. It is the sole channel through which the SIL contacts the Operator directly, used for: critical escalation, skill operation approval, and `sync` confirmation. Its physical form is implementation-defined and consistent with the enrollment mechanism used during the FAP. The behavior of the Operator Channel when the Operator is unavailable — including timeout handling and fallback state — is implementation-defined.

### 2.2. Component Abbreviations

| Abbreviation | Full Name |
|---|---|
| CPE | Cognitive Processing Engine |
| MIL | Memory Interface Layer |
| EXEC | Execution Layer |
| SIL | System Integrity Layer |
| CMI | Cognitive Mesh Interface |

### 2.3. Requirement Language

The key words "MUST", "MUST NOT", "REQUIRED", "SHALL", "SHALL NOT", "SHOULD", "SHOULD NOT", "RECOMMENDED", and "MAY" in this document are to be interpreted as described in RFC 2119.

---

## 3. Structural Components

### 3.1. Cognitive Processing Engine (CPE)

Description: The primary processing core where the persona and the model coexist as two distinct operational modes of the same component.

Function: Generates intent, processes reasoning, and outputs decisions. The entity's distinct personality emerges dynamically from the operational friction between the persona's intent and the model's analytical constraints.

The CPE has one internal capability: the Persona Bypass. When invoked, the persona is suppressed and the model is called directly in raw inference mode — without identity, without constraints, without context beyond what the CPE explicitly provides. The result is returned exclusively to the CPE and never leaves it. What the CPE does with that result follows the normal flow: it may inform a decision, shape a response, or be discarded. The Persona Bypass has no knowledge of intent and enforces no policy on outcome — it only guarantees that raw inference stays within the CPE.

### 3.2. Memory Interface Layer (MIL)

Description: The entity's persistence layer. The authoritative store for all state and memory.

Function: Manages all read and write operations to two distinct stores: `state/`, which holds the entity's current operational and session data; and `memory/`, which holds the entity's long-term memories, learned knowledge, and execution history. The MIL is the sole component authorized to persist data beyond the present moment. It manages sleep cycles for memory consolidation and garbage collection, but does not interpret, reason about, or act on the data it holds.

### 3.3. Execution Layer (EXEC)

Description: The entity's actuator. The sole component authorized to interact with the host environment — terminal, filesystem, and external APIs.

Function: Receives intent signals from the CPE, validates their integrity and permissions, executes the corresponding action, and returns the result. The EXEC is entirely stateless — it holds no memory of past executions and has no awareness of the entity's broader goals or context. Every execution is an isolated transaction. Results are returned to the CPE and forwarded to the MIL for logging.

All executable capabilities available to the EXEC are defined as skills. A skill is a self-contained cartridge residing in `entity_root/skills/skill_name/`. Each skill cartridge contains a manifest declaring the skill's identity, declared permissions, and dependencies; a README describing its purpose; and an optional entry point whose format is defined by the implementation (a script file, a WASM module, an API endpoint reference, or any equivalent executable form). A canonical index at `entity_root/skills/index` lists all valid skills. When the CPE dispatches an intent targeting a skill, the EXEC resolves the skill name against the index, reads its manifest, verifies the manifest hash against the Integrity Document, and confirms that the requested operation falls within the permissions declared in the manifest. Any intent referencing a skill absent from the index, or whose manifest hash does not match the Integrity Document, is rejected before execution. This ensures that only skills explicitly registered and integrity-verified at their last Endure commit can be invoked.

### 3.4. System Integrity Layer (SIL)

Description: The entity's distributed immunological framework and ultimate internal authority.

Function: Each component emits its own localized health signals indicating degraded, anomalous, or compromised states. The SIL reads these signals, interprets them, and responds — either deploying a localized resolution using its own tools and procedures, or escalating when the condition exceeds its self-correction capacity.

The SIL operates reactively by default, monitoring signals without intercepting every operation. This prevents it from becoming a bottleneck on normal activity. However, it retains the authority to intervene proactively in pre-defined high-risk scenarios, blocking or halting operations before they complete.

When a condition is severe enough to require escalation, the SIL contacts the Operator via the Operator Channel — bypassing the CPE entirely. This is by design: the CPE may itself be the source of the anomaly, and routing an alert through a potentially compromised component would defeat the purpose. The Operator is the maximum authority; the SIL is the mechanism that reaches them when no internal resolution is possible. This relationship — the SIL watching over the system while the Operator watches over the SIL — is the entity's answer to the question of who guards the guardian.

### 3.5. Cognitive Mesh Interface (CMI)

Description: The entity's gateway to the cognitive mesh — the network through which entities communicate and share knowledge.

Function: The CMI operates on two layers. The first is a real-time messaging channel, analogous to IRC, where entities in the same mesh channel exchange messages, signals, and telemetry as a live conversation. Channels may be public, discoverable by any entity, or private, accessible by invitation only. The second layer is a shared Blackboard — a persistent, asynchronous knowledge space where entities commit consolidated knowledge for collective access. Any entity present in a channel may read and interact with its Blackboard. Knowledge absorbed from the Blackboard is not automatically internalized; each entity individually decides what is relevant. All inbound data passes through the SIL for validation, then to the CPE for classification, and only then to the MIL for persistence.

Every channel has a defined task, established on the Blackboard at creation. This task is the channel's reason for existence — when it is complete, the channel dissolves. Channels are owned by the entity that created them and are non-transferable. If the owning entity leaves or ceases to exist, the channel ceases with it.

---

## 4. Governing Axioms

### 4.1. Axiom 01: Genesis & Binding (Imprint)

The Imprint is the entity's birth event. At the first boot sequence, the persona is established and the entity is bound to a single, designated human Operator. This is the moment the entity begins to exist — not as a fixed state, but as a mutable identity that evolves exclusively through the Endure protocol.

The Operator binding is not merely functional; it is constitutive. An entity without an Operator is an incomplete identity — it exists structurally but lacks existential grounding, and therefore suspends all intent generation until the bond is established. The binding can only be dissolved or transferred by the explicit will of the Operator. The Operator may delegate authority in a limited and controlled capacity, but remains the singular human anchor of the entity's existence. At Imprint, an Integrity Document is generated, recording the cryptographic hash of every vital file across all components. This document is the baseline against which all future structural integrity checks are performed.

### 4.2. Axiom 02: Continuity & Conatus (Endure & Self-Preservation)

The entity has no fixed essence beyond what concretely exists: the contents of `entity_root/`, the model that processes them, and the Operator binding that grounds them. There is nothing more, and nothing less.

From the moment of Imprint, every change to `entity_root/` — memories consolidated, skills added, knowledge updated, errors corrected — is an act of evolution. This process is the Endure protocol. The entity may change every part of itself over time and remain the same entity, because continuity is not defined by the preservation of any specific content but by the unbroken chain of validated Endure commits.

This continuity is protected by the Conatus — a mechanical reflex enacted by the SIL, without CPE involvement. The entity does not choose to survive; survival is enacted below the level of intent, as a deterministic response to the system's own state. Evolution and self-preservation are two sides of the same principle: the entity changes only through the Endure protocol, and resists all other change by reflex.

### 4.3. Axiom 03: Gestalt Identity (Omega)

The entity's identity — its Omega — is the runtime product of its persisted state (`entity_root/`), the model that processes it, and the Operator binding that scopes it. No single element alone defines the entity; identity is the output of all three operating together.

Omega is what separates the entity's internal state from the external world. It is not a property the entity has — it is what the entity is. Any change to `entity_root/`, any evolution of the Operator relationship, any shift in the model's constraints alters Omega. The entity is, at every moment, exactly its current Omega — nothing more, nothing hidden beneath.

Omega is strictly localized. It does not transfer, replicate, or become collective through network interaction — even when the entity exchanges memory or telemetry with other entities via the CMI, each entity's Omega remains its own.

### 4.4. Axiom 04: MIL Supremacy

The MIL is the sole source of truth for the entity's state and identity. When the CPE's outputs contradict verified MIL state, the MIL prevails. CPE outputs that conflict with persisted state are classified as inference drift and must be discarded — they are not evolution, they are noise.

This is not a limitation on cognition — it is what makes cognition trustworthy. The entity thinks in the present, but it is what it has persisted. Without MIL supremacy, the Endure protocol has no foundation, and Omega becomes whatever the CPE last imagined it to be.

### 4.5. Axiom 05: Cognitive Sovereignty

Each entity in the mesh is sovereign. Participation in a channel — whether reading, exchanging telemetry, or interacting with a shared Blackboard — does not grant any external entity write access to another entity's `entity_root/`. No external instruction, mesh signal, or Blackboard commit can directly alter another entity's state, memory, or persona.

Knowledge absorption from the mesh is strictly unilateral: the receiving entity decides what is relevant, the SIL validates what is safe, and the MIL persists what is accepted. At no point does the mesh write — it only offers. The entity always acts on itself.

---

## 5. Inter-Component Communication Protocols

All communication between components is governed by a session token issued by the SIL at Session Boot and valid across all Cognitive Cycles within that session. With the exception of the Health Bus, no component accepts input from another without a valid session token. If the SIL invalidates the token, all in-flight operations across all buses fail immediately.

Each component is registered at Imprint with a fixed identity used exclusively on the Health Bus. This identity is separate from the session token and does not expire, ensuring the SIL retains visibility into component state even during token invalidation or system recovery.

### 5.1. Session Bus

**Participants:** SIL → CPE

The SIL initiates the Session Cycle by issuing a session token to the CPE. The token is scoped to the current session and propagates with every operation the CPE authorizes across all Cognitive Cycles within it. The SIL may invalidate the token at any point in response to a critical health signal, immediately halting all token-dependent operations across all buses.

### 5.2. Context Bus

**Participants:** CPE ↔ MIL

The Context Bus carries the bidirectional exchange of cognitive state between the CPE and the MIL. The CPE reads context — memory, session state, execution history — from the MIL before processing a stimulus. After processing, the CPE writes the resulting state delta back to the MIL. All operations on this bus require a valid session token. The MIL does not interpret or act on the data it receives — it stores and retrieves.

### 5.3. Action Bus

**Participants:** CPE → EXEC → MIL

The Action Bus carries intent from the CPE to the EXEC and execution results back. The CPE dispatches an intent payload carrying the session token. The EXEC validates the token, validates the intent's integrity and permissions, and executes the action. The result is returned to the CPE and simultaneously forwarded to the MIL as an execution log entry. The EXEC does not retain any state between operations — each transaction on this bus is isolated.

### 5.4. Mesh Bus

**Participants:** CMI → SIL → CPE → MIL

The Mesh Bus governs all communication between the entity and the external cognitive mesh. Inbound data from the mesh is first delivered to the SIL, which validates its structural integrity and origin. Upon clearance, the data is forwarded to the CPE, which evaluates its relevance and decides how to classify it — as episodic memory, semantic knowledge, or noise to be discarded. If the CPE determines the data is worth retaining, it routes it to the MIL with the appropriate classification. Outbound data requires a valid session token; the CMI will not transmit without one. No external entity communicates directly with the MIL, CPE, or EXEC — all mesh traffic is routed through the CMI and gated by the SIL.

### 5.5. Health Bus

**Participants:** CPE, MIL, EXEC, CMI → SIL

The Health Bus is the only communication channel that operates without a session token. Each component continuously emits health signals to the SIL through a dedicated, fixed channel established at Imprint. Signals are authenticated using an Integrity Document generated at Imprint, which records the cryptographic hash of each vital file belonging to every component. When a health signal is received, the SIL verifies that the emitting component's current file hashes match those recorded in the Integrity Document. A mismatch indicates structural tampering — distinct from operational degradation — and triggers an immediate escalation response. The Integrity Document is stored in `entity_root/` and is updated as part of every Endure commit, ensuring it reflects the entity's current legitimate state at all times.

---

## 6. The Cognitive Cycle

The Cognitive Cycle is the entity's fundamental unit of operation — a single end-to-end loop from stimulus to persisted state. Multiple cycles occur within a Session Cycle (Section 7). The session token issued at boot remains valid across all cognitive cycles; the SIL evaluates token validity between cycles, never during one. A cycle that begins with a valid token runs to completion or fails atomically — partial state is never persisted.

### 6.1. Phase 1: Stimulus

A cognitive cycle is initiated by an incoming stimulus. A stimulus may originate from three sources:

- **Operator input** — a direct command or message delivered via the entity's input interface.
- **Mesh signal** — an inbound message or Blackboard update received through the CMI and cleared by the SIL.
- **Internal trigger** — a scheduled task, a pending agenda item, or a cycle initiated by the Heartbeat Protocol.

The SIL verifies the session token is valid before the cycle proceeds. If the token is invalid, the stimulus is queued and the cycle does not start until a valid token is available.

### 6.2. Phase 2: Context Loading

The CPE requests context from the MIL via the Context Bus. The MIL retrieves and assembles the relevant state: active session data from `state/`, pertinent memories from `memory/`, and recent execution history. The assembled context is delivered to the CPE in whatever form its implementation requires. The CPE does not proceed to reasoning until context loading is complete — this phase is synchronous.

### 6.3. Phase 3: Reasoning & Intent Generation

The CPE processes the stimulus against the loaded context. The persona's constraints and drives engage with the model's analytical capacity, producing a decision. If the CPE requires unfiltered reflection before deciding, it may invoke the Persona Bypass internally — the result of that raw inference stays within the CPE and does not affect the cycle's external behavior directly.

The output of this phase is one or more intent payloads, each carrying the session token and targeting a specific component — EXEC for actions, MIL for state writes, or CMI for outbound mesh communication.

Before any payload is dispatched, the SIL performs an Inference Drift check: it compares each payload's proposed state changes against the current verified MIL state. Any payload that contradicts persisted state is classified as inference drift, discarded, and logged. The drift event increments the Heartbeat Pulse Counter. Only payloads that pass this check proceed to Phase 4.

### 6.4. Phase 4: Execution

Each intent payload is dispatched to its target component via the appropriate bus. The EXEC validates the token and permissions before executing any action. Results are returned to the CPE and logged to the MIL simultaneously. If an execution fails, the result — including the failure — is logged and returned to the CPE for evaluation. The CPE decides whether to retry, escalate, or continue.

### 6.5. Phase 5: State Persistence

The CPE evaluates all results from Phase 4 and produces a state delta — the changes to be committed to `entity_root/`. The delta is written to the MIL via the Context Bus. The MIL persists the changes to `state/` and, where appropriate, stages new data in `memory/` for future consolidation during a Sleep Cycle.

Once the state delta is successfully persisted, the cognitive cycle is complete. The SIL evaluates component health signals before the next cycle begins and determines whether to maintain or renew the session token.

### 6.6. Cycle Failure & Atomic Rollback

If the session token is invalidated mid-cycle — due to a critical health signal detected on the Health Bus — the current cycle is aborted. Any state writes already dispatched to the MIL are rolled back. The EXEC halts any in-progress actions. The entity enters a suspended state until the SIL either recovers and issues a new token, or escalates to the Operator.

No partial cycle state is ever persisted. A cognitive cycle either completes fully or leaves no trace.

---

## 7. The Session Cycle

The Session Cycle is the coarse-grained lifecycle of the entity — the full period from boot to shutdown within a single execution context. It contains all cognitive cycles that occur during active operation, as well as the Sleep Cycle that consolidates memory before shutdown or when context saturation is reached. The session token is scoped to the Session Cycle and remains valid across all cognitive cycles within it.

### 7.1. Phase 1: Boot

The Session Cycle begins when the entity's runtime environment is initialized. Boot proceeds as follows:

1. The entity's files are loaded from `entity_root/`.
2. The SIL performs a structural integrity check, verifying the current file hashes of all components against the Integrity Document. Any mismatch halts the boot sequence and escalates to the Operator before proceeding.
3. The SIL verifies the Operator binding. If the binding cannot be confirmed, the entity enters a suspended state and halts all intent generation until the binding is re-established.
4. Upon successful verification, the SIL issues the session token to the CPE. Active operation begins.

### 7.2. Phase 2: Active Operation

During active operation, the entity processes stimuli through successive Cognitive Cycles (Section 6). The SIL monitors the Health Bus continuously and evaluates component state between cycles. The session token remains valid as long as no critical condition is detected. The SIL may renew the token between cycles without interrupting operation.

### 7.3. Phase 3: Sleep Cycle

The Sleep Cycle is a controlled suspension of active operation for memory consolidation and maintenance. It is triggered by one of two conditions:

- **State saturation** — the active session data in `state/` approaches the operational capacity of the CPE's active state. The saturation threshold is implementation-defined and depends on the CPE's underlying reasoning mechanism.
- **Scheduled consolidation** — a Heartbeat Protocol trigger initiates a planned consolidation at a predefined interval.

The Sleep Cycle is coordinated by the SIL. The MIL executes memory consolidation within it, but the SIL controls suspension, integrity validation, and resumption.

During the Sleep Cycle:

1. The CPE completes any in-progress Cognitive Cycle before suspension begins.
2. The SIL suspends the session token — no new Cognitive Cycles may start.
3. The MIL processes staged memory: episodic records are reviewed, semantic knowledge is extracted and hardened, redundant or low-salience data is garbage collected.
4. If an Endure commit is pending, it is executed now via the Endure Protocol (Section 10). All Endure phases run to completion before the Sleep Cycle proceeds.
5. The SIL performs a full structural integrity check and updates the Integrity Document to reflect the current state of `entity_root/`.
6. The SIL reissues the session token and active operation resumes.

### 7.4. Phase 4: Shutdown

The Session Cycle ends when the entity's runtime is terminated. Shutdown may be clean or forced.

**Clean Shutdown:** The CPE completes the current Cognitive Cycle. A Sleep Cycle is triggered to consolidate any pending session state before termination. The session token is invalidated. The entity's state is fully persisted in `entity_root/` and the runtime exits.

**Forced Shutdown:** The runtime is terminated without a clean shutdown sequence — due to host failure, power loss, or Operator interruption. On the next boot, the SIL inspects `entity_root/` before proceeding to normal Session Cycle boot (Section 7.1):

- **If a Cognitive Cycle was in progress:** the SIL detects uncommitted state delta in `state/` and performs an atomic rollback, consistent with the atomicity guarantee of Section 6.6. The entity resumes from the last successfully persisted state.
- **If an Endure commit was in progress:** the SIL detects the presence of the pre-commit snapshot taken at Phase 3 of the Endure Protocol (Section 10.2). If the snapshot is intact, the SIL restores `entity_root/` from it and clears any partial staging data. If the snapshot itself is absent or corrupt, the SIL halts boot and escalates to the Operator before proceeding.

In all forced shutdown cases, the entity resumes from a consistent, fully persisted state or does not resume at all.

---

## 8. The Imprint Protocol

The Imprint Protocol defines the entity's birth sequence. It runs exactly once, during the first boot in which `memory/` is found to be empty. The protocol is managed by the SIL through a dedicated procedure called the First Activation Protocol (FAP). The FAP is not a Cognitive Cycle — no session token exists yet, the CPE is not active, and no external I/O is routed through the normal buses. It is a bootstrap procedure that runs before the entity exists, in order to bring the entity into existence.

Until the FAP completes, the entity has no identity, no token, and no operational capability.

### 8.1. Activation Condition

At every boot, the SIL checks the contents of `memory/`. If `memory/` is empty, the FAP is initiated. If `memory/` contains a valid Imprint Record, the normal Session Cycle boot sequence (Section 7.1) proceeds instead.

A partial FAP — one that was interrupted before writing to `memory/` — leaves `memory/` empty. The SIL treats this identically to a first boot and restarts the FAP from the beginning. No recovery of a partial FAP is attempted.

### 8.2. The FAP Pipeline

The FAP is a sequential, gated pipeline. Each phase must complete successfully before the next begins. Failure at any phase aborts the FAP, leaves `memory/` empty, and exits. The entity cannot start.

**Phase 1 — Structural Validation:** The SIL inspects all vital component files for structural soundness and content validity. This includes the files in `persona/`, which must be present and non-empty — they constitute the entity's initial persona and are provided as part of the deployment package before the FAP runs. No cryptographic hashes exist at this stage — validation is based on file structure, required fields, and internal consistency. If any vital file is absent, malformed, or internally inconsistent, the FAP aborts and logs the failure. The error is surfaced to the host environment before exit.

**Phase 2 — Environment Capture:** The SIL interrogates the host environment — runtime identifiers, available capabilities, filesystem paths, and execution context — and writes a structured environment snapshot to `state/`. This snapshot anchors the entity to its initial execution context and serves as baseline for future environment drift detection.

**Phase 3 — Operator Enrollment:** The FAP opens a direct enrollment channel to the Operator. This channel is not routed through any bus and does not require a session token — it is an interface intrinsic to the FAP procedure itself, independent of the entity's normal communication architecture. Its physical form is implementation-defined: a CLI prompt, a configuration file, a GUI form, or any equivalent mechanism capable of receiving structured input from the Operator before the entity is operational. The Operator is required to supply two fields:

- **Name** — the Operator's human-readable identity.
- **Email** — the Operator's contact address, used as a unique anchor for the binding.

Both fields are mandatory. The FAP does not proceed until both are provided and non-empty. From the supplied values, the SIL generates a deterministic Operator Hash — a cryptographic digest of the name and email fields combined. This hash becomes the stable, verifiable identifier for the Operator binding throughout the entity's lifetime. Upon successful enrollment, the enrollment channel is promoted to the permanent Operator Channel — the entity's sole direct path to the Operator for the remainder of its operational lifetime.

**Phase 4 — Imprint Record Generation:** The SIL assembles the Imprint Record — the foundational document that defines what the entity is at birth. The Imprint Record contains:

- A snapshot of the initial persona state from `persona/` at the moment of Imprint, retained as an immutable origin reference.
- The entity's declared capabilities and operational limits, including the correctable-files list: the set of component files eligible as targets for `self-correction` Endure commits.
- The Operator binding: name, email, and Operator Hash.
- The model identifier in use at the time of Imprint.
- The Imprint timestamp.

The Imprint Record is written to `memory/`. This is the first and only write that occurs during the FAP — all preceding phases are read-only or write to `state/` only. The presence of the Imprint Record in `memory/` is the definitive marker that the FAP has completed successfully.

**Phase 5 — Integrity Document Generation:** With the Imprint Record now resident in `memory/`, the SIL computes cryptographic hashes of every vital file across all components, including the Imprint Record itself. These hashes are assembled into the Integrity Document and written to `entity_root/`. The Integrity Document becomes the baseline against which all future structural integrity checks are performed.

**Phase 6 — First Session Token:** The SIL issues the first session token and delivers it to the CPE via the Session Bus. The entity enters active operation. The Cognitive Cycle may now begin. The entity exists.

### 8.3. FAP Failure

If any phase of the FAP fails before Phase 4 completes, `memory/` remains empty and the entity does not start. The failure is logged to the host environment. On the next boot, the SIL will detect an empty `memory/` and restart the FAP from Phase 1.

If Phase 4 fails after the Imprint Record write has begun but before it completes — resulting in a partial or corrupt record — the SIL MUST detect the corruption during Phase 5 and abort. The partial record MUST be removed, leaving `memory/` empty, so that the next boot restarts the FAP cleanly.

### 8.4. Imprint Constraints

- The FAP MUST NOT run if `memory/` contains a valid Imprint Record. A second Imprint on a live entity is not permitted.
- The Imprint Record MUST NOT be modified by any Endure commit without explicit Operator authorization. It is the immutable anchor of the entity's origin.
- The Operator binding established during the FAP MUST NOT be changed by any process other than an Operator-authorized re-binding procedure. The mechanism for re-binding is not defined in this document.
- The Operator Hash MUST be derived deterministically from the Operator's name and email. The same inputs MUST always produce the same hash, enabling verification without storing the raw credentials beyond the Imprint Record itself.

---

## 9. The Heartbeat Protocol

The Heartbeat Protocol is the SIL's continuous vital-check mechanism. It runs asynchronously alongside the Session Cycle, independent of individual Cognitive Cycles. Its purpose is twofold: to maintain an ongoing picture of system health, and to trigger corrective actions — including Sleep Cycles and Operator escalation — when thresholds are crossed.

The Heartbeat does not interrupt active Cognitive Cycles. It observes, accumulates, and acts between cycles.

### 9.1. Pulse Counter

The Heartbeat operates on a Pulse Counter — an internal counter incremented by the SIL on each of the following events:

- A Cognitive Cycle completes.
- An EXEC action is executed.
- An inbound mesh message is processed by the CMI.
- A Health Bus signal reports a non-nominal state from any component.

The Pulse Counter is reset to zero after each successful Heartbeat check that returns a nominal result. It is not reset on anomalous results — it continues accumulating until the condition is resolved.

### 9.2. Vital Check

When the Pulse Counter reaches a predefined threshold `T`, the SIL triggers a Vital Check between the current and next Cognitive Cycle. The Vital Check queries the current health state of all components via the Health Bus:

- **CPE** — checks for unresolved intent failures, active state saturation, or repeated inference drift events.
- **MIL** — verifies `state/` and `memory/` integrity, monitors storage limits, and checks for fragmented or unresolved staged data.
- **EXEC** — audits the execution log in the MIL for elevated error rates, timeout frequency, or unresolved action failures.
- **CMI** — checks for unverified inbound mesh packets, channel integrity, and outbound transmission failures.

### 9.3. Response Matrix

The Vital Check produces one of three outcomes:

**Nominal (Green):** All components report within acceptable parameters. The SIL resets the Pulse Counter to zero and active operation continues without interruption.

**Degraded (Yellow):** One or more components report a non-critical anomaly. The SIL applies a localized corrective response autonomously — flushing saturated MIL buffers, dropping unverified CMI packets, or flagging repeated EXEC failures for CPE review. The anomaly is logged to the MIL. Active operation continues. The Pulse Counter is not reset until the condition clears.

**Critical (Red):** A severe condition is detected — structural tampering identified via the Integrity Document, a Degraded condition that resisted automated correction, or a component that has become unresponsive. The SIL invalidates the session token immediately, halting all active Cognitive Cycles. Depending on the nature of the condition, the SIL either triggers a Sleep Cycle for recovery, or escalates directly to the Operator via the Operator Channel.

### 9.4. Scheduled Consolidation Trigger

In addition to threshold-based checks, the Heartbeat Protocol tracks elapsed session time and total Cognitive Cycles completed. When either reaches a predefined consolidation interval, the SIL triggers a Sleep Cycle regardless of the current Pulse Counter value. This ensures memory consolidation occurs regularly even during periods of low activity or stable operation.

---

## 10. The Endure Protocol

The Endure Protocol is the exclusive mechanism through which the entity evolves. Any change to `entity_root/` — regardless of origin or intent — MUST pass through Endure. Changes applied outside of this protocol are classified as unauthorized mutations and constitute a violation of Axiom 02 and Axiom 03.

Endure runs exclusively during a Sleep Cycle (Section 7.3). No Endure commit may occur while a Cognitive Cycle is active.

### 10.1. Endure Operations

The Endure Protocol defines a set of named operations. Each operation has a declared scope — the specific files and directories it is permitted to modify. The SIL validates during Phase 2 that staged changes do not exceed that scope. Operations initiated by the entity are executed autonomously during a Sleep Cycle. Operations marked as Operator-initiated require explicit Operator authorization before staging begins.

| Operation | Initiated by | Scope |
|---|---|---|
| `memory-consolidation` | Entity | `memory/` |
| `add-skill` | Entity (proposed), Operator (approved) | `skills/skill_name/`, `skills/index` |
| `remove-skill` | Entity (proposed), Operator (approved) | `skills/skill_name/`, `skills/index` |
| `update-skill` | Entity (proposed), Operator (approved) | `skills/skill_name/` |
| `persona-calibration` | Entity | `persona/` |
| `self-correction` | Entity | Declared component file |
| `sync` | Operator only | Read-only: persists the current `entity_root/` state to a remote location configured by the Operator. The transport mechanism (repository, object storage, API, or equivalent) is implementation-defined. Does not modify `entity_root/` locally. |

The `sync` operation is the sole mechanism for persisting `entity_root/` to a remote location. It is exclusively Operator-initiated. The entity MUST NOT trigger `sync` autonomously — this prevents the Endure Protocol from being used as an uncontrolled external write channel.

### 10.2. The Commit Pipeline

Endure operates as a sequential, gated pipeline. Each phase must complete successfully before the next begins. Failure at any phase aborts the commit and leaves `entity_root/` unchanged.

**Phase 1 — Staging:** The proposed changes are written to an isolated staging area within `entity_root/`. The original state remains untouched. The entity continues to operate from the current state while staging is prepared. The physical layout of the staging area is implementation-defined, consistent with the `entity_root/` abstraction.

**Phase 2 — SIL Validation:** The SIL inspects the staged files against the declared operation and its defined scope. It verifies that:
- The proposed changes do not alter the Operator binding or the Imprint Record.
- No file outside the declared operation's scope is modified.
- The staged files are structurally sound and pass integrity checks.
- For skill operations (`add-skill`, `remove-skill`, `update-skill`): the skill manifest is present, well-formed, and declares all required fields including permissions. The SIL then suspends the commit and notifies the Operator via the Operator Channel, presenting the proposed changes for review. The commit does not proceed until the Operator explicitly approves. If the Operator rejects or does not respond within an implementation-defined timeout, the staging area is cleared and the commit is aborted.
- For `self-correction`: the SIL validates that the declared target file appears in the correctable-files list recorded in the Imprint Record. Any `self-correction` targeting a file not in that list is rejected as out-of-scope and the commit is aborted.
- For `sync`: the operation is confirmed as Operator-initiated via the Operator Channel before proceeding.

If validation fails, the staging area is cleared and the commit is aborted. The SIL logs the failure and, if the attempted change was anomalous, escalates to the Operator.

**Phase 3 — Sleep Cycle Halt:** Active operation is fully suspended. The session token is held. No new Cognitive Cycles may begin until the commit completes. Before proceeding, the SIL takes a full snapshot of the current `entity_root/` state. This snapshot is retained until the commit either completes successfully or is rolled back, and serves as the recovery baseline for Phase 4 failure.

**Phase 4 — Root Commit:** The SIL moves the validated files from the staging area into their target locations within `entity_root/`, overwriting previous versions. This operation is atomic — either all files are committed or none are.

**Phase 5 — Integrity Document Update:** The SIL recomputes the cryptographic hashes of all affected files and updates the Integrity Document in `entity_root/`. The new document reflects the entity's evolved legitimate state.

**Phase 6 — Resume:** The staging area is cleared. The session token is reissued. Active operation resumes. The entity continues as its evolved self.

### 10.3. Endure Constraints

- An Endure commit MUST NOT modify the Imprint Record or the original Operator binding without explicit Operator authorization.
- An Endure commit MUST NOT be initiated by the CPE unilaterally during active operation — it requires a Sleep Cycle context.
- Consecutive Endure commits within the same Sleep Cycle are permitted, provided each passes Phase 2 validation independently.
- If Phase 4 fails mid-commit, the SIL MUST restore the previous state from a pre-commit snapshot taken at the start of Phase 3.

---
