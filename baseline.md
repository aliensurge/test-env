# Uptycs Cluster Expansion Baseline (BBCloud) — Sizing & Runbook

_Last updated: 2025-09-23 18:05 UTC+08:00_

> **Scope:** This page captures the current working baseline for Uptycs cluster capacity and how we size **incremental expansion**. It is meant to live in GitHub as a simple, copy‑paste‑ready reference and decision log. Figures below are **approximations** and should be used to guide planning, not as hard limits.

---

## 1) TL;DR

- **Current asset count:** ~**200K**  
- **Comfortable upper bound (near‑term):** ~**220–225K** assets (as communicated)  
- **Estimated incremental resources (per +1,000 endpoints):** ~**42.35 vCPUs** and ~**144.22 GiB RAM**  
- **CSPM resources (recently added for upgrade):** **98 vCPUs / 344 GiB** (temporary; to be tuned post‑upgrade)  
- **Recommendation:** Do **not** reduce resources yet; additional endpoints are expected. Re‑evaluate once growth stabilizes.

> **Note:** “Estimated per‑1,000 endpoints” is a planning heuristic provided in discussion and is not a strict extrapolation rule. Use with caution.

---

## 2) What we know (from 2 Jul / follow‑ups)

> **Quoted update** (abridged for this doc):  
> • Resources added for CSPM: **98 vCPUs / 344 GiB**  
> • **Estimated resource requirements for additional endpoints:** ~**42.35 vCPUs** and ~**144.22 GiB** **per 1,000 endpoints** (good approximation; not an exact extrapolation)  
> • **Current asset count:** ~**200K**  
> • **The system should comfortably handle:** ~**220–225K** assets  
> • **Post‑upgrade:** extra resources were added to support the transition; now that’s complete, we’ll scale back a bit to stay within expected limits  
> • **Resource adjustment consideration:** If we maintain ~200K, we could reduce some nodes; however, given expected endpoint growth, the suggestion is to **hold off** for now.

---

## 3) Incremental sizing model (heuristic)

Use the following **planning estimate** for incremental growth:

```
For every +1,000 endpoints  ->  +42.35 vCPUs  &  +144.22 GiB RAM
```

### Example scenarios (incremental to existing footprint)

| Additional endpoints | vCPUs (≈) | RAM GiB (≈) |
|---:|---:|---:|
| +1,000 | 42.35 | 144.22 |
| +5,000 | 211.75 | 721.10 |
| +10,000 | 423.50 | 1,442.20 |
| +25,000 | 1,058.75 | 3,605.50 |
| +50,000 | 2,117.50 | 7,211.00 |
| +100,000 | 4,235.00 | 14,422.00 |

> **Caveats**
> - These are **not** commitments; actual usage depends on workload mix (query concurrency, rule sets, data retention, ingest patterns, CSPM cadence, etc.).  
> - Validate with **observability** (CPU/mem usage, queue/ingest lag, scheduler backlogs) before committing to scale in/out.

---

## 4) Current state & headroom

- **Current assets:** ~**200K**  
- **Comfort zone (communicated):** up to ~**220–225K** assets  
- **Interpretation:** We likely have **~20–25K** assets of headroom _under typical load_ before hitting a comfort threshold—**not** a hard cap.

### If/when adding 20–25K endpoints (illustrative only)

- +**20K** endpoints → ~**847 vCPUs / 2,884.4 GiB** (incremental)  
- +**25K** endpoints → ~**1,058.75 vCPUs / 3,605.5 GiB** (incremental)

> Again, treat as **planning guidance**—validate against real utilization trends.

---

## 5) Post‑upgrade CSPM resources

- **Added for upgrade:** **98 vCPUs / 344 GiB** to support transition.  
- **Now that the upgrade is done:** tune toward steady‑state and right‑size gradually.  
- **Guidance:** Prefer **gradual step‑downs** with watchful monitoring vs. one‑shot reductions.

**Suggested approach**
1. Identify candidate node pools for small step‑down (e.g., ~5–10% compute at a time).  
2. Apply change during **quiet windows** with rollback ready.  
3. **Observe**: CPU/memory, scheduler backlog, query latency, ingest/scan lag, and alert volumes.  
4. If stable for 3–5 business days, consider the next step. If not, revert.

---

## 6) Scale‑in/scale‑out guardrails

- **Hold off on reductions** while we expect endpoint onboarding.  
- Maintain a **buffer** for bursty ingest/scan activity (e.g., patch‑Tuesday, major rollouts).  
- Treat **220–225K** as a **soft comfort ceiling** until confirmed by sustained telemetry.  
- For scale‑out, prefer **pre‑provisioning** before large onboarding waves to avoid backlog.

**Operational signals to watch**
- Sustained **CPU > 70–75%** or **RAM pressure** on critical pools.  
- **Ingest or queue lag** creeping upward across peak windows.  
- **Scheduler backlog** lengthening (jobs waiting, longer latencies).  
- **SLO/SLA symptoms**: delayed detections, slow queries, or throttling indicators.

---

## 7) Decision log

Add entries here as we make changes or confirm limits.

| Date (SGT) | Decision | Rationale | Owner | Next review |
|---|---|---|---|---|
| 2025‑09‑23 | Keep current resources; **no reductions** yet | Additional endpoints expected; avoid oscillation | Platform | 2025‑10‑07 |
| (add) |  |  |  |  |

---

## 8) How to use / update this page

- If planning **endpoint growth**, use the table in §3 to **estimate** incremental vCPU/RAM.  
- Cross‑check against current **utilization dashboards** and recent growth trends.  
- After any scale change, add an entry to the **Decision log** (§7) and monitor for 1–2 weeks.  
- If numbers materially change (e.g., new workload mix, different CSPM cadence), update the **heuristic** and highlight the change in this section.

---

## 9) Assumptions & caveats

- “Per‑1,000 endpoints” numbers are **back‑of‑the‑envelope** estimates from discussion; they’re useful to plan order‑of‑magnitude but **not** binding.  
- Real usage depends on factors like data retention, query concurrency, detection content, schedule windows, and infra heterogeneity.  
- Use **observability** and **canary step‑downs/ups** to validate before committing to larger changes.

---

### Appendix A — Quick math

```
vCPU_needed  ≈  42.35 × (additional_endpoints / 1,000)
RAM_GiB      ≈ 144.22 × (additional_endpoints / 1,000)
```

> Replace `additional_endpoints` with the planned increase amount (e.g., 5,000). This yields the **incremental** resources to budget for.
