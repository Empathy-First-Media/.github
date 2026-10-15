# Empathy-First-Media/.github

Fact: Org workflow templates live in `workflow-templates/`. New org repos:
Actions > New workflow > "1Password Secrets" adds the smoke workflow.
Fact: Zero-setup for new org repos requires ONE org Actions secret
`OP_SERVICE_ACCOUNT_TOKEN` (visibility: all repositories). Set it once
after `gh auth refresh -h github.com -s admin:org`:
`op read 'op://<vault-id>/<sa-item-id>/credential' | gh secret set OP_SERVICE_ACCOUNT_TOKEN --org Empathy-First-Media --visibility all`
Until then, run `scripts/bootstrap-op-secrets.sh` (in dlynch90/op-actions-template) per repo.
