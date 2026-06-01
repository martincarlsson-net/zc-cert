# zc-cert

Automates SCEP certificate enrolment on Fedora Linux clients using FortiAuthenticator as the CA, then deploys an 802.1X/EAP-TLS Wi-Fi profile using the issued certificate.

## How it works

1. **scep_client** — installs certmonger, deploys the CA trust chain, and registers the FortiAuthenticator SCEP CA with a wrapper that works around SHA-1 signing.
2. **scep_enroll** — requests a device certificate over SCEP and waits for it to be issued.
3. **wifi_profile** — creates an EAP-TLS NetworkManager profile for the target SSID, tests it (40 s timeout), and rolls it back automatically on failure.

The connection test runs detached because activating the cert Wi-Fi drops the management Wi-Fi on the same radio — and with it the SSH session. A failsafe timer guarantees the host returns to its management network regardless of the test outcome.

## Requirements

- Fedora (tested on Fedora 44); other RHEL-family distros should work with minor path adjustments.
- `ansible` installed on the enrolling machine (for `ansible-pull`).
- Vault password shared with you out-of-band.

## Enrol a new client

Install Ansible, then run:

```bash
sudo dnf install -y ansible
sudo ansible-pull -U https://github.com/martincarlsson-net/zc-cert.git \
  playbooks/scep_enroll.yml --ask-vault-pass
```

Enter the vault password when prompted. The playbook handles everything else.

## Secrets

Sensitive values (FAC URL, CA names, SCEP challenge password, certificate subject, Wi-Fi SSID) are stored in `group_vars/all/vault.yml`, encrypted with `ansible-vault`. The vault password is never committed.

To update a secret (e.g. rotate the SCEP challenge password):

```bash
# Decrypt in-place, edit, re-encrypt
ansible-vault decrypt group_vars/all/vault.yml
$EDITOR group_vars/all/vault.yml
ansible-vault encrypt group_vars/all/vault.yml
git add group_vars/all/vault.yml && git commit -m "rotate SCEP challenge password"
git push
```

## Variables

Non-sensitive tunables (timeouts, PMF) are in `group_vars/all/vars.yml`.  
Role-level defaults (file paths, binary paths) are in each role's `defaults/main.yml`.
