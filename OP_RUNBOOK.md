# 1Password for GitHub Actions

## Configuration

Use organization Actions secrets scoped to **Private repositories**:

- `OP_SERVICE_ACCOUNT_TOKEN`: a service-account token with read access to the Automation vault only.
- `OP_SA_CANARY_REF`: an ID-based `op://<vault-id>/<item-id>/<field>` reference to the dedicated canary.

Confirm each private caller has access to both secrets. Do not broaden visibility
to public repositories or add repository overrides with the same names.
Keep the resolved canary in 1Password; do not copy its value into GitHub.
Other workflows may use `OP_VAULT_ID` for ID-based references, but this canary
uses only the two named secrets above. Never print tokens or resolved values.

## Canary contract

The caller template uses `.github/workflows/op-canary.yml` at:

`7a7a2f9573fd2e86da775780bce528c7ec6ea2bd`

The reusable workflow declares both secrets as required. It rejects an empty
token or missing `op://` reference, calls the vendor action, and then requires
a nonempty `steps.load_canary.outputs.OP_SA_CANARY`. It imposes no fixed length.

Secret loading uses `1password/load-secrets-action` v5.0.1 pinned to
`70062d7a876d3eb6334754fa26efd2fbd90c32f2`, with `export-env: false`.
Authentication, reference resolution, and masking remain in the vendor action.
Do not add an authentication wrapper or use `op run` in CI.

## Install or update a caller

1. In a private repository, choose **Actions > New workflow > 1Password Secrets**,
   or copy [workflow-templates/op-secrets.yml](workflow-templates/op-secrets.yml).
2. Keep the two explicit secret mappings. Confirm their organization access scope.
3. For an existing caller, update its full commit pin through a reviewed PR.
   Updating the shared workflow alone does not change callers pinned to older commits.
4. Dispatch `op-secrets-smoke` and inspect the exact workflow revision and result.
   Require success from the input check, vendor load, and nonempty-output check.

A successful run verifies that caller at that time. Configuration inspection and
older runs do not prove current authentication or production-secret coverage.

## Rotate the service account

1. Create a replacement read-only service account for the Automation vault in
   1Password. Keep the existing account available until the replacement passes.
2. Update `OP_SERVICE_ACCOUNT_TOKEN` in organization **Settings > Secrets and
   variables > Actions**, preserving **Private repositories** visibility.
   Use the native secret editor or `gh secret set`; never put the token in logs.
3. Dispatch current canary callers using the replacement. Check every affected
   consumer before retiring the old account; do not treat one caller as all coverage.
4. If verification fails, restore the previous organization secret if needed,
   diagnose the failing step, and verify the replacement before continuing.
5. Revoke the old service account in 1Password after its consumers pass with
   the replacement. Dispatch the canary again after revocation.

## Troubleshooting

- **Input check fails:** confirm access to both named secrets and an ID-based reference.
- **Vendor load fails:** inspect the action error for authentication, reference,
  vault-permission, or service/connectivity problems. Do not print secret values.
- **Output check fails:** the action did not provide a nonempty `OP_SA_CANARY` output.
  Check the dedicated field and output wiring; preserve the failure check.
- **Queued or startup failure:** inspect GitHub's run annotations and repository
  Actions settings, policies, runner availability, concurrency, and usage/billing.
  The status alone does not establish which of these caused the problem.

## Maintenance

Keep action and reusable-workflow references pinned to full commit SHAs. Review
vendor updates and validate callers after pin changes. Use [1Password's action
documentation](https://www.1password.dev/ci-cd/github-actions) for supported inputs.
[GAP_ANALYSIS.md](GAP_ANALYSIS.md) preserves historical proposals and incidents;
revalidate its unresolved items before acting. Git history retains prior run
inventories and device-local diagnostics; they are not current operating state.
