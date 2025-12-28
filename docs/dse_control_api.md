# DSE Control API Specification
Version: 0.1.0 (Draft)
Status: Internal / Pre‑Release
Owner: QuantFold Stabilizer Architecture Group

1. Overview
The Deterministic Stabilizer Engine (DSE) is the coordination and integrity layer of the QuantFold compute marketplace. It ensures that:
- Jobs are assigned deterministically
- Provers report results consistently
- Scorecards evolve under a stable, auditable EMA law
- The system remains predictable under concurrency
- All components communicate through a well‑defined, versioned API

This document defines the Control API, the contract between:
- Scheduler
- Persistor Service
- Backend Manager
- Prover Nodes
- Observability Layer
- Future ZK‑Rollup settlement logic

The API is designed for long‑term stability, auditability, and permissionless decentralization.

2. Transport Layer

2.1 Primary Transport
- gRPC (recommended)
- HTTP/2
- TLS 1.3 required
- Idle timeout: 30s
- Max request size: 10 MB

2.2 Event Transport
- Kafka (or compatible MQ)
- JSON or Protobuf payloads
- At‑least‑once delivery semantics
- Partitioning:
  - prover_id for score updates
  - network_job_id for job audits

2.3 Authentication
- mTLS for internal components
- JWT or API keys for external provers
- Key rotation supported
- Revocation via CRL or JWKS

3. Core Concepts

3.1 Job
A unit of work submitted by a client and executed by a prover.

3.2 Prover
A backend node capable of performing compute tasks.

3.3 Scorecard
A reputation and performance metric updated via EMA.

3.4 Stabilizer State Machine
PENDING -> ASSIGNED -> RUNNING -> COMPLETED -> AUDITED

3.5 Versioning
All mutable state uses optimistic locking with a version counter.

4. API Methods
Each method includes:
- Endpoint
- Request fields
- Response fields
- Error conditions

4.1 Submit Job
POST /v1/jobs/submit

Request:
- client_job_id (string, optional)
- payload (bytes or opaque JSON)
- max_cost_micros (int64)
- priority (enum: LOW | NORMAL | HIGH)

Response:
- network_job_id (int64)
- status = "PENDING"

Errors:
- DSE‑001 Invalid request
- DSE‑007 Internal error

4.2 Cancel Job
POST /v1/jobs/{network_job_id}/cancel

Response:
- status = "CANCELLED"

Errors:
- DSE‑003 Job not found
- DSE‑004 Prover not found

4.3 Query Job Status
GET /v1/jobs/{network_job_id}

Response:
- status (string)
- assigned_prover (string or null)
- timestamps (object)
- final_cost_micros (int64 or null)

Errors:
- DSE‑003 Job not found

4.4 Query Prover State
GET /v1/provers/{prover_id}

Response:
- score_value (float)
- successful_jobs (int64)
- failed_jobs (int64)
- version (int64)
- last_heartbeat (timestamp)

Errors:
- DSE‑004 Prover not found

4.5 Report Job Result
POST /v1/jobs/{network_job_id}/result

Request:
- success (bool)
- final_cost_micros (int64)
- error_message (string, optional)

Response:
- status = "COMPLETED"

Errors:
- DSE‑003 Job not found

4.6 Heartbeat
POST /v1/provers/{prover_id}/heartbeat

Request:
hardware_profile (object):
  - cpu_cores (int)
  - cpu_model (string)
  - gpu_count (int)
  - gpu_model (string)
  - gpu_memory_gb (int)
  - ram_gb (int)
  - storage_type (string)
  - storage_iops (int)
  - network_bandwidth_gbps (float)

load_factor (float)

capabilities (object):
  - supports_fp32 (bool)
  - supports_fp16 (bool)
  - supports_int8 (bool)
  - max_batch_size (int)
  - supported_kernels (array of strings)
  - zk_acceleration (object):
      - groth16 (bool)
      - plonk (bool)
      - stark (bool)

Response:
- accepted (bool)

4.7 Capability Discovery
GET /v1/provers/{prover_id}/capabilities

Response:
- hardware_profile
- supported_features

5. Event Model
Events are emitted by the Scheduler and consumed by the Persistor Service.

5.1 ScorecardUpdateEvent
{
  "prover_id": "string",
  "delta_success": 1,
  "delta_failure": 0,
  "new_score_value": 93.2,
  "expected_version": 7,
  "event_id": "uuid",
  "emitted_at": "2025-12-28T03:50:00Z"
}

5.2 JobAuditEvent
{
  "network_job_id": 12345,
  "timestamp": "2025-12-28T03:50:00Z",
  "client_job_id": "abc-123",
  "prover_id": "prover-42",
  "status": 1,
  "final_cost_micros": 300123,
  "error_message": null,
  "event_id": "uuid"
}

6. Concurrency & Ordering Guarantees

6.1 Scorecard Updates
- Optimistic locking enforced via version
- Persistor retries up to N times
- EMA recomputed on conflict

6.2 Job Audits
- Append‑only
- Idempotent via event_id

6.3 Event Ordering
- Kafka partitioned by:
  - prover_id for score updates
  - network_job_id for job audits

7. Security Model

7.1 Authentication
- mTLS for internal components
- JWT or API keys for provers

7.2 Authorization
- Scheduler: full control
- Persistor: write‑only
- Prover: job execution only

7.3 Abuse Prevention
- Rate limiting
- Cost ceilings
- Slashing (future)

7.4 Audit Logging
- All control actions logged
- Immutable append‑only logs

8. Error Codes
DSE‑001 Invalid request
DSE‑002 Unauthorized
DSE‑003 Job not found
DSE‑004 Prover not found
DSE‑005 Version conflict
DSE‑006 MQ delivery failure
DSE‑007 Internal error

9. Compatibility & Migration

9.1 Versioning
- Semantic versioning
- Protocol version embedded in headers

9.2 Deprecation Policy
- 2‑version grace period
- Warnings emitted via logs + metrics

9.3 Backward Compatibility
- New fields must be optional
- Breaking changes require a new major version

10. Reference Examples

10.1 Job Submission Example
{
  "client_job_id": "abc-123",
  "payload": "...",
  "max_cost_micros": 500000,
  "priority": "HIGH"
}

10.2 Score Update Event Example
{
  "prover_id": "prover-42",
  "delta_success": 1,
  "delta_failure": 0,
  "new_score_value": 93.2,
  "expected_version": 7,
  "event_id": "uuid",
  "emitted_at": "2025-12-28T03:50:00Z"
}
