# CPSC 436C — Draft Architecture & Ethics Guardrails Submission

## 1. Draft Architecture Diagram

```mermaid
flowchart LR
  A[Client App] -->|POST /predict| B[API Gateway]
  B --> C[Compute: AWS Lambda]
  C --> D[(Model Store: S3)]
  C --> E[(Telemetry: DynamoDB)]
  E --> F[Observability: CloudWatch]

  subgraph Guardrails
    G[Retention TTL: raw=0d, agg=30d]
    H[k-anonymity ≥ 10; jitter applied]
    I[Region lock: ca-central-1]
    J[Cost guardrail: stay under free-tier]
  end

  F -.-> Guardrails

Notes
	•	Serverless architecture ensures scalability under student free-tier limits.
	•	Guardrails enforce privacy (TTL, k-anonymity), jurisdiction (region lock), and cost caps (fallback to baseline model).
	•	If free-tier or latency limit exceeded → degrade to rule-based baseline instantly.
```

---

## 2. Clause → Control → Test Snapshot

| Clause                                      | Control                                           | Enforcement Point | Test Signature                                                      |
| ------------------------------------------- | ------------------------------------------------- | ----------------- | ------------------------------------------------------------------- |
| Data must remain in Canada                  | All resources pinned to AWS ca-central-1          | Data              | `def test_data_region(): assert region == "canada"`                 |
| Raw inputs must not be retained             | Stateless API; raw input discarded post-inference | Compute / Data    | `def test_raw_retention(): assert raw_ttl_days == 0`                |
| Aggregates must apply jitter before sharing | Apply Gaussian noise to aggregates                | Observability     | `def test_jitter(): assert noise_std > 0`                           |
| Cost must remain within free-tier           | Automatic downgrade to rule-based baseline        | Compute           | `def test_cost_guardrail(): assert monthly_cost <= free_tier_limit` |

---

## 3. Failing pytest (red bar)

### Example: OCAP Violation Test

```python
def test_data_region():
    region = "us-east-1"  # violation
    assert region == "canada", "OCAP violation: data left jurisdiction"
```

**Fail Output Example**

```
>       assert region == "canada"
E       AssertionError: OCAP violation: data left jurisdiction ('us-east-1')
```

### Example: Retention Violation Test

```python
def test_raw_retention_days():
    raw_retention_days = 7  # violation
    assert raw_retention_days == 0, "Raw data must not be retained"
```

**Fail Output Example**

```
E       AssertionError: Raw data must not be retained (found 7 days)
```

---

## 4. Two Empty Chair Assertions

**Stakeholder 1 — Student User**

- **Assertion:** `assert region == "canada", "OCAP violation: data left jurisdiction"`
- **Rationale:** Ensures user data never leaves Canada, maintaining jurisdictional privacy guarantees.
- **Enforcement:** Data layer — S3 bucket policy, Lambda region restriction.

**Stakeholder 2 — Student Developer**

- **Assertion:** `assert monthly_cost <= free_tier_limit, "Cost guardrail breached"`
- **Rationale:** Keeps API within free-tier limits to remain accessible to student developers.
- **Enforcement:** Observability — Budget alarms, auto-downgrade to rule-based fallback.
