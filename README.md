# Empathy-First-Media/.github

Shared GitHub Actions workflows and organization workflow templates.

## 1Password canary

Use **Actions > New workflow > 1Password Secrets** to install
[the caller template](workflow-templates/op-secrets.yml) in a private repository.
It pins the shared canary and passes only these named organization Actions secrets:

- `OP_SERVICE_ACCOUNT_TOKEN`: a service-account token with read access to the Automation vault only.
- `OP_SA_CANARY_REF`: an ID-based `op://<vault-id>/<item-id>/<field>` reference to the dedicated canary.

Set both secrets to **Private repositories** visibility and confirm the caller
can access them. Keep the canary value in 1Password. The workflow checks its
inputs, loads through the pinned vendor action, and requires a nonempty result
without printing secret values.

See [OP_RUNBOOK.md](OP_RUNBOOK.md) for setup, the current pinned contract,
rotation, and troubleshooting. [GAP_ANALYSIS.md](GAP_ANALYSIS.md) is historical
context; its account states, run results, and proposed work require revalidation.
