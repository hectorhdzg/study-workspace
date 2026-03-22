# Alerting & Incident Response

## SLIs, SLOs, and SLAs

| Term | Definition | Example |
|------|-----------|---------|
| **SLI** (Service Level Indicator) | A measurable metric of service quality | P95 latency, error rate, availability |
| **SLO** (Service Level Objective) | A target value for an SLI | "P95 latency < 200ms", "99.9% availability" |
| **SLA** (Service Level Agreement) | A contract with consequences if SLOs are breached | "99.95% uptime or customer gets credits" |

```
SLI  → What you measure     (e.g., success rate = successful requests / total requests)
SLO  → What you aim for     (e.g., success rate ≥ 99.9% over 30 days)
SLA  → What you promise     (e.g., if below 99.5%, customer gets 10% refund)
```

---

## Error Budget

**Error budget** = how much failure your SLO allows.

```
SLO: 99.9% availability over 30 days
Total minutes in 30 days: 43,200
Error budget: 0.1% × 43,200 = 43.2 minutes of downtime allowed
```

| Availability | Downtime / month | Downtime / year |
|-------------|-----------------|----------------|
| 99% | 7.3 hours | 3.65 days |
| 99.9% | 43 minutes | 8.76 hours |
| 99.95% | 22 minutes | 4.38 hours |
| 99.99% | 4.3 minutes | 52.6 minutes |
| 99.999% | 26 seconds | 5.26 minutes |

**If error budget is exhausted:** Freeze feature releases, focus on reliability.

---

## Alerting Strategy

### Good Alerts

An alert should be **actionable, urgent, and real**.

| Property | Good Alert | Bad Alert |
|----------|-----------|-----------|
| **Actionable** | "Payment error rate > 5% for 5 min" | "CPU is 80%" (so what?) |
| **On symptoms** | "P95 latency > 500ms" | "Pod restarted" (might be normal) |
| **Low noise** | Fires a few times per week | Fires 50 times per day |
| **Contextualized** | Links to dashboard + runbook | Just a metric name |

### Alert on SLOs, Not Resources

```
❌ Bad:  CPU > 80% → page the on-call
         (CPU can spike normally; no user impact)

✅ Good: Error rate > 1% for 5 minutes → page the on-call
         (Users are directly affected)
```

### Alert Severity Levels

| Severity | Action | Example |
|----------|--------|---------|
| **P1 / Critical** | Page immediately, all-hands | Service down, data loss |
| **P2 / High** | Page on-call, respond within 15 min | Error rate spike, partial degradation |
| **P3 / Medium** | Slack notification, respond within hours | Elevated latency, disk 80% |
| **P4 / Low** | Ticket, respond within days | Certificate expiring in 30 days |

---

## Alerting Rules (Prometheus Example)

```yaml
groups:
  - name: api-alerts
    rules:
      # High error rate
      - alert: HighErrorRate
        expr: |
          sum(rate(http_requests_total{status=~"5.."}[5m]))
          / sum(rate(http_requests_total[5m])) > 0.01
        for: 5m
        labels:
          severity: critical
        annotations:
          summary: "Error rate above 1% for 5 minutes"
          runbook: "https://wiki.example.com/runbooks/high-error-rate"

      # Slow responses
      - alert: HighLatency
        expr: |
          histogram_quantile(0.95, sum(rate(http_request_duration_seconds_bucket[5m])) by (le))
          > 0.5
        for: 10m
        labels:
          severity: warning
        annotations:
          summary: "P95 latency above 500ms for 10 minutes"
```

---

## Incident Response Framework

### 1. Detect

Automated alerts or user reports.

### 2. Triage

- **Is it real?** Check dashboards, not just one alert.
- **What's the severity?** How many users are affected?
- **Who owns this?** Route to the right team.

### 3. Mitigate (Stop the Bleeding)

| Action | When |
|--------|------|
| **Rollback deploy** | Issue started after a deployment |
| **Scale up** | Traffic surge causing saturation |
| **Feature flag off** | New feature causing errors |
| **Failover** | Primary service/region is down |
| **Rate limit** | Abusive traffic / DDoS |

**Mitigate first, root-cause later.** Getting users back online > understanding exactly why.

### 4. Resolve

Fix the root cause, verify metrics return to normal.

### 5. Post-Mortem (Blameless)

```markdown
## Incident Post-Mortem: Payment Service Outage (2026-03-15)

**Duration:** 45 minutes (14:15 - 15:00 UTC)
**Impact:** 12% of payment attempts failed
**Severity:** P1

### Timeline
- 14:00 — Deploy v2.3.1 to production
- 14:15 — Error rate alert fires (>5%)
- 14:20 — On-call acknowledges, begins investigation
- 14:30 — Root cause identified: DB migration added NOT NULL column
- 14:35 — Rollback initiated
- 14:45 — Rollback complete, error rate dropping
- 15:00 — Error rate back to baseline, incident resolved

### Root Cause
DB migration added a NOT NULL column without a default value.
Old application code didn't populate this field → INSERT failures.

### Action Items
- [ ] Add migration validation in CI pipeline
- [ ] Add integration test for payment flow against staging DB
- [ ] Improve canary deployment to catch errors before full rollout
```

---

## On-Call Best Practices

- **Rotation:** Weekly rotation, 2-person teams (primary + secondary)
- **Escalation:** If primary doesn't acknowledge in 10 min → secondary, then management
- **Runbooks:** Every alert links to a runbook with diagnostic steps
- **Handoff:** End-of-rotation debrief on any open issues
- **Compensation:** On-call should be compensated (pay or time off)
- **Noise budget:** If on-call is paged > 2 times/night, fix the alerts
