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

## Addendum 2026-10-14 (second evaluation)
- Fact: Smoke logic was duplicated per repo (drift risk). FIXED: central reusable `Empathy-First-Media/.github/.github/workflows/op-canary.yml@4cccef1720b7af877719a89647873de839c71b5a`; repos keep only a scheduling caller with `secrets: inherit`. Runs 38670252575/38670256397/38670260580 success (canary length 30). Template updated to caller form. Update once in `.github`, Dependabot bumps caller pins.
- Fact: Runner floated on `ubuntu-latest`. FIXED: reusable pins `ubuntu-24.04`.
- Fact: No repo rulesets in any org repo (`rulesets: []`); CODEOWNERS exists in universal-systems/backstage/OmniRoute but nothing enforces review of workflow changes. Native fix: repo ruleset requiring CODEOWNERS review for `.github/workflows/**` (Team-supported). Owner decision (merge friction).
- Fact: OmniRoute + excalidraw lack dependabot-auto-merge; SHA-bump PRs need manual merge there.
- Fact: System still proves a canary only; no production secret has migrated to `op://` yet (G9). Highest-value next step: pilot one low-risk secret in universal-systems, then delete its GitHub copy.
- Fact: Failure signal = scheduled run + watcher email only. Native upgrade: `if: failure()` issue creation with `GITHUB_TOKEN` (read default confirmed org-wide) — deferred, adds write need.

## Transcript judgment 2026-10-14 (live re-check)
- Fact: Org callers still pin `op-canary.yml@4cccef1720b7af877719a89647873de839c71b5a`. Latest runs still success: 38670252575, 38670256397, 38670260580. Canary item `5nawishhkdi4wneaosuxwcypqq` still readable by the GitHub Actions SA. Org repos have zero per-repo OP secrets.
- Fact: P0 defect. The GitHub Actions service-account token was rendered into this session's page snapshot while the org secret form was filled. Treat item `zn4exefxr4kxiwzgrvamcan3nm` as exposed. Rotation blocked here: `op service-account create` returns "authenticated as a service account" (exit 9); desktop app integration is down, so a user session cannot create the replacement.
- Fact: Same token is still set on dlynch90/developer-platform, chezmoi, developer-ecosystem, BRIO-APP-BACKUP, op-actions-template. Personal Actions remain billing-locked. Those copies stay valid until the old SA is revoked.
- Fact: OmniRoute runs 38665887323 and 38666939667 are still `queued` with null conclusion. Cancel returns 409. Earlier "they will expire" claim remains unverified.
- Fact: SUPERSEDED 2026-10-15. Org OP secrets are Private repositories. Public-tier service account is still not created. Old SA is not revoked.
- Fact: No production secret has moved to `op://`. The green runs prove the canary only.
- Bad Pattern this session: direct commits to default branches, no PR review; floating claims before re-check (stuck-run expiry, "FIXING NOW" marked before validation).


## Live correction 2026-10-15
- Fact: G2 visibility half is done (private only). Public-tier vault/SA is not done.
- Fact: Hand-written canary shell removed from op-canary.yml at 337cdf16ae843cbfa8036f876d38c8f4aab98d04.
- Fact: universal-systems main pin is stale until https://github.com/Empathy-First-Media/universal-systems/pull/88 merges. Ruleset blocked a direct push. That block is correct.
- Fact: bootstrap-op-secrets.sh removed from dlynch90/op-actions-template so it cannot recopy the exposed token.

## Sock audit 2026-10-15
- Fact: SUPERSEDED. "Desktop integration is down" is too broad. The sockets connect. Aside's sandbox cannot see the 1Password app bundle or signal the stale `op` helper processes. That is why rotation is blocked here, not because the sock file is missing.
