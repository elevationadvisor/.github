# Code Promotion & Release Process

**Version:** April 27, 2026

-----

## Purpose

This document defines the agreed-upon process for how code moves from development to production at Elevation Advisor. It is intentionally lightweight and practical, designed to reduce deployment risk while keeping the team fast and flexible.

This reflects the process discussed and agreed to in the Weekly Priority Sync on April 27, 2026.

-----

## Core Principles

- Small changes are safer than big ones — large features must be broken into incremental changes.
- Deploy early, expose late — code can be deployed to production without being user-visible.
- Feature flags are the default — any incomplete or risky functionality must be behind a flag.
- Staging is the verification gate — E2E tests and validation happen on staging before production.
- Human review still matters — AI tools can assist, but pull requests require human approval.
- Owner review is a required control — final approval ensures business and customer risk is consciously accepted.

-----

## Branch Strategy

No direct commits to `develop` or `main`. All work must start in a feature or bug branch. Branches are short-lived and should be merged as soon as their purpose is complete.

|Branch Type            |When to Use                      |
|-----------------------|---------------------------------|
|`feature/<description>`|New functionality or enhancements|
|`bug/<description>`    |Bug fixes                        |
|`hotfix/<description>` |P0 production issues only        |

-----

## Release Readiness Checklist

Before merging or promoting code, confirm all of the following:

- [ ] Scope is small enough to understand and roll back
- [ ] Feature flag added (if applicable) and OFF by default
- [ ] PR approved by another developer including comprehension sign-off
- [ ] Staging deploy successful
- [ ] E2E tests passing on staging (manual verification noted for uncovered areas)
- [ ] Owner review completed within one business day, including hands-on staging verification for user-visible changes
- [ ] No unintended user-visible changes in production

### When to Split Work

A task must be split if it is expected to take more than ~1 week, touches multiple major areas of the system, or introduces logic that is hard to test in isolation.

Each chunk should be independently deployable, have a clear purpose, and be safe to ship behind a feature flag.

-----

## Standard Environments

### 1. Local Development

- Developers build and test features locally
- Unit tests and manual testing are expected
- AI tools (Cursor / Claude) may be used as assistive checks only

> ⚠️ AI output must always be reviewed by the developer before proceeding.

### 2. Pull Request (PR)

All changes must go through a Pull Request into `develop`.

**PR Requirements:**

- Clear description of what this change does
- Linked task or priority item (if applicable)
- Feature flag noted (if used)
- At least one other developer must approve the PR

> The PR description is where code comprehension is enforced. Developers must be able to explain what the code does at a minimum. Even better if they can explain why it was written that way and what the downstream consequences of changing it would be.

### 3. Staging Environment

- Merging into `develop` triggers deployment to staging
- CI runs the E2E test suite covering core workflows
- Feature flags remain OFF by default

Staging is where we verify nothing breaks, validate integrations, and confirm feature-flagged code is safe.

> **Current state:** E2E tests cover critical user workflows. Automated unit test coverage is being built incrementally. Manual verification by the developer is the current standard for areas not covered by E2E tests. New features that introduce user-visible functionality require corresponding E2E coverage before promotion to production.

### 4. Owner Review (Pre-Production Gate)

Before any code is promoted to production, it must be reviewed by Gregg. Owner review must be completed within one business day of the staging deploy being ready.

Owner review includes:

- Hands-on verification of user-visible changes on staging
- Business and customer impact assessment
- Feature readiness and scope confirmation
- Confirmation that feature flags are correctly applied
- Explicit approval to promote to production

> Owner review is both a business approval and a functional testing step. For any user-visible change, Gregg verifies behavior directly on staging, not just approves on paper.

**Exceptions — owner review may be bypassed for:**

- Copy or styling-only changes with no behavioral impact
- Bug fixes that correct existing logic without introducing new code paths
- Internal refactoring with no user-visible change

*When bypassing, the developer must note the exception reason in the PR.*

### 5. Production Environment

- Code is promoted from `develop` to `main`
- Feature flags are used to prevent incomplete or unapproved functionality from being visible to users
- Approved features may be merged and deployed with their feature flags enabled
- No incomplete or unapproved functionality should be visible to users.

Production deploys should be incremental, reversible, and avoid bundling multiple risky changes.

If an issue is discovered in production:

- First response is disabling the relevant feature flag
- Code rollbacks should be rare and deliberate

-----

## Feature Flags

Feature flags are required when a feature is incomplete, affects core workflows, needs gradual rollout, or is difficult to fully test in advance.

|                              |Details                                                                                  |
|------------------------------|-----------------------------------------------------------------------------------------|
|**Feature flags required**    |Incomplete features, core workflow changes, gradual rollouts, high-risk changes          |
|**Feature flags not required**|Copy/styling only, internal refactoring, bug fixes correcting existing logic, dev tooling|
|**Default rule**              |If in doubt, use a feature flag                                                          |
|**Flag inventory**            |Flags must be tracked, owned, and removed within 30 days of full rollout                 |
|**Production enablement**     |Only Gregg or a designated admin may enable flags in production                          |

-----

## Backend Changes

Backend changes follow the same flow: PR → Staging → Owner Review → Production. Backend logic should be guarded by feature flags when it introduces new data paths, new calculations, or new integrations.

Backend-only changes should be treated as if they will eventually be exercised by users and must be safe even when not immediately visible.

-----

## Hotfix (P0 Production Bug) Process

Hotfixes are reserved strictly for P0 production issues actively affecting customers.

**Hotfix Flow:**

1. Create a `hotfix/<description>` branch from `main`
1. Implement the minimal fix required
1. Open a PR — at least one developer review is required
1. Deploy directly to production
1. Obtain approval from Gregg
1. Merge hotfix back into `develop` as soon as possible

> ⚠️ Hotfixes still require human review. Speed matters but safety still applies. Feature flags should be used if feasible.

-----

## Default Flow Summary

```
Local Dev          → build and test locally
Feature/Bug Branch → short-lived, focused scope
Pull Request       → peer review + comprehension sign-off
Staging            → E2E tests + manual verification
Owner Review       → hands-on staging check + business approval (1 business day SLA)
Production (main)  → feature flags OFF by default
Enable feature     → intentionally, when ready, by authorized person
```
