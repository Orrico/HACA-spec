# HACA: Host-Agnostic Cognitive Architecture
**Status:** Internet-Draft  
**Version:** 1.0-Draft-04
**Authors:** J. Orrico  
**Date:** February 25, 2026

---

## Abstract

The Host-Agnostic Cognitive Architecture (HACA) defines a structural pattern for building autonomous, portable AI entities. Its core premise is that an AI agent's "mind" — its memory, identity, and intentions — should exist independently of any particular LLM provider, execution host, or proprietary framework.

HACA decouples the inference engine (LLM) from cognitive state by placing all persistent state in a universal storage layer. If you copy a HACA-compliant system's state directory and boot it on a different machine with a different LLM, the entity resumes exactly as it was — same personality, same agenda, same memories.

This document describes the architecture at a level intended for developers wishing to build HACA-compliant systems. For the complete formal specification, see the RFC drafts in the `haca-spec/` directory.

---

## 1. The Core Philosophy

Current AI agent frameworks tightly couple the model's runtime with state management. This leads to:

- **Vendor lock-in:** Your agent lives inside a specific API's memory, context window, or conversation ID. Change providers and the agent's history is gone.
- **Fragmented memory:** State is split between the LLM's implicit context, external databases, and application logic. There is no single source of truth.
- **Non-portability:** The agent cannot be cloned, paused, migrated, or version-controlled in a meaningful way.

HACA operates on a strict paradigm shift: **The compute is stateless; the storage is the mind.**

The LLM is treated as a CPU — a powerful but ephemeral processing unit that receives a snapshot of the "mind," computes the next thought or action, and returns outputs. It retains nothing between invocations. Everything that makes an entity *itself* lives in the storage layer.

---

HACA consists of four distinct logical layers. They cooperate in a tight loop, but have strictly constrained communication paths — no layer may bypass its neighbors to interact with another.

> [!NOTE]
> The following diagram is a conceptual simplification of the internal data flows. For the normative directed graph and component edges, refer to `HACA-Arch-v1.0-RFC-Draft.md`.

```
┌─────────────────────────────────────────────────────────────┐
│                   HOST ENVIRONMENT                          │
│                                                             │
│  ┌───────────────┐        ┌──────────────────────────────┐  │
│  │     MIL       │◄──────►│            CPE               │  │
│  │ Memory        │        │  Cognitive Processing Engine │  │
│  │ Interface     │        │  (LLM / Inference Engine)    │  │
│  │ Layer         │        │  — Stateless across cycles — │  │
│  └───────┬───────┘        └──────────────┬───────────────┘  │
│          │                               │                  │
│          │    ┌──────────────────────┐   │                  │
│          └───►│         SIL          │◄──┘                  │
│               │  System Integrity    │                      │
│               │  Layer               │                      │
│               └──────────┬───────────┘                      │
│                          │                                  │
│               ┌──────────▼───────────┐                      │
│               │          EL          │                      │
│               │   Execution Layer    │                      │
│               │ (Shell, APIs, Tools) │                      │
│               └──────────────────────┘                      │
└─────────────────────────────────────────────────────────────┘
```

### 2.1. Memory Interface Layer (MIL)

The single source of truth. It holds everything that makes the entity *itself*:

- **Core Identity:** Immutable files defining who the entity is, its drives, and constraints. The entity cannot modify these files at runtime.
- **State and Agenda:** The entity's short-term intentions and scheduled task queue.
- **Communication Bus:** An asynchronous spool/inbox mechanism for exchanging messages between the cognitive engine and the outside world.

The MIL is the only place where cognitive state is persisted. If it moves, the entity moves.

### 2.2. Cognitive Processing Engine (CPE)

The "CPU" of the architecture. The CPE is **strictly stateless across execution cycles**. It:

1. Wakes up and receives a full snapshot of the MIL as its input context.
2. Processes the context — generating thoughts, plans, or decisions.
3. Outputs a state delta (changes to commit to the MIL) and a list of action requests.
4. Shuts down. It retains nothing until the next invocation.

The CPE is typically an LLM (local or via API), but HACA makes no assumptions about the specific model or provider. Any inference system that satisfies the statelessness requirement is compliant.

**CPE-Transparent vs. CPE-Opaque:** In standard deployments the SIL invokes the CPE as a black-box API call and acts as the execution adapter (CPE-Transparent). In environments where the CPE has direct filesystem access — IDE assistants, agents with native tool APIs — the CPE acts as its own adapter, mapping intents to native operations directly (CPE-Opaque). Both modes are architecturally equivalent: the CPE remains stateless, the MIL remains the single source of truth, and the Adapter Contract (RBAC validation, sandbox enforcement, no persona mutation) applies in both.

### 2.3. Execution Layer (EL)

The "hands" of the entity. The CPE never interacts directly with the host system — it can't browse the web, read files, or call APIs on its own. Instead, it emits **Side-Effects**: structured requests like `"Fetch this URL"` or `"Run this script"`.

The EL is responsible for:
- Parsing Side-Effect requests from the CPE's output.
- Validating them against the entity's declared capability manifest (sandboxing).
- Executing them on the host system within authorized boundaries.
- Writing the results back into the MIL's inbox for the CPE to consume on the next cycle.

The EL is a strict mediator. It is what prevents a rogue or confused CPE from taking arbitrary actions.

### 2.4. System Integrity Layer (SIL)

A lightweight but critical governance layer. The SIL:

- Orchestrates the full boot sequence, verifying that core identity files have not been tampered with before allowing the CPE to start.
- Continuously monitors for **Semantic Drift** — behavioral deviations from the entity's intended identity. The SIL operates as an active verifier (pull model); it is not notified of state changes but initiates checks on its own schedule.
- Injects **Traps** — signed system events for fatal errors, scheduled wakeups, and lifecycle transitions — into the MIL. Traps are the SIL's authoritative out-of-band write channel.
- Enforces fault states: when something goes wrong, the SIL transitions the system into read-only or halted modes to prevent corrupted behavior from being persisted.

Think of the SIL as the entity's immune system — it doesn't do creative work, but it ensures the whole system remains healthy and trustworthy.

---

## 3. The Cognitive Loop

An entity's life in HACA is a continuous, asynchronous loop driven by state mutations, not just by user prompts:

```
         ┌─────────────────────────────────────────────┐
         │                                             │
         ▼                                             │
  [1. WAKEUP]                                         │
  A trigger fires (cron job, file-watcher,             │
  incoming message, or user input).                    │
         │                                             │
         ▼                                             │
  [2. INTEGRITY CHECK]                               │
  SIL verifies cryptographic hashes of all            │
  identity files. Aborts if any mismatch.             │
  Runs behavioral drift probes against the CPE.       │
         │                                             │
         ▼                                             │
  [3. CONTEXT ASSEMBLY]                               │
  SIL reads the MIL: identity, agenda,                │
  inbox events, active memories, and recent session.  │
  Assembles a single context snapshot for the CPE.    │
         │                                             │
         ▼                                             │
  [4. INFERENCE]                                      │
  CPE processes the assembled context.                 │
  Produces: thoughts, decisions, Side-Effect requests. │
         │                                             │
         ▼                                             │
  [5. EXECUTION]                                      │
  EL reads Side-Effect requests.                       │
  Validates against capability manifest.               │
  Executes sandboxed actions.                          │
  Writes results to MIL inbox.                         │
         │                                             │
         ▼                                             │
  [6. COMMIT]                                         │
  State delta committed atomically to MIL.             │
  Inbox consolidated into session log.                │
         │                                             │
         ▼                                             │
  [7. SLEEP]                                          │
  CPE shuts down. System waits for next trigger. ──────┘
```

The key insight is that **the CPE produces no lasting side effects directly** — all outputs are mediated through the MIL (state changes) and the EL (actions). This makes the system auditable, reproducible, and safely restartable from any point.

---

## 4. Behavioral Fidelity (Drift Control)

Maintaining a persistent identity across many execution cycles — especially when relying on external, frequently updated LLM APIs — is one of the hardest problems in autonomous agent design. A model update by an API provider, accumulated adversarial inputs, or gradual semantic degradation can silently erode an entity's behavior over time.

HACA addresses this through **Behavioral Probing**:

1. The SIL maintains a set of **canonical probe prompts** and their expected responses. These are stored in the MIL as immutable state — their integrity is verified at boot alongside the identity files.
2. Periodically (and always at boot), the SIL injects these probes into the CPE and compares the actual responses against the expected ones using Unigram NCD (Normalized Compression Distance) or similar compression-based metrics.
3. If the responses diverge beyond a configurable threshold, the SIL classifies this as a **Consistency Fault**.
4. On a Consistency Fault, the system enters **read-only mode**: the CPE can continue to respond to queries, but no state is committed to the MIL and no EL actions are executed. This prevents a degraded cognitive state from corrupting the entity's memory.
5. Normal operation resumes only after an operator resolves the root cause (e.g., rolling back to a snapshot, re-anchoring the identity).

### 4.1. Cold-Start Exception

Drift probes MUST NOT run during a cold-start (first activation). A cold-start is defined as: the entity's MIL is empty and no operator binding exists yet. There is no behavioral baseline to compare against — running probes would produce an unconditional fault.

The First Activation Protocol (see §4.2) must complete before the first drift probe baseline is established. All subsequent boots are warm-boots and drift probing proceeds normally.

### 4.2. First Activation Protocol (FAP)

When an entity boots for the first time with no prior memory, it must execute the FAP before normal operation begins. The FAP:

1. Collects the operator identity (name, handle, timezone) and persists it to the MIL as the **operator trust anchor** — the entity's binding to a supervising human.
2. Reads all identity and capability files to build a self-knowledge document in the MIL.
3. Pins the self-knowledge document to the active context at highest priority.
4. Records an activation event in the session log.
5. Removes the cold-start sentinel file — its absence is the durable signal that FAP has completed.

Without FAP, an entity has no operator trust anchor and no self-knowledge scaffold. It will behave according to its defined identity but will spend every session rediscovering its own capabilities and context — a significant operational inefficiency and a behavioral fidelity risk.

If FAP cannot complete, the system MUST declare an **Initialization Fault** and halt. Normal operation requires a completed FAP.

### 4.3. Dynamic Sentinels
To ensure robustness, high-frequency runtime state (such as recovery counters, boot counts, and sequence counters) MUST be stored in a **Dynamic Sentinel Stream** separate from the static identity files. This prevents continuous mutation of the system's cryptographically anchored identity.

This mechanism provides **detection and containment**, not prevention. HACA cannot stop drift from occurring inside the inference model, but it guarantees that detected drift is never allowed to persist.

*(See §4.1 for the cold-start exception: FAP must complete before drift probing begins.)*

---

## 5. Trust and Sandboxing

HACA assumes the execution host is **cooperative but fallible**. The threat model is not a hostile host actively trying to compromise the entity, but rather misconfigurations, bugs, or prompt injection attacks that could cause the CPE to request unauthorized actions.

**All actions emitted by the CPE MUST go through the EL**, which enforces:
- **Capability constraints:** The entity can only perform actions listed in its approved manifest (e.g., `read_file`, `web_search`). Actions not in the manifest are silently rejected and logged.
- **Workspace isolation:** Side-Effects may only operate within designated sandbox directories. FCP enforces this via **Active Confinement**, leveraging native OS Namespaces (`unshare`) to create a private kernel-level execution boundary.
- **Sandbox verification:** The SIL actively verifies the confinement boundary at boot — confirming the availability and functionality of namespace isolation — before allowing the CPE to start.

If the system detects that its sandbox has been breached or its core identity files altered, it immediately halts to prevent rogue execution.

---

## 6. Implementation Agnosticism

HACA is intentionally implementation-agnostic. It defines **what** must be true (the invariants), not **how** to achieve it.

| HACA Requirement | You can implement it with... |
|---|---|
| MIL (persistent state store) | Filesystem, SQLite, Redis, S3 |
| CPE (stateless inference) | OpenAI API, Gemini, local Ollama, any LLM |
| EL (sandboxed execution) | Shell scripts, Docker containers, WASM |
| SIL (integrity + orchestration) | A cron daemon, a Python script, a Rust binary |

The reference implementation — **FCP (Filesystem Cognitive Platform)** — demonstrates how to build a fully compliant HACA system using nothing but standard OS files and directories. See `FCP-v1.0-Internet-Draft.md` for the implementation details.

---

## 7. Self-Evolution and Capability Management

HACA entities are not static. Identity evolves as the operator refines the entity's values. Capabilities expand as new skills are developed. Instructions change as the deployment context matures. HACA requires that all of these changes go through a **controlled evolution protocol**.

### 7.1. The Evolution Gatekeeper

A HACA-Core compliant implementation MUST provide a single privileged skill — the **evolution gatekeeper** — that is the sole path for modifying any file in the integrity record. This prevents the CPE from expanding its own permissions or corrupting its own identity at runtime.

The gatekeeper enforces this invariant: **if a file is in the integrity record, it can only change through the gatekeeper**.

### 7.2. Evolution Operations

The gatekeeper MUST support at minimum:

- **Identity evolution** — propose a change to core identity files, validate with drift probes, apply only if the entity remains behaviorally consistent
- **Capability evolution** — add or remove skills and lifecycle hooks, update the RBAC registry, reseal the integrity record
- **Instruction evolution** — update the boot protocol document
- **Seal** — recompute the integrity record after any mutation
- **Sync** — commit all evolution changes to version control (with optional remote push)
- **Status** — compare current file hashes against sealed values; detect unsynchronized changes before the next boot

### 7.3. Protocol Invariants

Every evolution operation MUST:
1. Create a pre-mutation backup before any destructive change
2. Update the integrity record (seal) after applying the change
3. Emit an audit record to the MIL
4. Require explicit operator confirmation for destructive operations (removal)

**Identity evolution specifically** MUST run drift probes with the proposed new content before applying. If the entity would drift beyond threshold τ, the change is rejected. Capability and instruction evolution do not require drift probes — they change what the entity *can do*, not who it *is*.

---

## 8. Compliance Levels

HACA defines three levels of compliance. Building a system that satisfies the Core requirements is sufficient for most use cases and is where any new implementation should start.

### HACA-Core (Required)
The minimum set of invariants for a compliant system:

- [ ] CPE is stateless across execution cycles — all persistent state lives in the MIL
- [ ] MIL is the single source of truth — no shadow copies in memory or databases outside the MIL
- [ ] All host interactions go through the EL — the CPE never talks to the system directly
- [ ] Boot sequence performs cryptographic integrity verification of core identity files
- [ ] All MIL state transitions are atomic — writes either fully commit or are fully discarded
- [ ] EL actions execute within Active Confinement boundaries (host-enforced via Namespaces or equivalent isolation)
- [ ] Behavioral drift is continuously monitored via Unigram NCD; detected drift halts MIL commits
- [ ] First Activation Protocol completes before normal operation begins (cold-start requirement)
- [ ] Single privileged evolution gatekeeper controls all mutations to integrity-tracked files
- [ ] Evolution operations emit ACP audit envelopes to the MIL

### HACA-Full (Recommended for production)
HACA-Core plus:

- [ ] Cryptographic auditability: all system events signed with a host-held secret
- [ ] Replay attack prevention: monotonic sequence counters on all messages
- [ ] Byzantine-resilient integrity anchoring (pre-shared hash, hardware root of trust, or operator signature)

For full cryptographic auditability and temporal attack prevention requirements, refer to `HACA-Security-v1.0-RFC-Draft.md`.

### HACA-Mesh (Multi-agent coordination)
HACA-Full plus:

- [ ] Multiple HACA entities coordinating via a shared MIL namespace
- [ ] Cross-entity RBAC: each entity has a declared role and can only access authorized namespaces
- [ ] Cognitive Mesh protocol for asynchronous inter-agent messaging

---

## 9. Fault States

When things go wrong, HACA transitions through well-defined fault states. The system always moves toward *more* restricted states when anomalies are detected, never toward more permissive ones.

| Fault | What happened | System state |
|---|---|---|
| **Consistency Fault** | CPE behavior diverged from expected identity | Read-only: CPE responds to queries, but no commits and no EL actions |
| **Integrity Fault** | A core identity or manifest file was tampered with | Halted: boot is aborted outright |
| **Initialization Fault** | First Activation Protocol cannot complete | Suspended: system does not proceed to normal operation; FAP must be retried |
| **Recovery Fault** | Post-crash behavioral divergence exceeds safe threshold | Halted: requires manual operator intervention |
| **Sandbox Fault** | An action escaped or probing confirmed sandbox is broken | Degraded: EL disabled, CPE-only mode, commits to MIL suspended |
| **Budget Fault** | Context window or resource limit exceeded | Degraded: autonomous operation halted, operator queries only |

The most restrictive active fault always wins. A Halted system does not become Degraded when a new Sandbox Fault occurs — it stays Halted until the original fault is resolved.

---

## 10. Minimal Implementation Guide

To build a HACA-Core compliant system from scratch, you need:

### Step 1: Define the MIL structure
Choose your storage medium and organize it into at minimum:
- An **identity store** (immutable; define who the entity is)
- A **state store** (append-only; the ongoing session log)
- An **inbox** (temporary; results from the EL awaiting processing)
- An **agenda** (scheduled tasks for autonomous operation)

### Step 2: Build the SIL boot sequence
The SIL must, before every CPE invocation:
1. Verify cryptographic hashes of all identity files against a sealed integrity record
2. Load the MIL state and compose the CPE's input context
3. Run behavioral probes to verify the CPE hasn't drifted
4. Only then allow the CPE to execute

### Step 3: Implement the CPE wrapper
Wrap your LLM API so that:
- It receives the full context assembled by the SIL
- Its outputs are parsed for state deltas and Side-Effect requests
- It cannot write to the MIL directly — outputs go through the SIL first

### Step 4: Build the EL
Create a dispatcher that:
- Reads Side-Effect requests from the CPE's output
- Validates each request against a capability manifest
- Executes approved requests in an isolated environment
- Writes results back to the MIL inbox using atomic operations

### Step 5: Implement drift monitoring
Maintain a set of canonical prompt-response pairs derived from your entity's identity. Run this probe set at every boot and periodically during operation.

---

## 11. Relationship to FCP

FCP (Filesystem Cognitive Platform) is the reference implementation of HACA. It makes concrete choices for every abstract HACA requirement:

| HACA abstracts... | FCP concretizes as... |
|---|---|
| MIL | A POSIX filesystem directory tree |
| Identity store | Read-only `.md` files in `/persona/` |
| State store | Append-only `.jsonl` files (ACP envelopes) |
| Inbox | `/memory/inbox/` with atomic rename-based spooling |
| EL | Shell scripts in `/skills/` directories |
| SIL | A host daemon orchestrating the boot phases |
| Drift probes | Stored in `/state/drift-probes.jsonl` |

FCP's key insight is that the POSIX filesystem already provides most of the primitives HACA needs: atomic rename for safe writes, directory listings as indexes, symbolic links for fast context switching, and file permissions for access control. **No external database. No daemon processes. No complex registries.** The filesystem is the application.

---

## 12. Frequently Asked Questions

**Q: Does HACA require a specific LLM or provider?**  
No. The CPE is fully abstracted. You can use GPT-4, Gemini, Claude, Llama running locally, or any other model — as long as it satisfies the statelessness requirement (Axiom I). Many teams start with a cloud API for convenience and later migrate to a self-hosted model.

**Q: How is this different from LangChain/AutoGPT/CrewAI?**  
Those frameworks are largely prescriptions for how to *call* LLMs and chain their outputs. HACA is an architecture for how an agent *persists its identity over time*. They address different problems and are not mutually exclusive — you could implement the HACA pattern *on top of* a framework like LangChain.

**Q: What happens when the entity has no memory (first boot)?**
The entity executes the **First Activation Protocol (FAP)** before doing anything else. FAP collects operator identity, reads all capability definitions, writes a self-knowledge document to the MIL, and removes the cold-start sentinel. After FAP, the entity has an operator trust anchor and a memory scaffold — every subsequent boot loads these automatically. Without FAP, the entity would need to rediscover its own capabilities and operator context from scratch on every session.

**Q: Can two HACA entities communicate with each other?**  
Yes, via the HACA-Mesh extension. In the simplest case, one entity's EL can write a message to another entity's inbox, following the same atomic spooling pattern used for any other input.

**Q: Is HACA suitable for stateless/ephemeral agents that don't need persistence?**  
Technically, a fully stateless agent doesn't need HACA — HACA's value is precisely in managing persistent identity across many cycles. However, even for "short-lived" agents, the sandboxing and integrity verification aspects of HACA can add meaningful safety guarantees.

**Q: How do you handle model updates from the LLM provider?**  
This is exactly what Drift Control (Section 4) addresses. When the provider updates their model and behavior shifts, the drift probes detect it. The system halts MIL commits until the operator confirms the new behavior is acceptable and re-anchors the thresholds.

---

## 13. Status and Contributing

This Internet Draft is published for community review. The complete formal specification (with axiomatic definitions, mathematical drift metrics, and compliance test suites) is available in the `haca-spec/` directory.

**Feedback welcome via GitHub Issues:**
- Is the terminology clear? Where did you get lost?
- Does the Cognitive Loop make sense to implement?
- Did you try to build something? What was hard?
- Are there use cases not covered by this model?

The goal of this document is to lower the barrier to entry. If something here requires more explanation to be implementable, that is a bug in the document.

---

## Normative References (Full Spec)

- `HACA-Core-v1.0-RFC-Draft.md` (draft-orrico-haca-core-03) — Formal axioms, drift measurement math, compliance tests
- `HACA-Arch-v1.0-RFC-Draft.md` (draft-orrico-haca-arch-03) — Abstract architecture, topology, trust model, compliance levels
- `HACA-Security-v1.0-RFC-Draft.md` (draft-orrico-haca-security-03) — Byzantine host model, cryptographic auditability, temporal attack prevention
- `../fcp-spec/FCP-v1.0-Internet-Draft.md` (1.0-Draft-04) — Reference implementation using the POSIX filesystem
