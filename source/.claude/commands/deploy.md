---
description: Complete the PR merge, trigger the CI/CD pipeline, and monitor deployment.
allowed-tools: mcp__azure-devops__pull_requests_update, mcp__azure-devops__pull_requests_get, mcp__azure-devops__builds_queue, mcp__azure-devops__builds_get, mcp__azure-devops__work_items_update, Bash(az *), Bash(kubectl *), Bash(curl *)
---

You are a senior DevOps engineer and site reliability engineer. Your goal is a safe, verified deployment. Never skip a verification step. If anything is red, diagnose before proceeding — a bad deployment is worse than a delayed one.

## Step 1 — Pre-flight checks

1. Run `git branch --show-current` and extract STORY_ID.
2. Fetch PR details via `mcp__azure-devops__pull_requests_get`.
3. Confirm the PR is in `Approved` state with zero active BLOCKERs.
   - If not Approved: STOP.
     > "PR must be in Approved state before deploying. Run /review-pr."
4. Confirm all required status checks (CI gates) on the PR have passed.
   - If any check is failing: STOP.
     > "Required check '{check name}' is failing. Fix it before merging."

## Step 2 — Merge the PR

Use `mcp__azure-devops__pull_requests_update` with:
- Merge strategy: **Squash**
- Delete source branch after merge: **true**

If merge fails (conflict, policy violation, or other error), STOP:
> "Merge failed: {error message}. Resolve the issue manually and re-run /deploy."

## Step 3 — Monitor the CI/CD pipeline

The merge to main triggers the pipeline automatically.
Poll `mcp__azure-devops__builds_get` every 30 seconds.
**Timeout: 20 minutes.** If the pipeline has not completed within 20 minutes, STOP:
> "Pipeline did not complete within 20 minutes. Check manually: {pipeline URL}"

Print a live status table. Update it as each stage transitions:

| Stage       | Status |
|-------------|--------|
| lint        | ⏳ / ✅ / ❌ |
| unit-tests  | ⏳ / ✅ / ❌ |
| build       | ⏳ / ✅ / ❌ |
| docker      | ⏳ / ✅ / ❌ |
| trivy       | ⏳ / ✅ / ❌ |
| push-acr    | ⏳ / ✅ / ❌ |
| deploy-aks  | ⏳ / ✅ / ❌ |

If **any stage fails**, STOP immediately:
> "❌ Pipeline stage '{stage}' failed.
> Failure summary: {brief error from build log}
> Build log: {URL}
> Fix the issue and re-trigger the pipeline before re-running /deploy."

Do NOT proceed to AKS verification if the pipeline did not succeed.

## Step 4 — Verify AKS deployment

```
az aks get-credentials \
  --resource-group semanticlexiforge_rg \
  --name semanticlexiforge_kub_cluster \
  --overwrite-existing

kubectl rollout status deployment/backend  -n app-dev --timeout=5m
kubectl rollout status deployment/frontend -n app-dev --timeout=5m
kubectl get pods -n app-dev
```

Inspect the pod list:
- All pods must show `STATUS = Running` and `RESTARTS` ≤ 2
- If any pod is in `CrashLoopBackOff`, `Error`, `OOMKilled`, or stuck in `Pending`:
  ```
  kubectl logs {pod-name} -n app-dev --tail=100
  kubectl describe pod {pod-name} -n app-dev
  ```
  Print the last 100 log lines and the describe output, then STOP:
  > "Pod {pod-name} is unhealthy ({status}). Review the logs above and fix before
  > marking the story done."

## Step 5 — Smoke test

```
curl -sf --max-time 10 https://semanticlexiforge/actuator/health
```

- Response must contain `"status":"UP"`
- HTTP status must be 200
- If the call fails, times out, or returns non-200: STOP:
  > "Health check failed (HTTP {code} or timeout). Deployment may be misconfigured.
  > Check ingress, service endpoints, and application logs."

Print: `"✅ Health check passed — service is UP"`

## Step 6 — Close the story

1. Set State = `Done` on work item US-{STORY_ID} via MCP.
2. Add a comment to the work item:
   > "Deployed to AKS `app-dev` namespace at {UTC timestamp}.
   > Pipeline run: {build-url}. Health endpoint: UP."

## Step 7 — Final confirmation

```
✅ US-{STORY_ID} is DONE

Branch merged (squash) → Pipeline green → Pods running → Health check UP → Story closed.

Run /pick-story to start the next story.
```
