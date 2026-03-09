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
4. [Integrity Enforcement](#4-integrity-enforcement)
   - 4.1 [Semantic Probes](#41-semantic-probes)
   - 4.2 [Drift Policy](#42-drift-policy)
   - 4.3 [Heartbeat Configuration](#43-heartbeat-configuration)
   - 4.4 [Watchdog Configuration](#44-watchdog-configuration)
   - 4.5 [Integrity Log Policy](#45-integrity-log-policy)
5. [Execution Policy](#5-execution-policy)
   - 5.1 [Action Ledger](#51-action-ledger)
   - 5.2 [Background Skills](#52-background-skills)
   - 5.3 [Skill Reprocessing](#53-skill-reprocessing)
6. [Memory Policy](#6-memory-policy)
   - 6.1 [Pre-Session Buffer](#61-pre-session-buffer)
7. [Fault & Recovery](#7-fault--recovery)
   - 7.1 [Crash Recovery](#71-crash-recovery)
   - 7.2 [Context Window Policy](#72-context-window-policy)
8. [Escalation](#8-escalation)
   - 8.1 [Operator Channel](#81-operator-channel)
   - 8.2 [Passive Distress Beacon](#82-passive-distress-beacon)
9. [CMI Policy](#9-cmi-policy)
10. [Evolution Gate](#10-evolution-gate)
11. [Compliance](#11-compliance)

---

## 1. Introduction

HACA-Core is the zero-autonomy Cognitive Profile for HACA-compliant entities. It is designed for deployments where the entity must operate within a fully predictable, auditable, and Operator-controlled boundary: industrial agents, critical automation pipelines, regulated environments, and any context where autonomous structural change is unacceptable.

HACA defines two mutually exclusive Cognitive Profiles. HACA-Core enforces zero autonomy — the entity's structure is sealed at Imprint and changes only by explicit Operator instruction. HACA-Evolve permits supervised autonomy — the entity may propose and execute structural evolution within an authorization scope granted by the Operator at first activation. The choice between profiles is a deployment decision: HACA-Core when control and auditability are the primary requirements; HACA-Evolve when the entity needs to grow and adapt within defined boundaries.

HACA-Core requires Transparent CPE topology. It assumes a cooperative but potentially faulty host environment and presupposes that the HACA integration layer has exclusive control over all component interaction and host actuation — a guarantee that Opaque topologies cannot provide.

This document defines the five axioms that govern HACA-Core behavior, followed by the profile-specific policies HACA-Arch delegates to the active Cognitive Profile: how axioms are enforced and measured by the integrity layer, how skills are executed safely, how session state is managed, how the system responds to faults, and how the escalation path to the Operator is structured. It closes with the Evolution Gate — the sole authorized path for structural change under this profile — and the compliance requirements an implementation must satisfy.

## 2. Relationship to HACA-Arch

HACA-Core operates within the structural framework defined by HACA-Arch. All invariants defined in HACA-Arch apply without exception. This document defines the profile-specific policies that HACA-Arch explicitly delegates to the active Cognitive Profile: drift thresholds and response procedures, memory consolidation policy, boot behavior, session policy, skill reprocessing criteria, CMI constraints, and the Operator authorization gate for structural evolution.

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
