# CLAUDE.md — zc-cert

## What this repo is
Generic Ansible automation for SCEP certificate enrolment and 802.1X/EAP-TLS
Wi-Fi profile deployment on Linux clients. Designed for ansible-pull.

## Privacy — CRITICAL
This is a **public repository**. The specific organisation and infrastructure
using it must never be identifiable from anything committed here.

Rules for every commit, comment, and file:
- **No organisation names** — not in file contents, comments, commit messages,
  or variable values.
- **No internal hostnames, FQDNs, or IP addresses** — not even in commit
  message bodies or `# examples`.
- **No SSID names, AD group names, or domain names** that identify the org.
  These belong exclusively in `vault.yml` (ansible-vault encrypted).
- **No Co-Authored-By trailers** — do not add Claude or any tool as a
  co-author in commit messages.
- Vault password is **never** committed, logged, or printed to stdout.

When in doubt: if it would tell an outsider which company runs this,
it does not belong in this repo.

## Sensitive data locations
All org-specific values live in `playbooks/group_vars/all/vault.yml`
(ansible-vault encrypted). Public files contain only generic defaults,
structure documentation, and placeholder comments.

## Key files
| Path | Purpose |
|------|---------|
| `playbooks/scep_enroll.yml` | Main playbook (runs all 4 roles) |
| `playbooks/group_vars/all/vars.yml` | Non-sensitive defaults |
| `playbooks/group_vars/all/vault.yml` | Encrypted secrets |
| `roles/scep_client/` | Install certmonger, deploy CA trust |
| `roles/scep_enroll/` | Request device certificate via SCEP |
| `roles/wifi_profile/` | Deploy EAP-TLS profile, test + rollback |
| `roles/telegram/` | Shared Telegram notification sender |
