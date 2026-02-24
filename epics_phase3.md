# Phase 3 Epics (Deferred)

Optional/advanced epics deferred to Phase 3 implementation.

---

## EP-16 — Household Formation & Entity Linking

Build automated household formation logic that groups related entities (spouses, joint account holders, family members) based on shared payment signals and address history. Leverage temporal entity tables to track household membership changes over time. Expose household API endpoints for FI customer service workflows.

**Stories:**
- S-16-01: Household Schema & Temporal Tracking
- S-16-02: Household Merge Rules (by address, surname, joint accounts)
- S-16-03: Household API (list members, add/remove, history)
- S-16-04: Household Integration Tests

---

## EP-17 — Probabilistic Scorer & Weak Signal ML

Build a probabilistic scoring model that learns from resolved (correct) and unresolved (ignored/rejected) stubs to predict match likelihood. Use weak signals (name similarity, address distance, transaction frequency proximity) to elevate Tier-4 INFERRED stubs toward Tier-3 confidence bands. Train on Phase 1 resolution data to identify patterns missed by deterministic blocking keys.

**Stories:**
- S-17-01: Feature Engineering (name_similarity, address_distance, velocity)
- S-17-02: ML Pipeline (training, offline inference on stubs)
- S-17-03: Weak Signal Scoring API (confidence boost for Tier-4)
- S-17-04: Probabilistic Scorer Integration Tests
- S-17-05: Model Evaluation & SLO Metrics

---

## EP-18 — Payment Network Discovery & Pattern Analysis

Build analytics on payment flow patterns to identify emerging relationships (sender → recipient networks, business supply chains, informal lending circles). Expose aggregated network insights to FIs for fraud detection, AML, and customer segmentation workflows. Privacy-preserving: expose only anonymized network metrics and patterns, never individual transaction routes.

**Stories:**
- S-18-01: Payment Network Model (directed graph, transaction volume, frequency)
- S-18-02: Network Clustering & Community Detection
- S-18-03: Privacy-Preserving Analytics API (anonymized network patterns)
- S-18-04: Network Integration Tests

---

## EP-19 — Advanced Adapter: Wire Network Analysis

Extend wires adapter (EP-11) with network analysis: track recurring international wire corridors, flag anomalies (new beneficiary, unusual amount, weekend timing), and integrate with external correspondent banking databases to enrich beneficiary entity data.

**Stories:**
- S-19-01: Correspondent Banking Database Integration
- S-19-02: Wire Network Pattern Recognition
- S-19-03: Anomaly Detection in Wire Flows
- S-19-04: Wire Network Integration Tests

---

## EP-20 — Regulatory Reporting & Compliance Dashboard

Build automated regulatory reporting for CTR (Currency Transaction Report), SAR (Suspicious Activity Report), and AML compliance. Expose dashboard for compliance officers showing: entity risk scores, transaction patterns, network anomalies, and audit trails.

**Stories:**
- S-20-01: Entity Risk Scoring (CTR/SAR/AML signals)
- S-20-02: Regulatory Report Generation (CTR, SAR templates)
- S-20-03: Compliance Officer Dashboard
- S-20-04: Audit Trail & Evidence Collection
- S-20-05: Compliance Reporting Integration Tests
