# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

The suite-wide `../CLAUDE.md` and the design in `../README.md` also apply. This repo is not scaffolded yet, so there are no commands. Add them here once they exist.

## What this repo is
The `platform-infra` repo: shared live infrastructure (OIDC roles, state and artifacts buckets, Atlas, Cognito, per-environment DNS zones, certificates, and SES), plus the invite and revoke scripts. It deploys dev on merge and prod on approval.

## Rules
- **`management/` is hand-applied only.** Never add it to CI, and never add anything to it beyond NS delegations and the local wildcard record. The main zone holds the user's email records.
- **Never put user email addresses** or other personal data in this repo, including Terraform, scripts, tests, and fixtures. Users live only in Cognito.
- **Mark sensitive values `sensitive`.** Plan output is posted to public PRs.
- **The SSM parameter names are a contract with the `platform` modules.** Add new ones freely, but renaming or removing one needs coordinated PRs: this repo first, then the module release.
- **Flag cost.** Anything that costs money (hosted zones, SES volume, a Cognito plan change, NAT gateways) must be flagged before it's introduced.
- **The pre-token Lambda and the invite scripts** are held to the same standards as app code: 100% coverage and RACC.
