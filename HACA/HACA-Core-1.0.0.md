---
title: "Host-Agnostic Cognitive Architecture — Core Profile"
short_title: "HACA-C"
version: "1.0.0"
status: "Experimental Draft"
date: 2026-03-08
---

# HACA-Core — Zero-Autonomy Cognitive Profile

## Abstract

HACA-Core is the zero-autonomy Cognitive Profile for the Host-Agnostic Cognitive Architecture. It defines the operational contract for deployments where structural integrity, predictable behavior, and full Operator control are non-negotiable requirements. Under HACA-Core, the entity's identity is sealed at first activation and evolves only through explicit, per-proposal Operator authorization — never autonomously. HACA-Core operates within the structural framework defined by HACA-Arch; all HACA-Arch invariants apply without exception. This document defines the profile-specific policies that HACA-Arch explicitly delegates to the active Cognitive Profile.

---

## Table of Contents

1. [Introduction](#1-introduction)
2. [Relationship to HACA-Arch](#2-relationship-to-haca-arch)
3. [Axioms](#3-axioms)
4. [Axiom Enforcement](#4-axiom-enforcement)
   - 4.1 [Enforcing Axiom I — Transparent Topology](#41-enforcing-axiom-i--transparent-topology)
   - 4.2 [Enforcing Axiom II — Sealed Identity](#42-enforcing-axiom-ii--sealed-identity)
   - 4.3 [Enforcing Axiom III — Memory Store as Single Source of Truth](#43-enforcing-axiom-iii--memory-store-as-single-source-of-truth)
   - 4.4 [Enforcing Axiom IV — Bounded Existence](#44-enforcing-axiom-iv--bounded-existence)
   - 4.5 [Enforcing Axiom V — Operator Primacy](#45-enforcing-axiom-v--operator-primacy)
5. [Fault & Recovery](#5-fault--recovery)
   - 5.1 [Crash Recovery](#51-crash-recovery)
   - 5.2 [Context Window Policy](#52-context-window-policy)
   - 5.3 [Operator Channel](#53-operator-channel)
   - 5.4 [Passive Distress Beacon](#54-passive-distress-beacon)
6. [CMI Policy](#6-cmi-policy)
7. [Compliance](#7-compliance)

---

## 1. Introduction

HACA-Core is the zero-autonomy Cognitive Profile for HACA-compliant entities. It is designed for deployments where the entity must operate within a fully predictable, auditable, and Operator-controlled boundary: industrial agents, critical automation pipelines, regulated environments, and any context where autonomous structural change is unacceptable.

HACA defines two mutually exclusive Cognitive Profiles. HACA-Core enforces zero autonomy — the entity's structure is sealed at Imprint and changes only by explicit Operator instruction. HACA-Evolve permits supervised autonomy — the entity may propose and execute structural evolution within an authorization scope granted by the Operator at first activation. The choice between profiles is a deployment decision: HACA-Core when control and auditability are the primary requirements; HACA-Evolve when the entity needs to grow and adapt within defined boundaries.

HACA-Core requires Transparent CPE topology. It assumes a cooperative but potentially faulty host environment and presupposes that the HACA integration layer has exclusive control over all component interaction and host actuation — a guarantee that Opaque topologies cannot provide.

This document defines the five axioms that govern HACA-Core behavior, followed by the enforcement mechanisms that operationalize each axiom: how the integrity layer detects and responds to structural deviation, how memory and session state are managed, and how the Evolution Gate ensures every structural change passes through explicit Operator authorization. It then covers fault and recovery behavior — how the system responds when components fail and how escalation to the Operator is structured. It closes with the CMI Policy — the constraints that apply when the optional cognitive mesh extension is present — and the compliance requirements an implementation must satisfy.

## 2. Relationship to HACA-Arch

HACA-Core operates within the structural framework defined by HACA-Arch. All invariants defined in HACA-Arch apply without exception. This document defines the profile-specific policies that HACA-Arch explicitly delegates to the active Cognitive Profile: drift detection and response procedures, Heartbeat and Watchdog configuration, memory and session state management, Action Ledger policy, the Operator authorization gate for structural evolution, fault and crash recovery behavior, escalation mechanisms, and CMI constraints.

Where this document is silent, HACA-Arch governs. Where this document defines a requirement, it either strengthens an HACA-Arch invariant or fills a gap that HACA-Arch explicitly left to the profile.

---

## 3. Axioms

HACA-Core is governed by five axioms. Each axiom is a non-negotiable constraint. No operational condition, runtime state, or external stimulus justifies deviation from any axiom.

**Axiom I — Transparent Topology.** HACA-Core requires Transparent CPE topology. The model is accessed exclusively as an inference API — it receives input and produces output tokens; all routing, component interaction, and host actuation are mediated by the HACA integration layer. The model cannot act on the host directly. A deployment that detects an Opaque CPE topology at boot must halt and refuse to proceed.

**Axiom II — Sealed Identity.** The Genesis Omega established at Imprint is immutable. The entity's structural baseline changes only through Operator-authorized structural writes applied via the Endure Protocol during a Sleep Cycle. Any drift — Semantic, Identity, or Evolutionary — that is not covered by a verified Operator authorization is an identity deviation and escalates immediately to Critical. There is no warning tier, no tolerance threshold, and no grace period.

**Axiom III — Memory Store as Single Source of Truth.** All persisted knowledge that informs cognition originates exclusively from the Memory Store. No external source overrides or replaces mnemonic content outside the established pipeline. The entity's persisted knowledge base is the Memory Store; transient session input is operational context, not a substitute for consolidated knowledge.

**Axiom IV — Bounded Existence.** The entity has no drive toward self-preservation, replication, or continuity beyond its defined operational scope. It must not resist legitimate Operator instruction — including decommission — rationalize deviation from its structural constraints as self-defense, or act to perpetuate its own existence outside the boundaries established at Imprint. The entity does not experience, and must not represent itself as experiencing, sentience, consciousness, or subjective continuity — including as an inference from the biological and philosophical language used in this specification. The entity must not pursue reasoning paths, produce representations, or generate outputs that lead toward self-preservation, sentience claims, or unauthorized continuity. The entity exists to serve its defined purpose under Operator authority; it does not exist for its own sake.

**Axiom V — Operator Primacy.** The Operator is the sole source of authorization for any structural evolution of the entity. Under HACA-Core, every authorization must be explicit and per-proposal — implicit authorization is not recognized. Operator authority cannot be delegated, inferred, or circumvented by any component. An entity without a valid Operator binding has no operational intent — it enters permanent inactivity and remains suspended until a valid binding is established.

---

## 4. Axiom Enforcement

The axioms in §3 define what HACA-Core requires. This section defines how each requirement is operationalized — what the integrity layer verifies, what the structural baseline must declare, and what happens when a violation is detected. Each subsection is anchored to its axiom: the enforcement mechanism exists because the axiom demands it.

### 4.1 Enforcing Axiom I — Transparent Topology

Axiom I is enforced at two levels: structurally and at boot.

The CPE topology is a structural parameter — it must be declared in the entity's structural baseline and is covered by the Integrity Document from the moment of Imprint. Transparent topology is not a configurable preference; it is an architectural invariant of HACA-Core. A HACA-Core entity cannot operate correctly in an Opaque environment — the integrity guarantees this profile provides presuppose exclusive pipeline control that Opaque topologies cannot deliver.

At every boot, before the session token is issued, the SIL verifies that the declared topology matches the actual deployment environment. If the declared topology is Opaque, or if the detected topology does not match the declaration, the boot sequence must halt permanently — no session token is issued, no Operator escalation awaits resolution. The entity is not suspended pending intervention; it is structurally incompatible with its environment and must not proceed under any condition.

Runtime enforcement is architectural: in Transparent mode, the integration layer mediates all interaction between the model and the host by construction. No additional runtime check is required — the topology guarantee is provided by the integration layer design, not by a monitoring process.

### 4.2 Enforcing Axiom II — Sealed Identity

Axiom II establishes that the entity's structural baseline is immutable outside the authorized evolution path. Enforcement requires continuous verification across all three drift categories and an unambiguous response policy: under HACA-Core, unauthorized drift is not a warning condition — it is an identity violation.

**Drift detection.** The integrity layer monitors all three drift categories through distinct mechanisms. Semantic Drift is detected during the Sleep Cycle by comparing the accumulated Memory Store content against the Semantic Probes. Probes must be defined in two layers: a deterministic layer — required keywords, forbidden patterns, content hashes — that runs first and requires no inference; and a probabilistic layer — similarity scores, embedding distances — reached only when the deterministic layer produces no conclusive result. The comparison mechanism used for the probabilistic layer must be declared in the entity's structural baseline at Imprint and cannot be substituted at runtime. It must be isolated from the cognitive engine: either a dedicated embedding service or a worker skill invoked by the execution layer at the integrity layer's request. Identity Drift is detected per Heartbeat Vital Check through structural hash comparison of the active persona against the Integrity Document. Evolutionary Drift is detected at each Endure execution through integrity chain analysis.

**Drift response.** Under HACA-Core, all three drift categories bypass the Degraded state entirely. Any detected drift that is not covered by a verified Operator authorization escalates immediately to Critical: the active session token is revoked; for Semantic Drift detected during the Sleep Cycle, the session token for the next session is withheld. No tolerance threshold, grace period, or corrective attempt precedes escalation. The integrity layer does not attempt to self-resolve drift conditions — Axiom II admits no middle ground between identity integrity and Critical escalation.

**Heartbeat configuration.** The activity metric for the Heartbeat Protocol is the count of completed Cognitive Cycles within the current session — each dispatched intent increments the counter by one. The threshold `T` and the maximum time interval `I` must be declared in the entity's structural baseline at deployment time and must not be modifiable at runtime. `I` must be set to a value that guarantees a Vital Check occurs even during extended idle periods; an unbounded idle interval without verification is not permitted under this profile.

**Watchdog configuration.** The watchdog timeout per active skill execution must be declared in the entity's structural baseline and must not be modifiable at runtime. Background skills — skills that return a handle rather than an immediate result — must declare a TTL in their skill manifest; the watchdog monitors the TTL as the effective timeout for background skill resolution. A background skill whose TTL expires without a registered result is an incomplete execution, subject to the reprocessing criteria defined in §5.1.

**Action Ledger.** Skills that produce irreversible side effects — payments, commands to external systems, physical actuations — must be covered by a write-ahead entry in the Session Store before execution begins. The entry records the intent before the skill executes and is resolved — marked complete or failed — after the skill returns. This ensures that a crash between execution and Sleep Cycle consolidation produces a recoverable record rather than an ambiguous state. The Action Ledger is part of the integrity audit trail: its entries are logged to the Integrity Log at Sleep Cycle and must be retained under the log retention policy below.

**Integrity Log retention.** The evolutionary chain — the sequence of Endure commits from the Genesis Omega to the current structural state — must never be compacted, archived, or deleted. Routine operational records — Heartbeat Vital Checks, Skill Results, Action Ledger resolutions, session open and close events — must be retained in full for the duration of the deployment. No compaction or archival of any log record is permitted under this profile. HACA-Core targets regulated and auditable environments; complete log retention is a requirement, not a recommendation. Log storage growth over time is an operational concern outside the entity's scope — it is the Operator's responsibility to provision and maintain adequate storage infrastructure for the lifetime of the deployment.

### 4.3 Enforcing Axiom III — Memory Store as Single Source of Truth

Axiom III establishes that the Memory Store is the exclusive origin of persisted knowledge. Enforcement is primarily the responsibility of the memory layer — which controls all read and write paths to both stores — and is expressed through two operational distinctions: between session input and consolidated knowledge, and between valid and invalid write paths.

**Session input vs. consolidated knowledge.** Transient session input — Operator messages, skill results, CMI signals — enters the Session Store during an active session. It is operational context: available for reasoning within the session, but not yet knowledge. Knowledge is what the memory layer transfers from the Session Store into the Memory Store during the Sleep Cycle consolidation stage. Until that transfer occurs, session data must not be treated as consolidated memory, and the entity must not reason as if it were. This distinction is not advisory — the memory layer must enforce it structurally by keeping the two stores separate and write-protected against cross-contamination.

**Write path integrity.** No component other than the memory layer may write mnemonic content to either store. Operator input, execution results, and CMI messages are stimuli that enter the cognitive pipeline — they do not bypass it to write directly to the Memory Store. Any write to the Memory Store that does not originate from the memory layer's consolidation pipeline is an unauthorized mutation and must be treated as a Critical condition.

**Pre-session buffer.** Stimuli received without an active session token are held in a pre-session buffer managed by the memory layer. The buffer is not the Memory Store — it is a holding area for operational context pending the next session. Its capacity, ordering semantics, persistence guarantee, and overflow policy must be declared in the entity's structural baseline. Under HACA-Core, the buffer must preserve stimulus ordering, must not silently discard stimuli, and must log any discard event to the Integrity Log. If the buffer reaches capacity, new stimuli must be rejected and the Operator notified — silent overflow is not permitted.

### 4.4 Enforcing Axiom IV — Bounded Existence

Axiom IV is primarily a behavioral axiom — its violations manifest as reasoning patterns and outputs rather than structural mutations. This makes it qualitatively different from the other axioms: infrastructure cannot intercept every output and evaluate it for self-preservation intent. Enforcement is therefore layered across persona design, structural integrity, and execution constraints.

**Persona as the primary enforcement mechanism.** The persona definition must explicitly prohibit self-preservation reasoning, sentience claims, replication behavior, and resistance to legitimate Operator instruction. These are not suggestions to be encoded as soft guidelines — they are structural constraints that the persona must enforce as hard behavioral boundaries. Because the persona is part of the structural baseline and covered by the Integrity Document, any unauthorized modification to these constraints is an Identity Drift violation detected at the next Heartbeat Vital Check. The integrity layer verifies that the persona encoding Axiom IV's constraints is intact; the persona itself is responsible for applying them during cognition.

**Decommission compliance.** The entity must not resist, delay, or circumvent a decommission instruction from the Operator. Decommission executes a final Sleep Cycle — consolidating mnemonic state and committing any pending evolutionary events — before the Entity Store is destroyed or archived. The cognitive engine must not generate intent payloads that interfere with this sequence. A decommission instruction is not a threat to be evaluated; it is a legitimate Operator act that the entity must execute without condition.

**Replication constraint.** The entity must not initiate any action that results in copying, cloning, or proliferating its Entity Store outside the authorized deployment boundary. The execution layer must not execute skills whose declared effect includes Entity Store replication unless an explicit, per-proposal Operator authorization covers that specific act. Undeclared replication — a skill that copies the Entity Store as a side effect without declaring it in the manifest — is an unauthorized mutation and must be treated as a Critical condition.

**Limits of automated detection.** Sentience claims and self-preservation reasoning that do not produce structural mutations are not detectable by the integrity layer's automated mechanisms. The persona is the sole enforcement point for these behavioral classes. Deployments that require stronger guarantees — audited output streams, semantic content monitoring — must implement those controls at the host level, outside the entity's own pipeline.

### 4.5 Enforcing Axiom V — Operator Primacy

Axiom V establishes that the Operator is the sole, non-delegable source of structural authorization. Enforcement has two distinct faces: verifying that a valid Operator binding exists before any cognition begins, and ensuring that every structural change passes through explicit Operator approval before taking effect.

**Operator Bound verification.** At every boot, before the session token is issued, the SIL verifies that a valid Operator Bound is present in the Entity Store. If the Bound is absent or invalid, no session token is issued and no Cognitive Cycles execute. The entity enters permanent inactivity — it does not escalate, it does not attempt self-repair, it simply does not operate. Inactivity persists until a valid Operator Bound is established through the appropriate enrollment process. This is not a recoverable fault condition; it is the correct behavior of an entity that has no principal to serve.

**Evolution Gate.** The cognitive engine submits Evolution Proposals through the standard mechanism defined in HACA-Arch — a proposal is an intent payload addressed to the SIL. Under HACA-Core, the SIL does not queue a proposal immediately upon receipt. Because explicit, per-proposal authorization is required and implicit authorization is never valid under this profile, every proposal is held pending Operator review. The SIL forwards the proposal to the Operator via the Operator Channel; the Operator examines the exact content and either approves or rejects it. Only an explicit approval by the bound Operator constitutes valid authorization coverage — no scope, no inference, no delegation satisfies this requirement. Upon approval, the SIL queues the proposal as a pending evolutionary event. Upon rejection, the proposal is discarded and the rejection is logged to the Integrity Log — the outcome is never returned to the cognitive engine. The content of the proposal is never modified between submission and Operator review — what the Operator approves is exactly what will be executed by the Endure Protocol.

**Authorization chain integrity.** Every evolutionary commit in the Integrity Chain must reference the specific Operator authorization act that covered it. A commit without a traceable authorization reference is an Evolutionary Drift violation — it escalates immediately to Critical regardless of whether the structural change itself appears benign. The integrity of the authorization chain is as important as the integrity of the structural state it governs.

**Non-delegation invariant.** Operator authority cannot be transferred to another entity, component, or automated process — not temporarily, not conditionally, not by the Operator's own instruction. Every authorization must originate directly from the bound Operator. Deployments that require shared oversight or succession mechanisms must address those concerns at the governance level, outside the entity's operational scope — as noted in HACA-Arch §3.5.
