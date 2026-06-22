# Ansible Project — Vault Setup Guide

## Project structure

```
ansib/
├── ansible.cfg
├── .gitignore
├── SECRET_ROTATION.md          # Rotation runbook + migration path
├── .vault_keys/                # NEVER committed — listed in .gitignore
│   ├── dev_pass.txt
│   ├── staging_pass.txt
│   └── prod_pass.txt
├── inventory/
│   ├── hosts.ini
│   └── group_vars/
│       ├── dev/
│       │   ├── vars.yml        # Non-sensitive — safe to commit
│       │   └── vault.yml       # Encrypted — safe to commit after encrypting
│       ├── staging/
│       │   ├── vars.yml
│       │   └── vault.yml
│       └── prod/
│           ├── vars.yml
│           └── vault.yml
└── playbooks/
    ├── setup-server.yml
    └── roles/
        ├── users/
        ├── docker/
        └── nginx/
```

---

## First-time setup

### 1. Create vault password files (do this once per machine)
```bash
mkdir -p .vault_keys
echo "your-dev-password"     > .vault_keys/dev_pass.txt
echo "your-staging-password" > .vault_keys/staging_pass.txt
echo "your-prod-password"    > .vault_keys/prod_pass.txt
chmod 600 .vault_keys/*.txt
```
> Prod password should come from your team's secrets manager, not be invented locally.

### 2. Edit the vault files with real secret values
```bash
ansible-vault edit --vault-id dev@.vault_keys/dev_pass.txt \
  inventory/group_vars/dev/vault.yml
```
Repeat for staging and prod.

### 3. Encrypt all vault files before first commit
```bash
ansible-vault encrypt --vault-id dev@.vault_keys/dev_pass.txt \
  inventory/group_vars/dev/vault.yml

ansible-vault encrypt --vault-id staging@.vault_keys/staging_pass.txt \
  inventory/group_vars/staging/vault.yml

ansible-vault encrypt --vault-id prod@.vault_keys/prod_pass.txt \
  inventory/group_vars/prod/vault.yml
```

---

## Running playbooks

```bash
# Dev
ansible-playbook playbooks/setup-server.yml -l dev \
  --vault-id dev@.vault_keys/dev_pass.txt

# Staging
ansible-playbook playbooks/setup-server.yml -l staging \
  --vault-id staging@.vault_keys/staging_pass.txt

# Production
ansible-playbook playbooks/setup-server.yml -l prod \
  --vault-id prod@.vault_keys/prod_pass.txt
```

---

## Definition of Done checklist

| Requirement | How it is met |
|---|---|
| No plaintext secrets in source control | vault.yml files are encrypted before any commit |
| Separate secret scopes per environment | Separate vault IDs (dev / staging / prod) with separate passwords |
| Vault password sources are access-controlled | `.vault_keys/` is in `.gitignore`; prod password lives in secrets manager |
| Secret rotation documented and tested | See `SECRET_ROTATION.md` |
| Logs do not reveal secret values | `no_log: true` set at play level; only safe tasks override with `no_log: false` |
| Migration path to centralized secrets manager | Documented in `SECRET_ROTATION.md` under "Migration path" |
