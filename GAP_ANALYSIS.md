# Gap Analysis — 1Password x GitHub Actions (Empathy-First-Media)
Verified 2026-10-14. Sources: 1password.dev (GitHub Actions, Service Accounts, load-secrets-action README), docs.github.com (Secure use, Rulesets, Lockfiles). Vendor-native only; no custom code beyond vendor snippets + config.

## What was already right
- Fact: Dedicated GitHub Actions service account, read-only, ID-based `op://` refs, `load-secrets-action` (owner: 1Password, Verified creator), `permissions: contents: read`, dispatch-only smoke, masked outputs, org secret inheritance proven on new + existing repos.

## Gaps found (severity ordered)
| # | Gap | Evidence | Vendor-native fix | Status |
|---|---|---|---|---|
| G1 | Floating tag `@v5` can be repointed (supply chain) | smoke workflows use `1password/load-secrets-action@v5`; GitHub Secure Use: pin full-length SHA, lockfiles supersede | Pin `70062d7a876d3eb6334754fa26efd2fbd90c32f2 # v5.0.1`; Dependabot github-actions bumps SHA+comment | DONE 2026-10-14, runs 38669988882/38669992430/38669996193 success |
| G2 | Org secret visible to ALL repos incl. public (OmniRoute, excalidraw) = broad Development-vault blast radius | Org secret visibility "All repositories" | Tier: org secret -> Private repositories; separate public-tier SA scoped to a new "GitHub Actions: Public CI" vault (canary + public-safe items only), stored as repo secret on public repos (repo secret overrides org, workflow unchanged) | NEEDS 1Password UI (vault+SA create); desktop CLI integration down, SA cannot create vaults/SAs |
| G3 | Canary depends on unrelated Context7 item (owner can rotate/delete -> false outage) | `OP_SA_CANARY_REF` -> Context7 item `2abogizfsusjkpz2432bastcdq` | Dedicated item `5nawishhkdi4wneaosuxwcypqq` "GitHub Actions - CI - Canary" | DONE 2026-10-14, org secret updated, canary length 30 verified |
| G4 | No scheduled verification (silent token expiry/rotation breakage) | smoke is dispatch-only | Native cron `17 14 * * 1` UTC weekly in smoke workflow | DONE 2026-10-14 (first scheduled run next Monday) |
| G5 | Required-workflow enforcement unavailable | Org plan = Team; GitHub docs: "Require workflows to pass" ruleset rule is Enterprise Cloud only | Fallback (native, Team): org workflow templates + org secrets (done) + repo ruleset requiring the local smoke status check where wanted. Re-evaluate on Enterprise upgrade | DOCUMENTED, plan-gated |
| G6 | Workflow dependency lockfiles not adopted | GitHub: lockfile (`.github/workflows/actions.lock`, `gh actions-lock`) supersedes SHA pinning, public preview | Pilot on universal-systems after G1; preview risk noted, SHA pin remains baseline | PILOT LATER |
| G7 | Workload Identity not adopted (still one long-lived token) | load-secrets-action README: Workload Identity = public preview, OIDC exchange, needs 1Password Environment + `OP_INTEGRATION_KEY` | Pilot only; preview + still has one secret. Re-check at GA | DOCUMENTED |
| G8 | Local `gh` cannot audit org secrets/policies (403 admin:org) | sandbox keychain write fails (`gh auth refresh` exit 161) | Run `gh auth refresh -h github.com -s admin:org` in native terminal once; then API audits become deterministic | OWNER ACTION |
| G9 | Org still holds legacy copied-value secrets (2025-11-11) beside 1Password pattern | Org secret list: AIRFLOW_*, NEO4J_*, etc. | Migrate each to 1Password item + `op://` load; delete GitHub copy per repo after smoke | BACKLOG (per-secret owner check) |
| G10 | No branch protection requiring smoke on main | universal-systems main: "Branch not protected" | Native repo ruleset: require `op-secrets-smoke` status check on PRs (Team-supported rule) once scheduled/PR trigger chosen | OPTIONAL (can block merges; owner decision) |

## Determinism rules adopted
- Fact: Action refs = full SHA + version comment only. Cron = `17 14 * * 1` UTC weekly. Canary = dedicated item ID. Refs = vault/item IDs only. One org secret per tier. Every change validated by a fresh dispatch run ID, never by config inspection alone.
