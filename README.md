# Empathy-First-Media/.github

Fact: Org workflow templates live in `workflow-templates/`. New org repos:
Actions > New workflow > "1Password Secrets" adds the smoke workflow.
Fact: Verified 2026-10-15. One org Actions secret `OP_SERVICE_ACCOUNT_TOKEN`, visibility Private repositories. Service account `GitHub Actions - Automation - Read` reads vault `Automation` (`zacszu4fq4wmuk7qnp7ehvhhxa`) only.
Fact: Org variable `OP_VAULT_ID` is `zacszu4fq4wmuk7qnp7ehvhhxa`, visibility Private repositories.
Good Pattern: After `gh auth refresh -h github.com -s admin:org`, set the bootstrap token once. Never print the token.
`op read 'op://zacszu4fq4wmuk7qnp7ehvhhxa/<sa-item-id>/credential' | gh secret set OP_SERVICE_ACCOUNT_TOKEN --org Empathy-First-Media --visibility private`
Bad Pattern: `--visibility all`, a per-repo bootstrap script, or a service account that can read Development.
