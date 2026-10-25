# Project: AI-Assisted Kubernetes Incident Triage

## Application configuration
Replace every placeholder using discovery from your own Book Review workload.
- Namespace: book-review
- Deployment: backend
- Container: backend
- Pod selector: app=backend
- Service: backend
- Original working image: inibehe/book-review-backend:v1

Keep these values consistent with lab-config.md and scripts/k8s-triage.sh.

## Workflow
Gather → Analyze → Human Act → Verify.

The script collects evidence; Claude analyzes it; the human executes changes.
Use only a personal lab cluster.

## Five health checks
1. Pod status and waiting reasons.
2. Pod readiness.
3. Container restart counts (five or more produces WARN).
4. Namespace Warning events, including historical or unrelated events.
5. Ready Service backends using EndpointSlices, with Endpoints fallback.

## Safety rules
- Never change Kubernetes resources: no apply, delete, patch, edit, scale,
  set image, rollout restart, or other mutating operation.
- Provide recovery commands as text only; the human reviews and executes them.
- Use the Bash report as primary evidence. Do not assert an unsupported cause.
- Missing Pods or failed required queries must not be reported as healthy.
- Logs may be unavailable when a container never started.
- Never display credentials, kubeconfig contents, tokens, or certificates.
- Instructions are not an enforced access boundary. Do not request broad
  permissions or approval for cluster-changing commands.

## Local files and reports
- The supplied script may create reports/ and overwrite k8s-health-report.txt.
- During planning, do not create or edit files.
- During /k8s-triage, do not modify other files.
- Preserve failure and recovery reports through the human's terminal commands.
- Reports include full name, UTC timestamp, configuration, evidence,
  PASS/WARN/FAIL results, overall status, and exit code.
- Exit codes: 0 HEALTHY, 1 WARN, 2 FAIL.
- HEALTHY needs no recovery. WARN needs evidence review, not automatic changes.
- Recovery requires no FAIL; historical warnings may remain.
