# Identity Resolution System — Database Schemas

PostgreSQL migrations (Flyway format) for the Jack Henry Identity Resolution (IDR) platform.

## Architecture Overview

**Core Principle:** PII stored in **plaintext** and secured via PostgreSQL Row-Level Security (RLS) + OAuth application scopes. No tokenization or hashing of identity fields.

**Key Tables:**
- `idr_account`: Payment accounts (account_number + routing, IBAN, SWIFT BIC)
- `idr_customer`: Entities (individuals, businesses, external stubs)
- `idr_customer_account`: Many-to-many linking (joint accounts, beneficiaries)
- `idr_household`: Entity groupings (Phase 1 basic structure; expansion in Phase 3)

**Blocking Keys:** Derived at runtime from plaintext columns; 8 blocking key types (BK-1 through BK-8):
- **BK-1:** SSN/TIN (plaintext digits)
- **BK-2:** Account + Routing (plaintext concat)
- **BK-3:** Email (plaintext, lowercase)
- **BK-4:** Phone (plaintext, E.164)
- **BK-5:** Core CIF (source_system:source_customer_id)
- **BK-6:** Name + Zip5 (last_name + zip, secondary only)
- **BK-7:** SWIFT BIC + IBAN (international wires)
- **BK-8:** PAN or card network token (Tier 5 stubs only)

---

## Files

### `ep01_entity_data_model.sql` (V1)
**Stories:** S-01-01, S-01-02, S-01-03, S-01-04

Core entity and account tables with temporal columns, RLS policies, and indexes for blocking key lookups.

**Key Entities:**
- `idr_account`: Account records (account_number, routing_number, IBAN, SWIFT BIC)
- `idr_customer`: Customer/member/entity records (ssn, email, phone, address, name)
- `idr_customer_account`: Junction table (role: PRIMARY, JOINT, BENEFICIARY)
- `idr_household`: Household groupings (joint accounts, shared address)
- `idr_household_member`: Membership records (link_type, link_confidence)

**RLS Policies:** All tables enforce `fi_id` isolation (no cross-FI reads).

**Indexes:** Blocking key indexes on (fi_id, bk_column) for O(1) candidate lookups.

---

### `ep03_04_resolution_engine.sql` (V3)
**Stories:** S-03-01 through S-03-06 (Blocking Key Derivation); S-04-01 through S-04-06 (Resolution Engine Core)

Blocking key definitions, tier definitions, match decision log, and manual review queue (Phase 2 deferred).

**Key Tables:**
- `idr_blocking_key_def`: Metadata for 8 blocking keys (BK-1–BK-8)
  - Tier contribution (min_tier_solo)
  - Whether usable as sole key (BK-6 = FALSE)
  - PII class and cross-FI eligibility
- `idr_resolution_tier_def`: Confidence band mapping (Tier 1–5)
  - Tier 1 (DEFINITIVE, 1.0): BK-1 exact match within FI
  - Tier 2 (HIGH, 0.90–0.99): BK-2 or BK-5 match
  - Tier 3 (VERIFIED, 0.85–0.92): BK-3 or BK-4 match
  - Tier 4 (INFERRED, 0.65–0.84): BK-6 secondary or BK-7 (wires)
  - Tier 5 (UNRESOLVED, < 0.65): BK-8 only or no match
- `idr_match_decision_log`: Append-only audit trail (monthly partitions)
  - event_id, matched_bks, tier, confidence_score, action (MERGE/STUB_EXTERNAL/STUB_MINIMAL/ROUTE_TO_REVIEW)
  - Indexed by decision_timestamp and fi_id
- `idr_manual_review_queue` (Phase 2 deferred): Tier 3 conflicts awaiting analyst review

---

### `ep07_global_link_table.sql` (V4)
**Stories:** S-07-01 (idr_global_person_id Link Table Schema)

JHBI-internal cross-FI identity linking (NOT exposed via FI-scoped Query API).

**Key Table:**
- `idr_global_links`: Maps `idr_global_person_id` (UUID) to (fi_id, idr_customer_id) pairs
  - Qualifying signals: BK-1 (SSN) and BK-2 (account+routing) only
  - Email and phone intentionally excluded (by design)
  - Includes conflict detection and audit trail

**Access Control:** JHBI internal service account only; requires `idr:global:read` scope.

---

### `ep08_audit_observability.sql` (V5)
**Stories:** S-08-01, S-08-02 (Phase 2 deferred), S-08-03, S-08-05

Production observability, audit logging, and infrastructure.

**Key Tables:**
- `idr_entity_audit_log`: Append-only immutable audit trail (annual partitions)
  - Records every entity state change (create, merge, update)
  - Retention: 7 years minimum
  - Event types: SEEDED, MERGED, DELETED, UPDATED
- `idr_resolution_metrics_ledger`: Daily metrics snapshot
  - Resolution throughput, tier distribution, pipeline latency
  - Source for Grafana dashboard and SLO tracking
- `idr_load_test_registry`: Load test run results
  - Tracks 10k events/minute throughput test
  - Captures Redis and PostgreSQL metrics

**OpenTelemetry Integration:**
- All tables include `correlation_id` (OpenTelemetry trace_id)
- Structured JSON logging with trace context
- Distributed tracing across full resolution pipeline

---

### `identity_event_schema.json`
**Format:** JSON Schema (Draft 7)

Kafka event schema for `idr.entity.events` topic. Defines IdentityEvent structure:
- Event metadata (event_id, event_type, timestamp, source_product)
- Originator and destination parties (SSN, account, email, phone, address)
- Normalized and derived attributes
- Compliance metadata (source_adapter, trace_id)

**Used by:**
- EP-02 Identifier Normalization Service (produces normalized events)
- All payment adapters (EP-05–15) (produce raw events; normalized by EP-02)
- EP-04 Resolution Engine (consumes normalized events)

---

## Flyway Migration Order

Migrations are applied in order:

1. **V1__entity_data_model.sql** — Core schema (accounts, customers, households)
2. **V2__pci_vault.sql** — (Deferred/optional) PAN tokenization service
3. **V3__resolution_engine.sql** — Blocking keys, tier assignment, match decision log
4. **V4__global_link_table.sql** — Cross-FI global person registry
5. **V5__audit_observability.sql** — Audit log, metrics, infrastructure

---

## Security Considerations

### Row-Level Security (RLS)
All PII tables enforce RLS policies. Examples:
```sql
CREATE POLICY idr_customer_fi_isolation ON idr_customer
    USING (fi_id = current_setting('app.fi_id', true));
```

Applications **must** set `app.fi_id` at session start. Queries without proper RLS context will return 0 rows.

### Plaintext Storage
PII is stored **plaintext** (not encrypted, hashed, or tokenized) because:
1. Blocking key matching requires plaintext for O(1) candidate lookups
2. RLS + OAuth scopes provide tenant isolation and access control
3. Queries need to access raw values for normalization and matching

### Blocking Key Derivation
Blocking keys are derived **at query time** using normalized plaintext values:
```
BK-1 (SSN): SHA-256(normalized_ssn + platform_salt, NOT per-FI)
BK-2 (Account): plaintext account_number || ':' || routing_number
BK-3 (Email): LOWER(email)
BK-4 (Phone): E.164_normalize(phone)
```

---

## Development Notes

### Temporal Queries
Use point-in-time views for historical entity snapshots:
```sql
SELECT * FROM idr_customer_current
WHERE fi_id = 'fi_first_midwest' AND idr_customer_id = '...';

-- Returns only active, non-merged customers
-- (WHERE valid_to IS NULL AND status = 'ACTIVE')
```

### Adding New Migrations
1. Create `VN__description.sql` (N = next version number)
2. Include header comment with EP/story references
3. Add RLS policies to all PII tables
4. Document in this README

### Partitioning Strategy
- `idr_match_decision_log`: Monthly (3-year rolling window)
- `idr_entity_audit_log`: Annual (7-year retention)
- `idr_resolution_metrics_ledger`: No partition (daily summary only)

---

## References

- **Design Document:** [docs/design/JH_Identity_Resolution_Design_v0.4.docx](../docs/design/JH_Identity_Resolution_Design_v0.4.docx)
- **Epics & Stories:** [epics.md](../epics.md), [stories.md](../stories.md)
- **Phase 3 Future:** [epics_phase3.md](../epics_phase3.md), [stories_phase3.md](../stories_phase3.md)
- **Phase 4 Exploratory:** [epics_phase4.md](../epics_phase4.md), [stories_phase4.md](../stories_phase4.md)

---

**Last Updated:** February 24, 2026  
**Version:** Phase 1 (v0.4)
