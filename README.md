# zc-cert

Automates Fedora workstation provisioning, certificate enrolment, and
802.1X/EAP-TLS Wi-Fi authentication using an internal PKI and RADIUS
infrastructure. Playbooks run locally on each client — no central Ansible
server required.

## Workstation setup overview

Setup is split into four ordered playbooks. Reboot where indicated.

| Step | Playbook | Privilege | Reboot after |
|------|----------|-----------|--------------|
| 1 | `playbooks/himmelblau.yml`   | root (sudo) | **Yes** — then log in as your Entra ID user |
| 2 | `playbooks/system_root.yml`  | root (sudo) | Recommended |
| 3 | `playbooks/system_user.yml`  | user (no sudo) | No |
| 4 | `playbooks/scep_enroll.yml`  | root (sudo) | No |

## Requirements

- Fedora Linux (other RHEL-family distros may work with minor adjustments)
- `git` and `ansible-core` installed on the client
- `community.general` and `ansible.posix` collections
  (`ansible-galaxy collection install community.general ansible.posix`)
- Vault password obtained out-of-band from your administrator
- Vault values configured (see [Vault values](#vault-values))

## Step 1 — Himmelblau / Entra ID join

Joins the device to Entra ID so users authenticate against it.

```bash
sudo ansible-playbook -c local -i inventory/hosts playbooks/himmelblau.yml \
  -e himmelblau_shell_choice=bash
```

**Reboot**, then log in with your Entra ID user before continuing.

## Step 2 — System install (root)

Hostname, third-party repos, system packages, Docker, libvirt, snapper and CLI
tooling.

```bash
sudo ansible-playbook -c local -i inventory/hosts playbooks/system_root.yml
```

## Step 3 — System install (user)

Per-user desktop config: SSH defaults, OPKSSH, GNOME extensions, LazyVim, GUI
flatpaks. Run as your Entra ID user, **without sudo**.

```bash
ansible-playbook -c local -i inventory/hosts playbooks/system_user.yml
```

## Step 4 — Certificate enrolment + Wi-Fi

Requests a device certificate from the internal CA over SCEP and deploys an
EAP-TLS Wi-Fi profile that uses it. The profile is tested automatically and
rolled back if the connection or internet check fails.

For a fresh client, this one-liner installs the prerequisites and runs it:

```bash
_VAULT_PW=<vault-password>

sudo dnf install -y git ansible-core && \
sudo mkdir -p /etc/ansible && \
echo "$_VAULT_PW" | sudo tee /etc/ansible/.vault-pass > /dev/null && \
sudo chmod 600 /etc/ansible/.vault-pass && \
sudo ansible-pull -U https://github.com/martincarlsson-net/zc-cert.git playbooks/scep_enroll.yml
```

The vault password file at `/etc/ansible/.vault-pass` is picked up automatically
via `ansible.cfg`, so re-runs are just:

```bash
sudo ansible-pull -U https://github.com/martincarlsson-net/zc-cert.git playbooks/scep_enroll.yml
```

## Vault values

Organisation-specific and sensitive values live in
`playbooks/group_vars/all/vault.yml` (ansible-vault encrypted) so nothing
identifying is committed to this public repo. In addition to the SCEP/Wi-Fi
values, the workstation playbooks require:

| Key | Used by | Description |
|-----|---------|-------------|
| `entra_domains`    | step 1 | List of primary Entra ID domain(s), e.g. `["example.com"]` |
| `hostname_prefix`  | step 2 | Site/org code prefixed to the product serial as the hostname |
| `opkssh_tenant_id` | step 3 | Azure tenant GUID for the OPKSSH OIDC issuer |
| `opkssh_client_id` | step 3 | Azure app client ID (GUID) for OPKSSH |

Edit the vault to add or change them:

```bash
ansible-vault edit playbooks/group_vars/all/vault.yml
```

## Re-running

The playbooks are idempotent. For step 4, if a valid certificate already
exists, enrolment is skipped and the Wi-Fi profile is recreated and retested.

## NVIDIA drivers (optional)

```bash
sudo dnf install -y https://download1.rpmfusion.org/free/fedora/rpmfusion-free-release-$(rpm -E %fedora).noarch.rpm
sudo dnf install -y https://download1.rpmfusion.org/nonfree/fedora/rpmfusion-nonfree-release-$(rpm -E %fedora).noarch.rpm
sudo dnf install -y akmod-nvidia xorg-x11-drv-nvidia-cuda
```

Reboot after installing.
