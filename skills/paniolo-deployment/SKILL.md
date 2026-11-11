---
name: paniolo-deployment
description: |
  Core deployment workflow, environment configuration, secrets management, health monitoring, rollbacks, and incident triage. Use when deploying code, configuring environments, rolling back failed deploys, or verifying deployment health.
license: MIT
metadata:
  version: 0.2.0
tags:
- deployment
- operations
references: []
---

**Requires:** file-read, terminal (platform deploy CLI or curl). No network access needed for dry runs.

# Deployment and Operations

## Preconditions

- Identify target environment: local, staging, or production.
- Confirm whether changes touch static assets, API handlers, or database migrations.
- Verify environment configuration and secrets exist before running deployment commands.

## Pre-Deploy Quality Checklist

Always run the full test suite and static checks before deploying to staging or production:

```bash
<your lint command>
<your unit-test command>
```

Whenever possible, deploy to a staging environment first to validate build artifacts and migrations against real infrastructure.

## Deployment Workflow & Cache Invalidation

Deploy commands vary in how they treat edge networks and content delivery caches:

- **Plain deploy**: Deploys updated application bundles but may leave CDN or edge caches serving stale builds. Suitable only for internal, non-cached endpoints.
- **Deploy with cache purge**: Required for any user-facing UI or client-consumed API change. Purges edge caches so users immediately receive new code.
- **Database migrations**: Run migrations before deploying code that depends on new schema fields, using backward-compatible migration patterns.

## Environment Configuration & Secrets Management

- **One variable per environment**: Do not branch on runtime environment names within application logic. Set distinct variables per environment:
  ```env
  API_URL=http://localhost:3000       # Development
  API_URL=https://staging.example.com # Staging
  API_URL=https://api.example.com     # Production
  ```
- **Public configuration vs secrets**: Non-sensitive settings (regions, environment tags, public endpoints) belong in version-controlled config. Sensitive credentials (tokens, keys, database passwords) belong exclusively in the platform's secret vault.
- **Never commit secrets to git**: Secrets belong in gitignored `.env` files or platform secret storage.
- **Redeploy after secret rotation**: In most platforms, secrets apply only to deployments created after the secret was set or updated.

## Post-Deploy Health Verification & Monitoring

Confirm health independently of deploy tooling outputs:

```bash
curl https://<your-domain>/api/health
```

Health endpoints should return status, environment name, and an ISO timestamp so stale cached responses can be identified. Following deployment:

1. Tail live log streams for runtime startup errors or uncaught exceptions.
2. Monitor 5xx error rates and p95/p99 latency trends.
3. Test key user journeys in a private/incognito window to bypass local browser caches.

## Incident Recovery & Rollback Procedures

When a production defect occurs, prioritize stabilization over root-cause investigation:

### 1. Rollback via Git Revert (Recommended)

When deployments are automated via CI/CD, creating a revert commit preserves full audit history:

```bash
git revert HEAD
git push origin main
```

### 2. Platform-Native Rollback (Fastest)

Many hosting platforms (e.g. Cloudflare Pages/Workers, Vercel, AWS ECS) support instant rollback to a prior deployment ID without rebuilding. Use this when seconds count.

### 3. Manual Rollback (CI Outages)

If CI/CD is broken or degraded, check out the previous commit, build locally, and deploy directly:

```bash
git checkout main~1
<your build command>
<your deploy command>
git checkout main
```

## Troubleshooting Common Failures

- **Users see stale build after deploy**:
  1. Hard refresh browser (`Ctrl+F5` / `Cmd+Shift+R`).
  2. Test in an incognito window.
  3. If staleness persists across devices, purge CDN edge cache and verify deployed commit SHA.
- **Deployment hangs or times out in CI**:
  1. Reproduce build locally to isolate environment differences.
  2. Verify Node.js/Rust/runtime version matches between local and CI runner.
  3. Clear runner build cache and retry.
- **Missing or invalid secrets**:
  1. Inspect the platform's active secrets list rather than local files.
  2. Re-inject missing keys and trigger a fresh deployment.

## Evaluations (I/O Examples)

**Input:** "Prepare and deploy a hotfix to production"
**Expected:** Agent runs lint and unit tests, uses deploy command with cache purge, verifies health endpoint with curl, tails logs, and reports exact commit SHA deployed.

**Input:** "Production is returning 500 errors after our last deploy"
**Expected:** Agent advises immediate rollback (Git revert or platform rollback), checks health status, and postpones root-cause code fixes until system is stable.

## Do Not

- Do not deploy to production without passing lint and unit tests.
- Do not commit secrets, tokens, or credential files to git.
- Do not use plain deploy for user-facing changes when cache invalidation is required.
- Do not attempt deep code edits during an active incident — roll back first, diagnose after.

## References

- [paniolo-secrets](../paniolo-secrets/SKILL.md)
- [paniolo-safety-and-git-best-practices](../paniolo-safety-and-git-best-practices/SKILL.md)

---

*This skill is brought to you by [Paniolo.ai](https://paniolo.ai).*
