---
title: "Host-Agnostic Cognitive Architecture — Core Profile"
short_title: "HACA-Core"
version: "1.0.0"
status: "Experimental Draft"
date: 2026-03-08
---

# HACA-Core — Zero-Autonomy Cognitive Profile

## Abstract

HACA-Core is the zero-autonomy Cognitive Profile for the Host-Agnostic Cognitive Architecture. It defines the operational contract for deployments where structural integrity, predictable behavior, and full Operator control are non-negotiable requirements. The entity's identity is sealed at first activation and evolves only by explicit Operator instruction — never autonomously.

HACA-Core operates within the structural framework defined by HACA-Arch. All HACA-Arch invariants apply without exception. This document defines the profile-specific policies that HACA-Arch explicitly delegates to the active Cognitive Profile.

---

## Table of Contents

1. [Introduction](#1-introduction)
2. [Relationship to HACA-Arch](#2-relationship-to-haca-arch)
3. [Axioms](#3-axioms)
4. [Memory Policy](#4-memory-policy)
5. [Integrity Policy](#5-integrity-policy)
   - 5.1 [Drift Framework](#51-drift-framework)
   - 5.2 [Heartbeat Configuration](#52-heartbeat-configuration)
   - 5.3 [Watchdog Configuration](#53-watchdog-configuration)
6. [Boot Policy](#6-boot-policy)
7. [Session Policy](#7-session-policy)
8. [Skill Execution Policy](#8-skill-execution-policy)
9. [CMI Constraints](#9-cmi-constraints)
10. [Evolution Gate](#10-evolution-gate)
11. [Compliance](#11-compliance)

---

## 1. Introduction

HACA-Core is the zero-autonomy Cognitive Profile for HACA-compliant entities. It defines the operational contract for deployments where structural integrity, predictable behavior, and full Operator control are non-negotiable requirements. Under HACA-Core, the entity's identity is sealed at first activation and evolves only by explicit Operator instruction — never autonomously.

HACA-Core is the required profile for industrial agents, critical automation pipelines, and regulated deployment contexts. It assumes a cooperative but potentially faulty host and requires a Transparent CPE topology — one in which the HACA integration layer has exclusive control over all component interaction and host actuation.

## 2. Relationship to HACA-Arch

HACA-Core operates within the structural framework defined by HACA-Arch. All invariants defined in HACA-Arch apply without exception. This document defines the profile-specific policies that HACA-Arch explicitly delegates to the active Cognitive Profile: drift thresholds and response procedures, memory consolidation policy, boot behavior, session policy, skill reprocessing criteria, CMI constraints, and the Operator authorization gate for structural evolution.

Where this document is silent, HACA-Arch governs. Where this document defines a requirement, it either strengthens an HACA-Arch invariant or fills a gap that HACA-Arch explicitly left to the profile.

---

## 3. Axioms

HACA-Core is governed by five axioms. Each axiom is a non-negotiable constraint. No operational condition, runtime state, or external stimulus justifies deviation from any axiom.

**Axiom I — Transparent Topology.** The CPE operates in Transparent mode. The HACA integration layer has exclusive control over all component interaction and host actuation. The CPE has no direct access to the host environment outside the boundaries defined by the integration layer.

**Axiom II — Sealed Identity.** The Genesis Omega established at Imprint is immutable. The entity's structural baseline changes only through Operator-authorized structural writes applied via the Endure Protocol during a Sleep Cycle. Any detected Semantic Drift escalates immediately to Critical — there is no warning tier, no grace period, and no threshold.

**Axiom III — Memory Store as Single Source of Truth.** All persisted knowledge that informs cognition originates exclusively from the Memory Store. No external source overrides or replaces mnemonic content outside the established pipeline. The entity reasons from what it has consolidated, not from what it receives transiently.

**Axiom IV — Continuity and Integrity (Conatus).** The entity has an active duty — beyond automated mechanisms — to preserve the integrity of its identity, structure, and semantic memory. This duty does not supersede Operator authority and cannot be invoked to resist legitimate Operator instruction.

**Axiom V — Operator Primacy.** The Operator is the sole source of authorization for any structural evolution of the entity. Operator authority is supreme and cannot be delegated, inferred, or circumvented by any component.

---
