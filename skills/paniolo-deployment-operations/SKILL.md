---
name: paniolo-deployment-operations
description: |
  Deployment rollback, health monitoring, and production troubleshooting. Use when recovering from a failed deployment, checking deployment health, or diagnosing production issues.
license: MIT
metadata:
  version: 0.1.0
tags:
- deployment
references: []
---
**Requires:** file-read, terminal (your platform's deploy CLI). No network access needed.

# Deployment Operations Skill

## Use When

Use this skill when:

- Handling production incidents, rollback decisions, or post-deploy checks.
- Diagnosing runtime issues immediately after deployment.

Execution workflow:

1. Prioritize safe recovery (rollback/mitigation) before deep root-cause changes.
2. Validate system health with existing health checks and logs.
3. Document exact operational commands/actions taken.
4. After stabilization, identify follow-up fixes to prevent recurrence.

Output requirements:

- Report incident scope, recovery path used, and current health status.
- Include exact rollback/verification commands used.

## Rollback Procedures

Prefer the option highest in this list that your setup supports. Each is faster
to reason about than the one below it, and none require diagnosing the fault
first — that is the point.

### Rollback via Git Revert (recommended)

Works wherever CI deploys on push, and leaves the revert in history:

```bash
git revert HEAD
git push origin main
```

Then wait for the deployment pipeline to finish before judging the result.

### Platform-Native Rollback (fastest, when available)

Many hosts keep previous deployments addressable and can repoint traffic without
a rebuild. Check whether yours offers one — it is usually the quickest recovery
and does not depend on CI being healthy.

### Manual Rollback (when CI is slow or broken)

Build the previous commit and deploy it directly:

```bash
git checkout main~1
<your build command>
<your deploy command>

git checkout main   # reset your working tree afterwards
```

### Verify Rollback Succeeded

Confirm recovery from outside the deploy tooling — a green pipeline is not
evidence that users are served working code:

```bash
curl https://<your-domain>/api/health
```

Then watch live logs for the first minutes of traffic, and confirm the deployed
revision is the one you intended.

## Monitoring & Health Checks

A health endpoint should report status, environment, and a timestamp, so a
stale-but-healthy response is distinguishable from a fresh one:

```bash
curl https://<your-domain>/api/health
# Expected: { "status": "ok", "environment": "production", "timestamp": "..." }
```

Alongside it, check whatever your platform offers for live log streaming and for
error-rate and latency trends. After any deploy, look specifically for a 5xx
spike or a response-time regression rather than only for total failure.

## Troubleshooting

### Users Still See Old Version After Deploy

Establish where the staleness lives before acting:

1. Hard refresh. If that fixes it, the problem is caching, not the deploy.
2. Open a private window. If that fixes it, it is browser cache only.

If neither does, the CDN or edge cache is still serving the old build — purge it,
then confirm the deployed revision matches the commit you expect. If it persists
several minutes after a purge, purge from the provider's dashboard directly.

### Deployment Hangs or Times Out

Reproduce the build locally first, since a hang in CI and a hang locally have
very different causes:

```bash
<your build command>
```

Check that your runtime version matches what the deploy environment provides —
a mismatch here is a common cause of hangs that only appear remotely. If the
build succeeds locally, clear the build cache and retry, then check runtime
startup errors in the platform's logs.

### Environment Variables or Secrets Not Working

List what the deployed environment actually holds rather than what the config
file claims, set anything missing through your platform's secret mechanism, and
redeploy. Secrets usually apply only to deployments created after they are set.

## Do Not

- Do not attempt root-cause code fixes during an active incident — stabilize first, fix after.
- Do not deploy to production without running the pre-deploy checklist.
- Do not commit secrets to git — use your platform's secret storage.
- Do not use for planning new deployment workflows — load `deployment-strategies` instead.

## References

- Core deploy workflow:
  [deployment-strategies skill](../paniolo-deployment-strategies/SKILL.md)
- Cache + CI/CD: [cloudflare-cache-cicd skill](../paniolo-cloudflare-cache-cicd/SKILL.md)

## Skill Handoffs

- If modifying deployment configuration/workflows, also load `deployment-strategies`.
- If cache invalidation or CI cache behavior is involved, also load `cloudflare-cache-cicd`.
