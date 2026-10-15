# 1Password for GitHub Actions — Empathy-First-Media Runbook
Verified 2026-10-14 (op 2.41.0, load-secrets-action v5.0.1)

## Architecture (single source of truth)
- Fact: Org secret `OP_SERVICE_ACCOUNT_TOKEN` (Private repositories, changed 2026-10-15) = token of 1Password item `zn4exefxr4kxiwzgrvamcan3nm` "1Password - Development - Service Account Token (GitHub Actions)", vault `dfncpo4o2wrkjf6f3zo6zu35ta` (Development: Global Fallback), read-only.
- Fact: Org secret `OP_SA_CANARY_REF` (Private repositories, changed 2026-10-15) = `op://dfncpo4o2wrkjf6f3zo6zu35ta/5nawishhkdi4wneaosuxwcypqq/credential` (dedicated canary item "GitHub Actions - CI - Canary", dummy value, a ref not a real secret).
- Fact: Org variable `OP_VAULT_ID` (All repositories) = `dfncpo4o2wrkjf6f3zo6zu35ta`.
- Good Pattern: No per-repo OP secrets in this org. Repo secrets with the same name override org; do not add them except for a temporary rotation test.
- Good Pattern: Workflows load secrets live: `1password/load-secrets-action@v5` with `OP_SERVICE_ACCOUNT_TOKEN: ${{ secrets.OP_SERVICE_ACCOUNT_TOKEN }}` and `NAME: op://${{ vars.OP_VAULT_ID }}/<item-id>/credential`. Outputs are masked.
- Bad Pattern: `op run` in CI, copied secret values in GitHub, name-based `op://` refs, fork-PR access to secrets (GitHub blocks it by design).

## New repo (automatic)
1. Create the repo in Empathy-First-Media. Org secrets/variable already apply.
2. Actions > New workflow > "1Password Secrets" (template in this repo), or copy `.github/workflows/op-secrets-smoke.yml` from universal-systems.
3. Dispatch `op-secrets-smoke`. Expect success in ~15s and log "1Password load OK: canary non-empty (length 30 for the dedicated canary)".
4. Proof on file: fresh repo `op-newrepo-proof` passed run 38666635572 with zero per-repo secrets (archived after proof).

## Add a real secret to a workflow
1. Put/rotate the value in 1Password (ID-based item). Note item ID + field (`credential` for API Credential items).
2. In the workflow load step env add: `MY_KEY: op://${{ vars.OP_VAULT_ID }}/<item-id>/credential`
3. Consume `${{ steps.<load-step-id>.outputs.MY_KEY }}` or `export-env: true`. Never echo it.

## Rotate the service account
1. 1Password > Developer > Service Accounts: create new SA, read-only, vault `dfncpo4o2wrkjf6f3zo6zu35ta` only. Store token in the vault item above (replace `credential`), keep old SA until step 4 passes.
2. Org Settings > Secrets and variables > Actions: edit `OP_SERVICE_ACCOUNT_TOKEN` with the new token. (CLI alternative after `gh auth refresh -h github.com -s admin:org` outside the Aside sandbox: `op read 'op://dfncpo4o2wrkjf6f3zo6zu35ta/zn4exefxr4kxiwzgrvamcan3nm/credential' | gh secret set OP_SERVICE_ACCOUNT_TOKEN --org Empathy-First-Media --visibility all` — values never printed.)
3. Dispatch `op-secrets-smoke` in universal-systems + backstage. Expect green.
4. Delete the old service account in 1Password. Re-dispatch smoke once more.

## Troubleshooting
- Fact: Run stuck `queued` with no jobs = repo Actions disabled. Fix: repo Settings > Actions, or `gh api --method PUT repos/<owner>/<repo>/actions/permissions -F enabled=true -f allowed_actions=all` (fixed OmniRoute 2026-10-14, run 38667107120 then passed in 11s).
- Fact: `startup_failure` on every run in `dlynch90` personal repos = personal account billing lock ("account is locked due to a billing issue", run 38665791754). Org unaffected. Fix billing, then re-dispatch.
- Fact: Canary empty = check org secret exists + SA still has vault read; test locally: `OP_SERVICE_ACCOUNT_TOKEN=$(cat ~/.config/1password/service-account-token) op read 'op://dfncpo4o2wrkjf6f3zo6zu35ta/5nawishhkdi4wneaosuxwcypqq/credential'` (boolean only, never print).
- Fact: Archived repos (e.g. `dlynch90/developer-ecosystem`) reject workflow files by design.

## Validated inventory (2026-10-14)
| Repo | Smoke run | Result |
|---|---|---|
| universal-systems | 38667238376 (org secret only) | success 10s |
| backstage | 38667190843 (org secret only) | success 10s |
| OmniRoute | 38667107120 | success 11s |
| op-newrepo-proof (archived) | 38666635572 | success 15s |
| excalidraw | n/a (upstream fork, no workflow file added; org secret still applies if one is added) | documented |


## Live correction 2026-10-15 (supersedes conflicting lines above)
- Fact: OP_SERVICE_ACCOUNT_TOKEN and OP_SA_CANARY_REF visibility is Private repositories. Re-checked in the org settings UI.
- Fact: OP_VAULT_ID is still All repositories. It is a vault ID, not a secret.
- Fact: Public OmniRoute caller was removed. Do not treat run 38667107120 as current coverage.
- Fact: Personal dlynch90 copies of both OP secrets were deleted. Count is 0.
- Fact: Exposed service account is NOT revoked. Aside cannot rotate it. Native Terminal can.
- Fact: OmniRoute runs 38665887323 and 38666939667 are still queued. Cancel returns 409. They have not expired.
- Fact: Canary workflow no longer has a custom shell check. Vendor action failure is the check. Callers that can push pin 337cdf16ae843cbfa8036f876d38c8f4aab98d04. universal-systems main is still on 4cccef17 until PR 88 merges.

## Sock audit 2026-10-15 05:15 ET (supersedes "desktop app is down")
- Fact: `~/.config/op/op-daemon.sock` is a live socket. `connect()` succeeds. It is not a dead file.
- Fact: These also accept connections: Group Containers `t/s.sock`, `t/agent.sock`, and `/var/folders/_j/494ynvld7r368yhyjf7dggd00000gn/T/op-daemon.sock`.
- Fact: Three `op` helpers hold those sockets and Aside cannot signal them: pid 13394 is brew `op` 2.41.0, pid 79624 is mise `op` 2.40.0, pid 28528 is `2.40.0.upgrading`. `kill` returns Operation not permitted.
- Fact: `op account list` from Aside still fails in 34ms with the desktop-app connection error. The 1Password app process is running and listening on `s.sock`. Aside cannot read `/Applications/1Password.app` (`sandbox_extension_issue_file` Operation not permitted) and `open -a 1Password` returns "Unable to find application named 1Password".
- Fact: This session cannot restart the app, kill the stale helpers, or complete revocation. Native Terminal was reported working by another session. That was not re-verified here.
