# Incident Summary — Book Review Backend

**Name:** Inibehe Emmanuel Sunday
**Cluster:** AKS (book-review-aks2)
**Namespace:** book-review
**Workload:** Deployment backend (label app=backend, Service backend on port 3001)

## 1. Reported Symptom

After the backend image was changed, a new backend Pod would not start. It stayed at 0/1 READY with status ImagePullBackOff, and the rollout did not complete.

## 2. Evidence Collected

Evidence came from the read-only triage script, saved as reports/incident-failure-report.txt:

- Pod status: backend-77b4969d57-zw6c8 was 0/1 with status ImagePullBackOff.
- Readiness: not all matching Pods had ready containers.
- Events: "Failed to pull image inibehe/book-review-backend:does-not-exist ... not found", followed by "Back-off pulling image".
- Service backends: the EndpointSlice still listed the old Pod backend-78c9c7c7f9-scjml as ready=true at 10.224.0.19:3001, because the rolling update keeps the old Pod until the new one is Ready.
- Overall status: FAIL, exit code 2.

## 3. Most Likely Cause

The Deployment pointed to an image tag that does not exist on Docker Hub (inibehe/book-review-backend:does-not-exist). Kubernetes could not pull it, so the new Pod never started. The cause is supported by the image-pull error events and the ImagePullBackOff status.

## 4. Human-Approved Recovery Action

Claude suggested the recovery command as text only. I reviewed it and ran it myself:

    kubectl set image deployment/backend backend=inibehe/book-review-backend:v1 -n book-review
    kubectl rollout status deployment backend -n book-review

This restored the original working image recorded in lab-config.md.

## 5. Verification

I ran the triage script again and saved the result as reports/recovery-report.txt:

- PASS: 5, WARN: 1, FAIL: 0, overall status WARN, exit code 1.
- backend-78c9c7c7f9-scjml is 1/1 Running with 0 restarts.
- No Pod is in ImagePullBackOff or any other failure state.
- The backend Service has a ready endpoint at 10.224.0.19:3001.
- The remaining WARN is from historical Warning events about the deleted Pod backend-77b4969d57-zw6c8. The same events show the failed ReplicaSet scaled down from 1 to 0. Kubernetes keeps events for about one hour, so this warning clears on its own.

## 6. Safety Decision

Claude's role was limited to analysing evidence. CLAUDE.md and SKILL.md forbid apply, delete, patch, edit, scale, set image, rollout restart and any other change to the cluster. Claude gave recovery commands as text only, and I executed them myself after reviewing them. Because instructions alone do not enforce read-only access, the human stays in control of every change.

## 7. Agentic Loop Mapping

- **Gather:** the Bash script (scripts/k8s-triage.sh) ran read-only kubectl checks and saved the report.
- **Analyze:** Claude Code read the report through /k8s-triage, identified the failed checks and the supported cause, and suggested a recovery command.
- **Human Act:** I reviewed the suggestion and ran the recovery command myself.
- **Verify:** I ran the script again; the recovery report showed no FAIL and a ready Service backend.
