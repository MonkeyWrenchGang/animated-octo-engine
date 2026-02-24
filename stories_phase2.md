# Phase 2 Stories

Stories deferred to Phase 2 implementation.

---

## EP-04 — Resolution Engine Core

### S-04-05 Manual Review Queue (Tier 3 Conflicts)
As a resolution analyst, I want conflicting Tier-3 matches routed to a review queue, so that ambiguous identity cases are resolved by a human rather than an automated merge that could corrupt entity data.
- AC: Review queue implemented as a table with status (PENDING, RESOLVED, REJECTED).
- AC: Events routed to queue include: event_id, candidate_entity_ids, matched_bks, conflict_reason.
- AC: API endpoint to list pending reviews, accept merge, or reject with reason.
- AC: Accepted merges proceed as per S-04-03; rejected cases create new stub.
- AC: Queue age alert fires if items remain PENDING > 5 business days.

---

## EP-06 — FI-Scoped Query API

### S-06-04 API Rate Limiting & Auth Enforcement
As a platform operator, I want rate limiting and OAuth enforcement on the API, so that no FI can overwhelm the system and cross-tenant access is impossible.
- AC: OAuth 2.0 client credentials flow; fi_id extracted from JWT sub claim.
- AC: Rate limit: 1,000 requests/minute per fi_id (configurable).
- AC: Rate limit exceeded returns HTTP 429 with Retry-After header.
- AC: All requests logged with: fi_id, endpoint, latency, status code.
- AC: API latency SLO: P99 < 200ms for entity-by-ID lookups.

---

## EP-08 — Observability, Audit Logging & Infrastructure

### S-08-02 Resolution Pipeline Metrics Dashboard
As a platform engineer, I want a metrics dashboard showing resolution throughput, tier distribution, and pipeline latency, so that I can confirm the system meets its SLOs and detect anomalies.
- AC: Metrics: events_per_second, resolution_tier_distribution (T1–T5 %), pipeline_latency_p50/p95/p99, redis_lookup_latency, entity_write_latency.
- AC: Dashboard published in Grafana with 1-minute refresh.
- AC: Alert: p99 pipeline latency > 500ms for > 5 minutes.
- AC: Alert: Tier-5 (UNRESOLVED) rate > 30% over a 1-hour window.
- AC: Kafka consumer lag monitored per topic partition.
