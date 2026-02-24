# Phase 4 Stories (Future / Exploratory)

Research and exploratory stories for Phase 4. These represent experimental directions and proof-of-concepts not yet committed to full development.

---

## EP-21 — Graph-Based Entity Resolution (GER)

### S-21-01 Graph Schema & Property Model
As a research engineer, I want a knowledge graph schema representing entities, relationships, attributes, and confidence scores, so that I can experiment with graph algorithms for identity resolution.
- AC: Graph nodes: entities (idr_customer_id), attributes (ssn, email, phone, address, account_number), and relationship types.
- AC: Graph edges: OWNS (entity → account), HAS_EMAIL, HAS_PHONE, HAS_ADDRESS, CONNECTED_BY (entity → entity bidirectional).
- AC: Edge weights: confidence_score (0.0–1.0) based on matching tier or probabilistic scorer.
- AC: Graph tool: Neo4j or Memgraph (proof-of-concept).
- AC: Sync: daily batch job exports PostgreSQL entities + relationships to graph DB.

### S-21-02 Entity-Relationship Inference Engine
As a research engineer, I want graph inference algorithms (transitivity, belief propagation) that compute relationship likelihood between unmatched entities, so that I can discover new relationships beyond deterministic blocking keys.
- AC: Algorithm 1: Transitivity rule—if A owns both account1 and account2, and B is linked to account1 via blocking keys, infer A ↔ B relationship with confidence inherited from BK match.
- AC: Algorithm 2: Belief propagation—propagate match confidence through the graph (if A ≈ B and B ≈ C with high confidence, boost A ≈ C inference).
- AC: Output: inferred relationship candidates scored by algorithm confidence.

### S-21-03 Graph-Based Match Scoring vs. Tier Comparison
As a data scientist, I want to compare graph-inferred matches against Phase 1 tier-based matches on historical data, so that I can evaluate if graphs improve recall or precision.
- AC: Holdout test set: 100K historical Tier-4 stubs that were later confirmed as true positives (Phase 1 precision validation).
- AC: Run graph algorithm on snapshot of data before confirmation.
- AC: Metric 1: Recall—% of true-positive stubs discovered by graph vs. tier-based method.
- AC: Metric 2: Precision—% of graph-inferred matches that are actually true positives.
- AC: Report: if graph achieves recall > 80% and precision > 75%, proceed to Phase 4 production.

### S-21-04 Graph Scalability Study
As a platform engineer, I want to benchmark graph-based resolution on 100M+ entities and 1B edges, so that I know if graph approach is feasible at scale.
- AC: Infrastructure: Neo4j Enterprise on AWS, sized for 1B edges.
- AC: Benchmark: infer 100 new relationships per minute without impacting tier-based resolution latency.
- AC: Report: memory footprint, query latency, weekly update time.

---

## EP-22 — Cross-Border Identity Resolution

### S-22-01 International Address Normalization (ISO 19160-1)
As a resolution engineer, I want address normalization for 50+ countries following ISO 19160-1 standard, so that international payment parties are correctly deduplicated.
- AC: Standard: ISO 19160-1 (Postal Address Standard); parsing rules for each country.
- AC: Support: US, Canada, UK, EU (DE, FR, IT, ES, NL), APAC (Japan, Singapore, Australia), LATAM (Mexico, Brazil), Middle East (UAE, Saudi).
- AC: Parsing: country code + region + postal code + street address + unit normalization per country rules.
- AC: Validation: check against known postal authority databases (Google Earth, OpenStreetMap, country postal services).
- AC: Fallback: if normalization fails, store raw address with confidence=LOW, flag for manual review.

### S-22-02 Multi-Country Phone Validation
As a resolution engineer, I want phone number validation for all countries using ITU E.164 and country-specific carrier databases, so that international phone-based matches work correctly.
- AC: ITU E.164: all phones normalized to +CC-XXX...X (1–15 digits).
- AC: Country NANP (North American Numbering Plan) lookup: verify against known carrier databases for US/CA/MX/PR.
- AC: International: integrate with international carrier databases (Telesign, TwilioAPI) for carrier validation outside NANP.
- AC: Reject invalid formats; accept as BK-4 only if validated.
- AC: Test: 100+ sample phone numbers per country.

### S-22-03 Correspondent Banking Data Integration
As a wires engineer, I want to integrate real-time correspondent banking directories (SWIFT, FedWire, fedACH), so that international wire beneficiaries are linked to verified banks.
- AC: Data source: SWIFT directory (updated monthly), FedWire participant list (daily).
- AC: Schema: bic_code, bank_name, country, branch_code, contact_swift_bic.
- AC: Matching: on wire ingest, look up SWIFT BIC against directory; if match, add bank entity as known correspondent.
- AC: Enrichment: add correspondent attributes to beneficiary stub (bank_name, country, regulatory_status).
- AC: SLA: directory updates synced within 1 hour of publication.

### S-22-04 Transliteration & Name Matching (Cyrillic, Arabic, CJK)
As a resolution engineer, I want name matching algorithms that handle transliteration and phonetic matching across different scripts (Cyrillic, Arabic, Chinese, Japanese, Korean), so that international names are matched despite romanization variations.
- AC: Transliteration: use standard transliteration tables (ISO/IEC 7498 for Cyrillic, ALA-LC for Arabic, Pinyin for Mandarin, Romaji for Japanese, Hangul for Korean).
- AC: Phonetic matching: for each language, use phonetic matching (Soundex for English, Metaphone for European, Daitouryouon for Japanese).
- AC: Name matching: Levenshtein + phonetic match. Accept if Levenshtein >= 0.8 OR phonetic match AND same family name (if applicable).
- AC: Test: 50 names per language extracted from real payment data.

### S-22-05 Cross-Border Integration Tests
As a QA engineer, I want integration tests for cross-border resolution covering all major regions, so that regressions in international matching are caught.
- AC: Test: German address (with umlauts) normalized → matches duplicate with ASCII address.
- AC: Test: Swahili name in Cyrillic script transliterated → matches Latin-script variant.
- AC: Test: Chinese name matching via Pinyin → finds duplicate with different Romanization.
- AC: Test: International wire with SWIFT BIC → matched to known correspondent bank.
- AC: Test coverage: US, Canada, UK, Germany, France, Japan, Brazil, UAE.

---

## EP-23 — Privacy-Preserving Secure Multi-Party Computation (SMPC)

### S-23-01 SMPC Protocol Selection & POC
As a research engineer, I want to evaluate SMPC protocols (Garbled Circuits, Secret Sharing) for privacy-preserving BK matching between two FIs, so that JHBI can offer BK matching without access to plaintext keys.
- AC: Protocol 1 evaluation: Garbled Circuits (Yao's protocol) for Boolean circuits.
- AC: Protocol 2 evaluation: Shamir's Secret Sharing for threshold cryptography.
- AC: POC: implement simple BK equality comparison (does BK from FI-A == BK from FI-B?) using one protocol.
- AC: Metric: latency, proof size, communication rounds.
- AC: Decision: by end of Phase 4, recommend one protocol for full implementation or defer to Phase 5.

### S-23-02 Encrypted Blocking Key Comparison
As a protocol engineer, I want SMPC circuits that securely compare blocking keys from two FIs without revealing the keys to each other or to JHBI, so that cross-FI linking respects privacy.
- AC: Circuit: inputs = BK1_FI_A (encrypted), BK1_FI_B (encrypted); output = match_bit (1 if equal, else 0).
- AC: Protocol: FI-A and FI-B interact via SMPC server (potentially JHBI, potentially independent third party).
- AC: Guarantee: JHBI learns only match result (0 or 1), not the plaintext BKs.
- AC: Output: match result is BK-1 level (Tier 1 DEFINITIVE if match).

### S-23-03 SMPC Performance Benchmarks
As a platform engineer, I want SMPC performance metrics (latency, resource usage) on realistic blocking key datasets, so that SMPC viability is understood.
- AC: Benchmark: 1M BK-1 comparisons (FI-A vs. FI-B).
- AC: Metrics: wall-clock time, total communication (MB), memory (GB), CPU utilization.
- AC: Target: < 1 second per comparison (1M / 1 sec = 1M/sec throughput).
- AC: Report: if target met, proceed to S-23-04; if not, identify optimization or defer.

### S-23-04 SMPC Integration Tests
As a QA engineer, I want integration tests for SMPC-based cross-FI linking, so that protocol correctness is verified.
- AC: Test: two FIs with overlapping SSNs → SMPC matching finds correct pairs without exposing SSNs.
- AC: Test: false positives avoided (BKs don't match → secure output = 0).
- AC: Test: SMPC server is semi-honest adversary (honest-but-curious); doesn't learn BKs.
- AC: Test coverage: 10K SSNs per FI, 100K matching pairs.

---

## EP-24 — Zero-Knowledge Proofs (ZKP) for Identity Attestation

### S-24-01 ZKP Circuit Design for BK-1/BK-2
As a research engineer, I want ZKP circuits that prove entity ownership without revealing underlying identity data, so that identity attestation is possible without trust.
- AC: Circuit 1: prove P(entity_id) = hash(ssn || salt) WITHOUT revealing ssn.
- AC: Circuit 2: prove owns_account(account_number, routing_number) WITHOUT revealing account details.
- AC: Language: use Circom or similar ZK language.
- AC: Constraint count: < 100,000 to keep proof size manageable.

### S-24-02 ZKP Proof Generation & Verification
As a service engineer, I want APIs to generate and verify ZKP attestations, so that entities can prove identity claims.
- AC: API 1: POST /identity/attest { claim, witness } → { proof_json, proof_size_bytes }.
- AC: API 2: POST /identity/verify { claim, proof } → { valid: true/false }.
- AC: ZKP library: use snarkjs or similar for constraint satisfaction and proof generation.
- AC: Latency: proof generation < 5 seconds, verification < 100ms.
- AC: Proof size: < 1 KB for BK-1/BK-2 circuits.

### S-24-03 ZKP Integration Tests
As a QA engineer, I want integration tests for ZKP-based attestation, so that proof correctness is continuous.
- AC: Test: valid claim + correct witness → proof verifies.
- AC: Test: valid claim + wrong witness → proof fails.
- AC: Test: modified claim after proof generation → proof fails.
- AC: Test: zero-knowledge property: verifier learns nothing except claim validity.

---

## EP-25 — Biometric & Behavioral Signal Integration

### S-25-01 Biometric Signal Capture & Normalization
As a product engineer, I want APIs to capture biometric signals (fingerprint templates, iris scans, voice prints) from FI customer verification workflows, so that biometric becomes a blocking key candidate.
- AC: Supported biometrics: fingerprint (NIST MINEX format), iris (ISO/IEC 19794-6), voice (i-vector embeddings).
- AC: Capture: receive encrypted biometric templates from FI mobile/web apps; store encrypted in database.
- AC: Normalization: each template converted to vector embedding (fixed-size, normalized by biometric type).
- AC: Compliance: BIPA (Biometric Information Privacy Act) and GDPR consent tracking.

### S-25-02 Behavioral Signal Feature Extraction
As a data engineer, I want to extract behavioral features (typing speed variance, mouse movement patterns, device fingerprint, login time-of-day distribution) from authentication logs, so that behavioral biometrics contribute to confidence scores.
- AC: Features: average keystroke latency (ms), mouse speed (px/sec), touch pressure variance, device fingerprint hash, login hour distribution entropy.
- AC: Collection: instrument FI auth systems to send anonymous behavioral telemetry.
- AC: Storage: anonymous behavioral profile linked to idr_customer_id via hashed account_id (no plaintext link).
- AC: Privacy: no raw keystroke sequences or mouse coordinates stored; only aggregated statistics.

### S-25-03 Biometric + Behavioral Blocking Key Design
As a resolution engineer, I want new blocking keys (BK-9, BK-10) based on biometric and behavioral signals, so that these signals can be used for entity matching.
- AC: BK-9: fingerprint template similarity > 0.95 (using NIST MINEX matching).
- AC: BK-10: behavioral profile similarity > 0.90 (using cosine distance on feature vectors).
- AC: Tier assignment: BK-9 / BK-10 match → Tier 3–4 depending on additional signals (not Tier 1, due to biometric spoofing risks).
- AC: Design: research impact of biometric on false-positive rate vs. recall.

### S-25-04 Privacy & Compliance Impact Assessment
As a compliance officer, I want impact assessment for biometric and behavioral signal collection, so that regulatory and ethical risks are understood before deployment.
- AC: BIPA compliance: obtain explicit user consent for biometric collection in each state.
- AC: GDPR compliance: biometric = special category data; requires explicit data processing agreement.
- AC: Ethics: evaluate if behavioral monitoring creates perverse incentives or disparate impact on users of different devices/locations.
- AC: Report: identify mitigations (anonymization, consent, audit trails) before production rollout.

---

## EP-26 — Real-Time Identity Enrichment & Third-Party Data

### S-26-01 Third-Party Data Adapter Framework
As a platform engineer, I want a pluggable framework to integrate third-party data sources (APIs, data feeds, batch uploads), so that new enrichment sources can be added without code changes.
- AC: Adapter interface: `init(config)`, `fetch(entity_id, attributes)`, `enrich(entity_data)`, `get_metadata()`.
- AC: Configuration: adapter_name, endpoint, auth credentials, rate_limit, retry_policy.
- AC: Caching: Redis-backed cache; TTL per adapter (e.g., 30 days for credit bureau, 7 days for address validation).
- AC: Error handling: failed lookup returns NULL enrichment; does not block resolution pipeline.
- AC: Monitoring: latency, error_rate, cache_hit_rate metrics per adapter.

### S-26-02 Credit Bureau Integration (Equifax, Experian, TransUnion)
As a resolution engineer, I want to integrate real-time credit bureau APIs to enrich entity profiles with address history, phone history, and fraud indicators, so that entity resolution is boosted by authoritative data.
- AC: Adapter: Equifax FACT Act API (or equivalent Experian/TransUnion).
- AC: Lookup: query by SSN + name + address; return address_history, phone_history, fraud_indicators.
- AC: Enrichment: add to entity record as attributes (not blocking keys, but confidence boosters for Tier-4).
- AC: Compliance: subject to FCRA (Fair Credit Reporting Act) regulations; audit logging required.
- AC: Cost: manage API call budgets; prioritize lookups for high-value entities (Tier-4 candidates with multiple signals).

### S-26-03 Address Validator Integration (SmartyStreets, UPS, USPS)
As a resolution engineer, I want to validate addresses against USPS and UPS databases to detect moved addresses and correct format issues, so that address-based matching is more reliable.
- AC: Adapter: SmartyStreets Street API or direct USPS Web Tools.
- AC: Lookup: submit address (street, city, state, zip); return USPS standardized address, ZIP+4, delivery point barcode.
- AC: Matching: if entity address doesn't match USPS canonical form, attempt fuzzy match against known address variants.
- AC: Move detection: if USPS shows address as moved, flag for review (possible account takeover).

### S-26-04 Fraud Indicator Integration (SafeGraph, Minfraud)
As a resolution engineer, I want to query fraud indicator databases to identify high-risk addresses or phone numbers, so that fraud signals inform entity tier assignment.
- AC: Adapter: SafeGraph Places API (to identify addresses) + MaxMind minfraud (historical fraud info).
- AC: Lookup: query address or phone against known fraud blacklists.
- AC: Signals: if address is known fraud hotspot → reduce confidence score; if phone is known spammer → additional scrutiny.
- AC: Use: Tier-4 stubs with fraud indicators downgraded to Tier-5; not auto-merged.

### S-26-05 Third-Party Data Caching & SLA Management
As a platform engineer, I want intelligent caching and SLA management for third-party APIs so that rate limits and cost budgets are respected.
- AC: Cache: Redis with per-adapter TTL (e.g., credit bureau 30 days, address validator 7 days).
- AC: SLA: track API budget (calls/day, cost/month); issue alerts if approaching limits.
- AC: Fallback: if third-party API unavailable or budget exhausted, resolution proceeds without enrichment (graceful degradation).
- AC: Monitoring: latency percentiles (P50, P95, P99) for each adapter; alert if P99 > 1 second.

---

## EP-27 — Quantum-Ready Cryptography

### S-27-01 NIST PQC Algorithm Evaluation
As a security engineer, I want to evaluate NIST Post-Quantum Cryptography (PQC) finalists for use in blocking key derivation and audit log integrity, so that JHBI is prepared for quantum computing threats.
- AC: PQC algorithms to evaluate: CRYSTALS-Kyber (lattice-based KEM), CRYSTALS-Dilithium (lattice-based signature), Falcon (lattice-based signature).
- AC: Metrics: key size, signature size, signing time, verification time, security level.
- AC: Comparison: current SHA-256 / HMAC vs. PQC alternatives for determinism, performance.
- AC: Report: recommend algorithm and migration path by end of Q4 2026.

### S-27-02 Lattice-Based Signature POC
As a security engineer, I want to build a POC of lattice-based signatures (Dilithium or Falcon) for audit log integrity, so that future audit logs are post-quantum-secure.
- AC: Replace HMAC-SHA256 signatures on audit log entries with CRYSTALS-Dilithium signatures.
- AC: Implement: generate keypair, sign audit entries, verify signatures.
- AC: Performance: measure signing + verification latency on 10K audit entries.
- AC: Storage: compare signature size (bytes) vs. current HMAC-SHA256.

### S-27-03 Migration Plan: Classical → PQC
As a platform engineer, I want a migration strategy that transitions blocking key derivation and audit logs from classical cryptography to post-quantum cryptography, so that no legacy systems are left vulnerable.
- AC: Phase 1: on all NEW audit log entries, compute both SHA-256 and PQC signatures (hybrid mode).
- AC: Phase 2: deprecate SHA-256-only entries; require PQC signature for all new systems.
- AC: Phase 3: establish sunset date for SHA-256 (2030?); plan offline verification and archival of legacy entries.
- AC: Risk: during hybrid phase, maintain both classical and quantum-safe verification paths.
- AC: Document: publish migration timeline and backward-compatibility guarantees.

---
