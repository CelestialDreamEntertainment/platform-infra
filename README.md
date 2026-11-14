# platform-infra
Shared live infrastructure for the apps suite, managed with Terraform and deployed like an app: `plan` on PR, `apply` to dev on merge, and `apply` of the same commit to prod after manual approval. Prod releases get date tags (e.g. `2026.09.24`).

> **Status:** not scaffolded yet. The design lives in the suite README (`../README.md`), which is the source of truth until this repo has code.

## What's here (planned)
- **AWS foundations:** GitHub-to-AWS OIDC deploy roles (trusting only `main` and named environments), Terraform state buckets, and the build artifacts bucket.
- **Atlas:** one project per environment, each with a free M0 cluster.
- **Cognito:** one user pool per environment on the Essentials plan, with email one-time codes and optional passkeys and no self sign-up. It also includes the managed login custom domain (`auth.app.sabrinabea.com` / `auth.dev.sabrinabea.com`) and a **pre-token-generation Lambda** that copies group names into the `scope` claim.
- **DNS, certificates, and email per environment:**
    - the delegated hosted zone (`app.sabrinabea.com` in prod, `dev.sabrinabea.com` in dev)
    - a wildcard certificate in us-east-1
    - the parent `A` record Cognito requires
    - an SES identity (prod: `notify.app.sabrinabea.com`)
- **The SSM parameter contract:** shared values (e.g. `/platform/atlas/project-id`, the user pool ID, certificate ARNs, the artifacts bucket) that the `platform` Terraform modules read.
- **Invite script:** `pnpm invite <email> --group <scope> --env <env>` and `pnpm revoke ...`. They're run locally with my AWS credentials. Inviting adds the requested group plus the app's base group and sends an invitation through SES.

## `management/`
A small Terraform config for the main `sabrinabea.com` zone, which lives in the AWS Organization's management account alongside my Gmail records. **I apply it by hand. CI never runs it.** It contains only:
- NS records delegating `app.sabrinabea.com` to prod and `dev.sabrinabea.com` to dev
- the `*.local.sabrinabea.com` → `127.0.0.1` wildcard

## Security
- This repo holds the most powerful AWS roles in the suite, and it's the only one that does.
- AWS-connected PR jobs run only for branches in this repo, never forks.
- Sensitive variables and outputs are marked `sensitive`, because plan output is public.
- User email addresses never appear in this repo.
