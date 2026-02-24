# Phase 4 Epics (Future / Exploratory)

Forward-looking epics for Phase 4 and beyond. These represent emerging use cases and advanced architectures not yet prioritized.

---

## EP-21 — Graph-Based Entity Resolution (GER)

Exploratory phase: build a knowledge graph representing entities, their relationships, attributes, and confidence signals. Use graph algorithms (belief propagation, community detection) to infer new relationships and resolve identity ambiguities beyond deterministic blocking keys. Compare outcomes vs. Phase 1 tier-based approach.

**Research Goals:**
- Can graphs improve recall (find more matches) vs. tiers?
- What is the precision cost (false positives)?
- Scalability to 100M+ entities?

**Stories (exploratory):**
- S-21-01: Graph Schema & Property Model
- S-21-02: Entity-Relationship Inference Engine
- S-21-03: Graph-Based Match Scoring vs. Tier Comparison
- S-21-04: Graph Scalability Study

---

## EP-22 — Cross-Border Identity Resolution

Extend identity resolution to international entities and cross-border payment corridors:
- Multi-country address normalization (beyond USPS)
- International phone number handling (ITU E.164 + local carrier validation)
- SWIFT/IBAN validation and foreign correspondent databases
- Multi-language name matching and transliteration

**Stories:**
- S-22-01: International Address Normalization (ISO 19160-1)
- S-22-02: Multi-Country Phone Validation
- S-22-03: Correspondent Banking Data Integration
- S-22-04: Transliteration & Name Matching (Cyrillic, Arabic, CJK)
- S-22-05: Cross-Border Integration Tests

---

## EP-23 — Privacy-Preserving Secure Multi-Party Computation (SMPC)

Implement SMPC protocols to allow two FIs to link identities without revealing raw PII to each other or to JHBI:
- Oblivious Transfer (OT) for key exchange
- Garbled Circuits or Secret Sharing for privacy-preserving BK matching
- Homomorphic encryption for encrypted BK comparisons

**Research Goals:**
- Latency: can SMPC matching complete in <500ms?
- Adoption: how many FIs would use encrypted linking vs. trusting JHBI RLS?

**Stories:**
- S-23-01: SMPC Protocol Selection & POC
- S-23-02: Encrypted Blocking Key Comparison
- S-23-03: SMPC Performance Benchmarks
- S-23-04: SMPC Integration Tests

---

## EP-24 — Zero-Knowledge Proofs (ZKP) for Identity Attestation

Exploratory: enable entities to
 prove identity attributes (e.g., "I am the account holder") without revealing the account number or routing number to an external party. Use ZKP circuits to assert BK matches and tier thresholds.

**Research Goals:**
- Can ZKP reduce fraud in account-linked workflows?
- What is the verification latency?

**Stories:**
- S-24-01: ZKP Circuit Design for BK-1/BK-2
- S-24-02: ZKP Proof Generation & Verification
- S-24-03: ZKP Integration Tests

---

## EP-25 — Biometric & Behavioral Signal Integration

Exploratory: integrate biometric signals (fingerprint, iris, voice) and behavioral signals (typing speed, mouse movement, device fingerprint) as novel blocking keys or confidence boosters. Evaluate privacy and ethical implications.

**Research Goals:**
- Does biometric reduce false-positive merges?
- How to handle privacy regulations (BIPA, GDPR) for biometric storage?

**Stories:**
- S-25-01: Biometric Signal Capture & Normalization
- S-25-02: Behavioral Signal Feature Extraction
- S-25-03: Biometric + Behavioral Blocking Key Design
- S-25-04: Privacy & Compliance Impact Assessment

---

## EP-26 — Real-Time Identity Enrichment & Third-Party Data

Integrate third-party data sources (credit bureaus, address validators, fraud indicators) in real-time to enrich entity profiles and boost confidence scores.

**Stories:**
- S-26-01: Third-Party Data Adapter Framework
- S-26-02: Credit Bureau Integration (Equifax, Experian, TransUnion)
- S-26-03: Address Validator Integration (SmartyStreets, UPS, USPS)
- S-26-04: Fraud Indicator Integration (SafeGraph, Minfraud)
- S-26-05: Third-Party Data Caching & SLA Management

---

## EP-27 — Quantum-Ready Cryptography

Prepare for post-quantum cryptography standards (NIST PQC). Evaluate lattice-based signatures for future blocking key derivation and audit log integrity. NOT urgent, but important for forward-planning.

**Stories:**
- S-27-01: NIST PQC Algorithm Evaluation
- S-27-02: Lattice-Based Signature POC
- S-27-03: Migration Plan: Classical → PQC

