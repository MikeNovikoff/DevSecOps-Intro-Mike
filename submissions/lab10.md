# Lab 10 — Vulnerability Management with DefectDojo

## Task 1: DefectDojo Setup + Import

### DefectDojo Version

I deployed DefectDojo Community Edition locally from the official Docker Compose distribution. The instance was reachable at `http://localhost:8080` and the first admin token was retrieved from the `initializer` container logs.

### Product + Engagement

I created one product and one engagement to aggregate all findings from the course labs:

- **Product:** `Juice Shop`
- **Product Type:** `Engineering`
- **Engagement:** `Labs Security Testing`
- **Status:** `In Progress`
- **Target start:** 2026-09-01
- **Target end:** 2026-12-15

I kept the default `Engineering` product type because the course labs are grouped under the same engineering program used for the earlier labs.

### Imported Scan Reports

All reports were imported with the API endpoint `/api/v2/import-scan/` using `auto_create_context=true` so that DefectDojo created the product, engagement, and endpoints on the fly. The import script lives in `labs/lab10/imports/run-imports.sh` and auto-discovers the exact `scan_type` names from the local DefectDojo instance.

| Lab | Tool | File | Import result |
|-----|------|------|---------------|
| 4 | Anchore Grype | `labs/lab4/syft/grype-vuln-results.json` | Success |
| 4 | Aqua Trivy | `labs/lab4/trivy/trivy-vuln-detailed.json` | Success |
| 5 | Semgrep | `labs/lab5/semgrep/semgrep-results.json` | Success |
| 5 | OWASP ZAP | `labs/lab5/zap/zap-report-noauth.json` | Success |
| 5 | Nuclei | `labs/lab5/nuclei/nuclei-results.json` | Success |

The script discovers importer names from `/test_types/` when `jq` is available; otherwise it falls back to the default names `ZAP Scan`, `Semgrep JSON Report`, `Trivy Scan`, and `Grype`.

### Import Verification

After all imports finished, I checked the findings count and severity distribution through the DefectDojo web UI:

- **Total imported findings:** 265
- **After deduplication:** 234 unique findings
- **Active findings:** 234
- **Mitigated findings:** 0
- **Risk-accepted findings:** 0

Severity distribution (from the dashboard):

| Severity | Count |
|----------|------:|
| Critical | 11 |
| High | 98 |
| Medium | 92 |
| Low | 31 |
| Info | 2 |

The lower counts compared to a full production scan reflect the focused lab scope: only five reports were imported (Grype, Trivy, Semgrep, ZAP, Nuclei), rather than every scanner used across all labs.

### Deduplication Example

DefectDojo's deduplication engine collapsed the same vulnerability reported by several scanners into one finding. For example:

- **CVE:** `GHSA-c7hr-j4mj-j2w6` (jsonwebtoken signature verification bypass)
- **Severity:** Critical
- **Sources:** Anchore Grype (from the SBOM), Aqua Trivy (image scan)
- **Result:** one active finding with two `Found By` entries, instead of two separate line items

This is important because without deduplication the backlog would be inflated and SLA tracking would be noisy.

---

## Task 2: Governance Report

### SLA Configuration

I set the following SLA clock at the product level, aligned with common severity-based urgency:

| Severity | SLA |
|----------|----:|
| Critical | 24 hours |
| High | 7 days |
| Medium | 30 days |
| Low | 90 days |

### Executive Summary

The consolidated vulnerability backlog for `Juice Shop` contains 265 imported findings, of which 234 are unique after deduplication. The majority are High and Medium severity, and the most urgent Critical findings are dominated by the `jsonwebtoken` signature-bypass and `lodash` prototype-pollution CVEs flagged in Lab 4. No findings have been remediated yet, so MTTR cannot be computed. The immediate priority is to close the 11 Critical findings within the 24-hour SLA window.

### Findings by Severity (active)

| Severity | Count | SLA compliance |
|----------|------:|---------------|
| Critical | 11 | 100% (no SLA breaches yet) |
| High | 98 | 100% |
| Medium | 92 | 100% |
| Low | 31 | 100% |
| Info | 2 | 100% |

### Findings by Source Tool

| Tool | Active | Mitigated | Notes |
|------|-------:|----------:|-------|
| Anchore Grype | 89 | 0 | Dependency CVEs from the SBOM generated in Lab 4 |
| Aqua Trivy | 71 | 0 | Image and filesystem CVEs from Lab 4 |
| Semgrep | 42 | 0 | SAST findings from Lab 5, mostly injection and hardcoded secrets |
| OWASP ZAP | 22 | 0 | DAST findings from Lab 5, including XSS and authentication flaws |
| Nuclei | 10 | 0 | Network and misconfiguration checks from Lab 5 |

### Program Metrics

| Metric | Value | Comment |
|--------|-------|---------|
| MTTD | 0 days | Scans were imported immediately after collection |
| MTTR | Not available | No findings closed yet |
| Median vulnerability age | 0 days | Initial import baseline |
| Active backlog | 234 | After deduplication |
| SLA compliance | 100% | No SLAs exceeded at baseline |
| Critical findings | 11 | Top remediation target |

### Risk-Accepted Items

At the time of submission, no findings were marked as risk-accepted. I prefer to keep the SLA clock running on all active items until they are either fixed or formally accepted with an expiry date. This prevents backlog rot and forces a periodic review.

| Title | Severity | Risk accepted reason | Expiry date |
|-------|----------|----------------------|-------------|
| — | — | — | — |

### Next-Quarter Goal

For the next quarter, I would mature the **OWASP SAMM Defect Management** practice by wiring Falco runtime alerts (Lab 9) into DefectDojo. This would create a single SLA and MTTR clock for both scan-time and runtime findings, and close the last gap in the security feedback loop.

---

## Bonus: 5-Minute Interview Walkthrough

A full script is available in `submissions/lab10-walkthrough.md`.

- **Practiced runtime:** about 4:45
- **Anticipated Q&A questions covered:** 2 (Log4Shell-style response; open-source vs commercial tools)
- **Strongest one-liner:** "The SBOM turns 'are we affected?' from a week of archaeology into a query."
