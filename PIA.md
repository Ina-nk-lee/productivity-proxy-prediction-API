# PIA (full)

## Data Inventory

- Inputs: sleep_hours, caffeine_mg, screen_time_hours (+ optional bedtime, notifications).
- Outputs: proxy_productivity_score, uncertainty, risk_factors, suggestion.
- No identifiers or sensitive categories.

## Purpose Limitation

- Sole purpose: predict daily productivity proxy.
- Not for medical, hiring, or grading use. Disclaimer included.

## Minimization & Retention

- Raw inputs: 0d.
- Aggregates: ≤30d.
- Telemetry: latency, error counts only.

## Access Control & Governance

- Access limited to developers via auth.
- No external sharing of telemetry.

## Telemetry Decision Matrix

| Data Type              | Collected? | Granularity  | Retention | Guardrails / Notes              |
| ---------------------- | ---------- | ------------ | --------- | ------------------------------- |
| Input payloads         | ❌         | N/A          | 0 days    | Stateless; no storage           |
| Prediction outputs     | ❌         | N/A          | 0 days    | Returned to client only         |
| Aggregate metrics      | ✔️         | Daily totals | ≤ 30 days | Jitter/noise added; opt-in only |
| Latency / SLA metrics  | ✔️         | p95, avg     | ≤3 0 days | Used for SLA monitoring         |
| Error counts           | ✔️         | Aggregate    | ≤ 30 days | No raw input stored             |
| User identifiers (PII) | ❌         | N/A          | N/A       | Never collected                 |

## Residual Risks

- Misuse as medical tool → mitigated via disclaimers.
- Re-identification via telemetry → mitigated via jitter + k-anon.
- Confounding in screen time → disclosed as noisy.
