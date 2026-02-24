# Phase 3 Stories (Deferred)

Detailed user stories for Phase 3 epics. Deferred until Phase 1 provides production data and Phase 2 infrastructure is stable.

---

## EP-16 — Household Formation & Entity Linking

### S-16-01 Household Schema & Temporal Tracking
As a JHBI architect, I want a household table with temporal tracking (valid_from, valid_to), so that household membership changes over time are auditable and reversible.
- AC: household table: id, name, created_at, updated_at; foreign key to households (households.id).
- AC: household_members junction table: household_id, idr_customer_id, relationship_type, effective_date, end_date.
- AC: Relationship types: SPOUSE, CHILD, PARENT, CO_OWNER, OTHER.
- AC: RLS policy: FI staff can only access households for their own FI entities.
- AC: Audit log captures every household membership change.

### S-16-02 Household Merge Rules (by address, surname, joint accounts)
As a resolution engineer, I want automatic household merge logic based on shared address + surname or joint account signals, so that family relationships are discovered without manual intervention.
- AC: Rule 1: Same address + same last name → create HOUSEHOLD with high confidence.
- AC: Rule 2: Joint account holders → add to existing household or create if both unlinked.
- AC: Rule 3: Temporal constraint: merge only if address overlap within 90 days.
- AC: Household merge is promotable: if merged members later have SSN/BK-1 match, household link is strengthened.
- AC: Merge decision logged with audit trail: rule_triggered, confidence_score, actor.

### S-16-03 Household API (list members, add/remove, history)
As a bank CSR, I want an FI-scoped REST API to list household members, add a new member, remove a member, and view household history, so that I can manage customer household groupings.
- AC: GET /v1/entities/{id}/household → list all household members.
- AC: POST /v1/household/{id}/members → add member with relationship_type; requires CSR account with idr:household:write scope.
- AC: DELETE /v1/household/{id}/members/{member_id} → remove member; soft-delete (end_date set); not hard-deleted.
- AC: GET /v1/household/{id}/history → return temporal audit of all membership changes.
- AC: All endpoints return 401/403 if fi_id mismatch or no scope.

### S-16-04 Household Integration Tests
As a QA engineer, I want household formation integration tests so that household logic regressions are caught.
- AC: Test: same address + surname → household auto-created.
- AC: Test: joint account detected → members linked into household.
- AC: Test: household member added via API → history recorded.
- AC: Test: household member removed → soft-delete (end_date set), not hard-delete.
- AC: Test: CSR without idr:household:write scope cannot modify household.

---

## EP-17 — Probabilistic Scorer & Weak Signal ML

### S-17-01 Feature Engineering (name_similarity, address_distance, velocity)
As a data engineer, I want feature extraction for weak signals (Levenshtein distance on name, geographic distance between addresses, transaction frequency velocity), so that ML models have rich signals to learn from.
- AC: Feature 1: Name similarity using Levenshtein distance with 0.7 threshold (string distance / max_length).
- AC: Feature 2: Address distance using zip code centroids; compute haversine distance (km).
- AC: Feature 3: Velocity: days between last transaction sender A and first transaction recipient B at same merchant.
- AC: Features computed offline in a Spark/Python job; written to feature store (Parquet) keyed by (entity_id_1, entity_id_2).
- AC: Feature schema versioned; old features never deleted, new versions appended.

### S-17-02 ML Pipeline (training, offline inference on stubs)
As a data scientist, I want a training pipeline that learns from Phase 1 correct merges and rejected stubs, so that a probabilistic model can score weak Tier-4 matches.
- AC: Training data source: idr_match_decision_log where action = MERGE or ROUTE_TO_REVIEW → labeled as correct or incorrect.
- AC: Model: gradient boosted trees (XGBoost) with binary classification (match / no-match).
- AC: Output: P(match | features) for each (entity_1, entity_2, feature_vector).
- AC: Model versioned and exported to ONNX format.
- AC: Inference job runs offline daily: score all Tier-4 EXTERNAL_STUBs with P >= 0.7.

### S-17-03 Weak Signal Scoring API (confidence boost for Tier-4)
As a resolution engine, I want the probabilistic scorer to boost Tier-4 (INFERRED) matches toward Tier-3 confidence if weak signals meet thresholds, so that ignored stubs can be revisited.
- AC: API: POST /scoring/score_weak_match { entity_id_1, entity_id_2, features } → { p_match, recommended_tier }.
- AC: If p_match >= 0.8 AND name_similarity >= 0.7 AND address_distance <= 5km → recommend Tier-3 (VERIFIED).
- AC: Logging: all scoring calls logged for audit and model improvement feedback.
- AC: Latency SLO: < 100ms per request.

### S-17-04 Probabilistic Scorer Integration Tests
As a QA engineer, I want integration tests for ML-based scoring logic so that ML model changes don't regress resolution quality.
- AC: Test: Tier-4 stub with high name similarity + close address + similar velocity → score >= 0.8 → recommend Tier-3.
- AC: Test: Tier-4 stub with no name match + far address → score <= 0.3 → remain Tier-4.
- AC: Test: Model version update does not break inference API.

### S-17-05 Model Evaluation & SLO Metrics
As a platform engineer, I want model evaluation metrics (precision, recall, F1) tracked in Grafana and SLO alerts if model performance degrades, so that ML failures are caught quickly.
- AC: Metrics computed post-deployment on holdout test set.
- AC: Precision: of Tier-4 stubs boosted to Tier-3 by model, % that are true positives (later confirmed).
- AC: Recall: of all true matches in Tier-4 stubs, % discovered by model.
- AC: F1 = 2 * (precision * recall) / (precision + recall).
- AC: SLO alert: F1 < 0.75 over 7-day window → page on-call.

---

## EP-18 — Payment Network Discovery & Pattern Analysis

### S-18-01 Payment Network Model (directed graph, transaction volume, frequency)
As a data engineer, I want to build a directed payment network graph where nodes are entities and edges represent payment flows, so that network analysis algorithms can discover patterns.
- AC: Graph schema: entity_id_src → entity_id_dst with attributes (total_volume, transaction_count, last_date, first_date).
- AC: Graph constructed daily from payment events in Kafka: extract (originator, destination) pairs across all adapters (ACH, RTP, Wires, etc.).
- AC: Graph stored in PostgreSQL using adjacency list; exported to GraphML format for offline analysis.
- AC: Temporal: maintain 90-day network (consider only recent flows).
- AC: Scale: handle 10M entities, 100M edges.

### S-18-02 Network Clustering & Community Detection
As a data scientist, I want to run community detection (Louvain algorithm) on the payment network to identify business ecosystem clusters (e.g., all suppliers of a single manufacturer), so that we can infer group relationships.
- AC: Use Louvain algorithm or similar modularity-based clustering.
- AC: Community label = set of entities that frequently transact together.
- AC: Communities detected daily; membership stable (merge/split on topology change).
- AC: Export community assignments to idr_entity_community table.
- AC: No PII revealed in community output; only anonymized community_id.

### S-18-03 Privacy-Preserving Analytics API (anonymized network patterns)
As a data analyst at an FI, I want analytics on payment network patterns without seeing individual transaction details or routes, so that I can understand customer business ecosystems while respecting privacy.
- AC: API: GET /analytics/network/communities → list community_ids and member counts (no entity_ids).
- AC: API: GET /analytics/network/hubs → list highest-degree entities (senders, receivers) but anonymized (entity_id hashed per FI, not exposed).
- AC: API: GET /analytics/network/anomalies → list entities with unusual velocity changes (no PII).
- AC: All outputs approved by compliance team before release.

### S-18-04 Network Integration Tests
As a QA engineer, I want integration tests for network analysis so that graph algorithms correctly identify communities.
- AC: Test: synthetic network with 2 dense clusters and bridge node → Louvain finds 2 communities.
- AC: Test: daily network recomputation preserves stable community assignments.
- AC: Test: API results are anonymized (no entity_ids leaked).

---

