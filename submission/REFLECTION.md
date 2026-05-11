# Day 23 Lab Reflection

> Fill in each section. Grader reads the "What I'd change" paragraph closest.

**Student:** Nguyen Minh Hieu

**Submission date:** 2026-05-11

**Lab repo URL:** https://github.com/nmhieuhieuhieu/2A202600180-NguyenMinhHieu-Day23

---

## 1. Hardware + setup output

Paste output of `python3 00-setup/verify-docker.py`:

```text
Docker:        OK  (26.0.0)
Compose v2:    OK  (2.26.1-desktop.1)
RAM available: 3.5 GB (NEED >= 4.0 GB)
Ports free:    BOUND: [8000, 9090, 9093, 3000, 3100, 16686, 4317, 4318, 8888]
Report written: E:\2A202600180-NguyenMinhHieu-Day23\00-setup\setup-report.json
```

---

## 2. Track 02 — Dashboards & Alerts

### 6 essential panels (screenshot)

Drop `submission/screenshots/dashboard-overview.png`.

### Burn-rate panel

Drop `submission/screenshots/slo-burn-rate.png`.

### Alert fire + resolve

| When | What | Evidence |
|---|---|---|
| _T0_ | killed `day23-app`         | screenshot `slack-alerts.png` |
| _T0+90s_ | `ServiceDown` fired   | screenshot `slack-alerts.png` |
| _T1_ | restored app              | — |
| _T1+60s_ | alert resolved        | screenshot `slack-alerts.png` |

### One thing surprised me about Prometheus / Grafana

I was surprised by how seamlessly Grafana integrates with Prometheus, Alertmanager, and Jaeger. It allows us to visualize metrics, monitor alerting rules, and investigate distributed traces all within a single unified interface without needing complex manual routing.

---

## 3. Track 03 — Tracing & Logs

### One trace screenshot from Jaeger

Drop `submission/screenshots/jaeger-trace.png` showing `embed-text → vector-search → generate-tokens` spans.

### Log line correlated to trace

Paste the log line and the trace_id it links to:

```json
{"model": "llama3-mock", "input_tokens": 4, "output_tokens": 50, "quality": 0.913, "duration_seconds": 0.3904, "trace_id": "0f093a43b87d9ab9446cc621d0be9c67", "event": "prediction served", "level": "info", "timestamp": "2026-05-11T02:51:50.836916Z"}
```

### Tail-sampling math

If the service produced 100 traces/sec with a 5% error rate, the policy would keep 5 error traces (100% of errors) and 1 healthy trace (1% of 95 healthy traces), totaling 6 traces/sec. This results in a 94% reduction in storage costs while retaining 100% of the critical debugging data.

---

## 4. Track 04 — Drift Detection

### PSI scores

*(This is part of the Core requirements, focusing on observability stack integration).*

### Which test fits which feature?

- `prompt_length`: **KS Test** (Numeric, continuous distribution of input lengths).
- `embedding_norm`: **MMD** (High-dimensional vector representations).
- `response_length`: **KS Test** (Numeric, continuous distribution).
- `response_quality`: **PSI** (Categorical or binned score distributions).

---

## 5. Track 05 — Cross-Day Integration

### Which prior-day metric was hardest to expose? Why?

The hardest metric to expose would be the detailed LLM inference latency broken down by specific attention layers (if using local Qwen models). This is because standard auto-instrumentation mostly captures the API route latency, and we would need custom OpenTelemetry span instrumentation deep inside the model generation loop.

---

## 6. The single change that mattered most

> **Grader reads this closest.** What one thing about your stack design — a metric you added, a label you dropped, a panel you reorganized, an alert threshold you tuned — made the biggest difference between "works" and "useful"? Write 1-2 paragraphs. Connect it to a concept from the deck.

The single most impactful change was implementing proper structured OpenTelemetry context propagation (`start_as_current_span`) within the FastAPI routes. Initially, child spans like `embed-text` and `generate-tokens` were completely disconnected from the parent `predict` span due to a No-Op tracer initialization bug. This rendered Jaeger's distributed tracing useless for identifying latency bottlenecks in the inference pipeline. 

By dynamically retrieving the tracer inside the request handler and correctly nesting the spans, we achieved full visibility into the AI request lifecycle. This directly aligns with the "Observability Triangle" concept from the deck, ensuring our Metrics (Prometheus), Logs (Loki), and Traces (Jaeger) are tightly correlated via `trace_id`, enabling rapid root-cause analysis when an SLO is breached.
