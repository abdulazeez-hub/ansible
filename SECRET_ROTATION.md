# Secret Rotation Runbook

## When to rotate
- Every 90 days (scheduled)
- Immediately after a team member leaves
- Immediately if a secret is suspected to be exposed

---

## Rotating a user password

### Step 1 — Generate a new password
```bash
openssl rand -base64 20
```

### Step 2 — Decrypt the vault file for the target environment
```bash
ansible-vault decrypt --vault-id prod@.vault_keys/prod_pass.txt \
  inventory/group_vars/prod/vault.yml
```

### Step 3 — Update the password value in vault.yml
Edit `vault_devops_user_password` with the new value.

### Step 4 — Re-encrypt the vault file
```bash
ansible-vault encrypt --vault-id prod@.vault_keys/prod_pass.txt \
  inventory/group_vars/prod/vault.yml
```

### Step 5 — Re-run the playbook to apply the new password
```bash
ansible-playbook playbooks/setup-server.yml -l prod \
  --vault-id prod@.vault_keys/prod_pass.txt
```

### Step 6 — Verify login works with the new password, then commit
```bash
git add inventory/group_vars/prod/vault.yml
git commit -m "chore: rotate prod devops user password"
git push
```

---

## Rotating the vault password itself (rekey)

Use this when the master vault password is compromised or a team member who knew it leaves.

```bash
ansible-vault rekey --vault-id prod@.vault_keys/prod_pass.txt \
  --new-vault-id prod@prompt \
  inventory/group_vars/prod/vault.yml
```

Then update `.vault_keys/prod_pass.txt` (or your secrets manager entry) with the new password.

---

## Rotation test checklist
After any rotation:
- [ ] Playbook runs cleanly against the target environment
- [ ] SSH access with the new key/password confirmed
- [ ] Old credentials confirmed non-functional
- [ ] Rotation date logged in your team's secret inventory register

---

## Migration path to centralized secrets manager

### Current state
Secrets live in Ansible Vault encrypted files per environment.

### Target state (recommended: AWS Secrets Manager or HashiCorp Vault)

**Why migrate?**
- Centralized audit log of who accessed what secret and when
- Automatic rotation without re-running playbooks
- Fine-grained IAM/RBAC access control per secret
- No vault password files to manage on developer machines

**Migration steps (when approved):**

1. Store each secret in AWS Secrets Manager under a path like:
   `/<project>/<env>/devops_user_password`

2. Replace `vault.yml` variables with `aws_ssm` lookups in `vars.yml`:
   ```yaml
   devops_user_password: "{{ lookup('aws_ssm', '/myproject/prod/devops_user_password') }}"
   ```

3. Grant the CI/CD runner IAM role read access to the relevant secret paths only.

4. Remove vault files and vault password files from the project entirely.

5. Update `ansible.cfg` — remove any `vault_password_file` references.

> Until migration is approved, Ansible Vault remains the enforced standard.
> Do not mix both approaches in the same environment.
