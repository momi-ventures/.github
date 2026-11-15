---
trigger: always_on
description: "Fleet CI: GitHub-hosted Actions never run in momi-ventures; CI runs locally and on self-hosted runners"
---
<!-- secretary-managed: fleet-ci doctrine v1 -->

# CI in every momi-ventures repository

**Founder decision (2026-11-04 for secretary-v1, 2026-11-14 for the fleet): GitHub Actions billing stays
off, permanently.** Every GitHub-hosted job fails in about 2 s with 0 steps and the annotation
*"The job was not started because recent account payments have failed or your spending limit needs to be
increased."* That is the normal state of the org.

## What that means for you

- A check refused that way is **infrastructure, not a code failure**. Do not re-run it, do not spend time
  proving it unrelated, and **never ask the founder to fix billing or re-enable Actions**.
- Identify it exactly, by the annotation (`gh api repos/<o>/<r>/check-runs/<job id>/annotations`), never by
  "it failed fast". This is what `land-pr` does (`src/coordination/land_pr.ts`, `UNSTARTED_ANNOTATION`).
- The verdict comes from local CI on the merged or pushed tree:

| Repo | CI of record | Status it posts |
|---|---|---|
| secretary-v1 | `node bin/coord.ts land-pr <n>` (tests the merge), `npm run ci:local` before a PR, `node bin/ci_nightly.ts` | `secretary/land-pr`, `secretary/nightly` |
| EasyDeID | `python scripts/local-ci.py --profile full --post-status` (WPF tests need Windows) | `easydeid/local-ci` |
| any other repo | `node bin/coord.ts land-pr <n> --repo <checkout>` covers node and pytest suites; add the repo here when it gets its own runner | `secretary/land-pr` |

- Workflows run only on self-hosted runners: `runs-on: [self-hosted, secretary-ci, linux|windows]`.
  A job with no online runner stays QUEUED, never red; local CI still decides. `ci/check_workflows.ts <dir>`
  rejects a workflow that could schedule a hosted image.

## What to ask the founder instead

Only things that improve local CI, and only when they block real work:

1. **Runner capacity.** No Windows runner exists yet, so Windows legs (EasyDeID's WPF tests) only run on a
   developer machine. The ask: on a Windows host, elevated PowerShell, in a secretary-v1 checkout,
   `.\ci\runner\install-runner.ps1 -Scope org -Target momi-ventures` (one runner serves every private repo).
2. **A suite that is slow, flaky or missing**, with the measurement and the proposed fix.
3. **Scheduling the nightly run** (`node bin/ci_nightly.ts`) on a host: an operator decision, ~1 h of one machine.

Details, in `momi-ventures/secretary-v1`: `ci/README.md`, `docs/product/ci-of-record.md`.
