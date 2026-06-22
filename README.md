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
