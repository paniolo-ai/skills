---
name: paniolo-deployment-strategies
description: |
  Core deployment workflow, environment variable configuration, and secrets management. Use when deploying code, configuring environments, or managing secrets.
license: MIT
metadata:
  version: 0.1.0
tags:
- deployment
references: []
---
**Requires:** file-read, terminal (your platform's deploy CLI). No network access needed.

# Deployment Strategies Skill

## Use When

Use this skill when:

- Preparing or executing staging/production deployments.
- Updating environment configuration or secrets management steps.

Execution workflow:

1. Run pre-deploy quality checks before deployment changes.
2. Use the appropriate deploy command for the change type (including cache purge when needed).
3. Keep env/secrets handling aligned with existing project conventions.
4. Verify deployment health immediately after rollout.

Output requirements:

- Summarize deploy/config changes and affected environments.
- Include exact deploy and verification commands used.

## Common Scenarios

- Deploying a frontend, an API service, or both
- Configuring environment variables for staging or production
- Adding new secrets or credentials
- Running pre-deploy validation

## Deploy Workflow

### Pre-Deploy Checklist

Always run before deploying to production:

```bash
<your lint command>
<your unit-test command>
```

### Deploy Commands

Most projects expose more than one deploy path, and the difference between them
matters more than the names:

- **A plain deploy** ships new code but may leave a CDN or edge cache serving the
  previous build to users.
- **A deploy that also purges cache** is the one to use for any user-facing
  change — UI components, API behavior, bug fixes — so users see the new code
  immediately rather than a cached version.
- **A staging deploy** targets the pre-production environment. Deploy there
  first whenever the change is not trivially reversible.

Learn which of your project's scripts does which before the first production
deploy, not during one. A deploy that silently skips cache invalidation looks
identical to a successful one right up until users report stale behavior.

### Verify After Deploy

Confirm health from outside the deploy tooling — a green pipeline is not
evidence that users are served working code:

```bash
curl https://<your-domain>/api/health
```

Then tail live logs for the first minutes of traffic.

## Environment Configuration

Keep one variable per environment rather than branching on environment inside
code, so a misconfiguration fails visibly at startup instead of silently
selecting the wrong backend:

```env
API_URL=http://localhost:3000       # Development
API_URL=https://staging.example.com # Staging
API_URL=https://api.example.com     # Production
```

Non-secret configuration belongs in your platform's config file, committed to
the repo. Values that differ per environment but are not sensitive — region,
environment name, public endpoints — belong there rather than in secrets, so
they stay reviewable.

## Secrets Management

**Never commit secrets to git.** Use your platform's secret storage, which keeps
values out of the repository and out of build logs. Two rules travel with it:

- Secrets usually apply only to deployments created *after* they are set, so
  setting one is not complete until you redeploy.
- Local development secrets go in gitignored `.env` files, never in the
  committed config.

When a secret appears not to work, list what the deployed environment actually
holds rather than trusting the config file.

## Sub-Skills

- [Cache management + GitHub Actions CI/CD](../paniolo-cloudflare-cache-cicd/SKILL.md)
- [Rollback, monitoring, troubleshooting](../paniolo-deployment-operations/SKILL.md)

## Do Not

- Do not deploy to production without running lint and unit tests first.
- Do not commit secrets to git — use your platform's secret storage.
- Do not use a plain deploy for user-facing changes when your setup needs a
  cache purge. Users will see stale cached versions.
- Do not use for active incident recovery — load `deployment-operations` instead.

## Skill Handoffs

- If active production recovery is required, also load `deployment-operations`.
- If CDN cache behavior must change with deploy, also load `cloudflare-cache-cicd`.
