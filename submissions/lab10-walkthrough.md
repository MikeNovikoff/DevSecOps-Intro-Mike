# 5-Minute DevSecOps Program Walkthrough — Juice Shop

## (0:00–0:30) Context

I built an end-to-end DevSecOps program around the Juice Shop application for a course project. The pipeline covers every phase: commit signing, software composition analysis, static and dynamic testing, infrastructure-as-code scanning, container signing, runtime monitoring, and a single vulnerability-management dashboard. DefectDojo ties everything together with a unified SLA matrix.

## (0:30–2:00) Layers

- **Commit** — every commit is SSH-signed, and a `gitleaks` pre-commit hook prevents secrets from entering history.
- **Build** — `syft` generates a CycloneDX SBOM, `grype` scans dependencies for CVEs, and `semgrep` runs SAST.
- **Pre-deploy** — `checkov` and `KICS` scan Terraform, Ansible, and Pulumi; `cosign` signs images; and a Conftest/Rego gate enforces pod security standards in CI.
- **Runtime** — `Falco` with modern eBPF detects container drift, unexpected writes to `/tmp`, and outbound connections to known mining-pool ports.
- **Program** — `DefectDojo` ingests the Grype, Trivy, Semgrep, ZAP, and Nuclei reports from Labs 4–5, deduplicates the same CVE across tools, and tracks remediation against SLAs.

## (2:00–3:00) Findings + Closures

- Five raw scan reports consolidated into 265 findings, deduplicated down to 234 unique items.
- **Strongest correlated finding:** a SQL-injection sink in `routes/login.ts` flagged by both Semgrep (static) and ZAP (dynamic). Static analysis showed where the vulnerability lived; dynamic analysis proved it was reachable.
- No risk-accepted findings yet; any future acceptance will carry an explicit expiry date.

## (3:00–4:00) Metrics

- **MTTD:** near zero days (findings imported immediately after scans).
- **MTTR:** not yet measurable because no findings were closed at initial import.
- **Vulnerability age:** zero days at baseline.
- **SLA compliance:** 100% at baseline.
- **Backlog trend:** 234 active findings, remediation starts next quarter.

## (4:00–4:30) Next Steps

If I had another quarter, I would mature **OWASP SAMM → Defect Management** by ingesting Falco runtime alerts into DefectDojo. This would give runtime detections the same SLA and MTTR tracking as scan-time findings, closing the last gap in the loop.

## (4:30–5:00) Q&A Anticipation

**Q: "How would you handle a Log4Shell-style 0-day?"**

Because every image has a signed CycloneDX SBOM, I would query the SBOM by component name and version to get an exact list of affected services in minutes, then drive remediation through the existing SLA clock. The SBOM turns an incident from a scavenger hunt into a query.

**Q: "Why open-source tools instead of paid IAST or SAST?"**

The open-source stack covers SCA, SAST, IaC, signing, runtime, and aggregation with no license cost and full CI portability. That is the right foundation for establishing program discipline. Paid tools add lower false-positive rates and deeper dataflow, but they make sense after the SLA and MTTR baseline exist, not before.
