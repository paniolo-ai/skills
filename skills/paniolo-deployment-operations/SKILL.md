---
name: paniolo-deployment-operations
description: |
  Cloudflare deployment rollback, health monitoring, and production troubleshooting. Use when recovering from a failed deployment, checking deployment health, or diagnosing production issues.
license: MIT
metadata:
  version: 0.1.0
tags:
- deployment
references: []
---

**Requires:** file-read, terminal (Wrangler/Cloudflare CLI). No network access needed.

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

### Rollback via Git Revert (recommended)

```bash
# Revert the last commit and push — GitHub Actions auto-deploys the reversion
git revert HEAD
git push origin main
# Wait ~2 minutes for deployment
```

### Manual Rollback (faster if CI is slow)

```bash
# Build and deploy the previous commit directly
git checkout main~1
npm run build:all
npm run deploy:production

# Reset HEAD back to current state
git checkout main
```

### API-Only Rollback (if only the Worker broke)

```bash
wrangler rollback --env production
```

### Verify Rollback Succeeded

```bash
npm run health:check
curl https://<your-domain>/api/health
wrangler tail --env production
```

---

## Monitoring & Health Checks

### Health Check Endpoint

```bash
curl https://<your-domain>/api/health
# Expected: { "status": "ok", "environment": "production", "timestamp": "..." }
```

```bash
npm run health:check   # automated — checks API, frontend, DB, realtime
npm run status:deployment
wrangler tail --env production   # live log stream
```

### Cloudflare Analytics

Go to **Analytics → Traffic** in the Cloudflare dashboard to check for 5xx error spikes or response
time regressions after deploy.

---

## Troubleshooting

### Users Still See Old Version After Deploy

```bash
# 1. Try hard refresh in browser (Ctrl+Shift+R) — if that works, it's a cache issue
# 2. Open incognito window — if that works, it's browser cache only

npm run cache:purge         # purge Cloudflare CDN
npm run status:deployment   # confirm correct commit is deployed
```

If still broken after 5 minutes, manually purge in the Cloudflare dashboard: **Caching → Purge Cache
→ Purge Everything**.

### Deployment Hangs or Times Out

```bash
# Reproduce locally
npm run build:all

# Check Node version (must be 20+)
node --version

# Clear build cache and retry
npm run clean
npm run build:all
```

Then check `wrangler tail --env production` for runtime startup errors.

### Environment Variables Not Working

```bash
# List what's deployed
wrangler secret list --env production

# If missing, set and redeploy
wrangler secret put KEY_NAME --env production
npm run deploy
```

---

## Do Not

- Do not attempt root-cause code fixes during an active incident — stabilize first, fix after.
- Do not deploy to production without running the pre-deploy checklist.
- Do not commit secrets to git — use `wrangler secret put` instead.
- Do not use for planning new deployment workflows — load `deployment-strategies` instead.

## References

- Core deploy workflow:
  [deployment-strategies skill](../paniolo-deployment-strategies/SKILL.md)
- Cache + CI/CD: [cloudflare-cache-cicd skill](../paniolo-cloudflare-cache-cicd/SKILL.md)

## Skill Handoffs

- If modifying deployment configuration/workflows, also load `deployment-strategies`.
- If cache invalidation or CI cache behavior is involved, also load `cloudflare-cache-cicd`.