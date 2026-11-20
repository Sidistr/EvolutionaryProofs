# Evolutionary Proofs (v2.2)

**Antinomy-Driven Ratcheted Verification Escalation, Revocation Semilattices, and Domain-Separated Dual Sandboxing**

[![Version](https://img.shields.io/badge/version-v2.2-blue.svg)]()
[![Paper License: CC BY 4.0](https://img.shields.io/badge/Paper%20License-CC%20BY%204.0-lightgrey.svg)](https://creativecommons.org/licenses/by/4.0/)
[![Repo License: MIT](https://img.shields.io/badge/Repo%20License-MIT-green.svg)](./LICENSE)
[![DOI](https://img.shields.io/badge/Zenodo-10.5281%2Fzenodo.[<ZENODO_DOI>](https://doi.org/10.5281/zenodo.24383738)-0A66C2.svg)](https://doi.org/10.5281/zenodo.[<ZENODO_DOI>](https://doi.org/10.5281/zenodo.24383738))

> **Scope & Positioning:** This repository hosts a **conceptual position paper and architectural blueprint**—not a production library, empirical benchmark, or mechanized security proof. Its purpose is to formalize a coherent threat model, expose explicit architectural boundaries (*Schranke*), and invite critical peer feedback from cryptography and systems security researchers.

---

## Overview

Modern cryptographic verification protocols typically lock themselves into a **static trust regime**—relying exclusively on hardware enclaves (TEEs), pairing-based SNARKs, or fixed post-quantum STARKs. When an underlying assumption breaks (e.g., microarchitectural side-channels, circuit bugs, or future quantum threats), migrating live protocols without introducing man-in-the-middle (MITM) downgrade vectors remains an open systemic problem.

**Evolutionary Proofs** addresses this by framing verification and isolation upgrades as a **monotonic, antinomy-driven ratchet** rather than a linear tier ladder:

* **Revocation Join-Semilattice ($\mathcal{B}_t$):** Rather than imposing a rigid total order over incomparable primitives (such as pairing-based SNARKs vs. PQ-STARKs), the ratchet commits to a monotonically growing revocation set $\mathcal{B}_t = \mathcal{B}_t^{\text{proof}} \cup \mathcal{B}_t^{\text{iso}}$ over atomic, versioned mechanism identifiers $\mathcal{M}_t$ (`ADMIT` / `REVOKE` / `CHECKPOINT`), supporting lateral transitions and composite $k$-of-$n$ multi-prover policies ($\Phi_{k,n}$).
* **STH-Backed MMR Ratchet:** Digests of $\mathcal{B}_t$ and $\mathcal{M}_t$ are committed inside a threshold-signed Signed Tree Head ($\text{STH}_t$) anchored over an append-only Merkle Mountain Range (MMR). This preserves $O(\log t)$ consistency verification ($\pi_{\text{mmr}}$), append-only auditability, split-view detection (via witness cosigning), and deterministic derivation of role-specific active sets ($\mathcal{A}_{\text{prover}}$, $\mathcal{A}_{\text{verifier}}$, $\mathcal{A}_{\text{iso}}$).
* **Generalized Transcript Binding (`ctx`):** Binds active proofs and attestations to the current state root ($R_t$) across both interactive P2P sessions (fresh nonces) and non-interactive public verifiers such as Zcash or Ethereum (`chain_id`, `epoch_beacon`), accompanied by a conceptual downgrade game ($\text{Exp}^{\text{down}}_{\mathcal{A}}$) and an informal advantage bound.
* **Domain-Separated Dual Sandbox & Sanitizer Bridge:** Pairs the cryptographic ratchet with a software execution plane separating an untrusted **Online Edge (Sandbox A)** from a network-less **Offline Core (Sandbox B)** and an **Ephemeral Offline Staging** quarantine, mediated by a bounded-TCB **AG-ZKP Sanitizer** outside Domain 1.

---

## High-Level Architecture

```text
                       +-------------------------------------------+
                       |    DOMAIN 1: Minimal TCB / Orchestrator   |
                       |  Persistent Anchor (R_anchor, B_anchor)   |
                       |   STH & MMR Policy | Freshness Lease ΔT   |
                       +---------------------+---------------------+
                                             |
          +----------------------------------+----------------------------------+
          |                                  |                                  |
          v                                  v                                  v
+-------------------+          +---------------------------+          +-------------------+
|  SANDBOX A        |          |  BOUNDED-TCB BRIDGE       |          |  SANDBOX B        |
|  Online Edge      |  Typed   |  • Strict Schema / IPC    |  Typed   |  Offline Core     |
|  (Network Facing) | <======> |  • AG-ZKP Sanitizer*      | <======> |  (No Net Stack)   |
|  No Secrets / Key |   IPC    |  • Ephemeral Quarantine   |   IPC    |  Active Prover /  |
+-------------------+          |    (Offline Staging)      |          |  Verifier Engine  |
                               +---------------------------+          +-------------------+

* Sanitizer behavior is mechanism-specific:
  - Malleable SNARKs (e.g., Groth16): algebraic re-randomization + schema enforcement.
  - Hash-based STARKs & TEE Quotes: canonical schema checks, metadata stripping, rate limiting
    (not algebraically re-randomizable; relies on offline Sandbox B + deterministic Fiat-Shamir).
