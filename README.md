# zc-cert

Automates certificate enrolment and 802.1X/EAP-TLS Wi-Fi authentication on Linux clients using an internal PKI and RADIUS infrastructure.

## How it works

1. The client machine pulls this playbook from GitHub and runs it locally — no central Ansible server required.
2. A device certificate is requested from the internal CA over SCEP and stored on the client.
3. A Wi-Fi profile is created using the issued certificate for EAP-TLS authentication.
4. The profile is tested automatically. If the connection or internet check fails, the profile is rolled back and the client returns to its previous network.
5. A notification is sent with the result.

## Requirements

- Fedora Linux (other RHEL-family distros should work with minor adjustments)
- `git` and `ansible-core` installed on the client
- Vault password obtained out-of-band from your administrator

## Enrol a new client

```bash
_VAULT_PW=<vault-password>

sudo dnf install -y git ansible-core && \
echo "$_VAULT_PW" | sudo tee /root/.vault-pass > /dev/null && \
sudo chmod 600 /root/.vault-pass && \
sudo ansible-pull -U https://github.com/martincarlsson-net/zc-cert.git playbooks/scep_enroll.yml --vault-password-file /root/.vault-pass
```

## Re-running

The playbook is idempotent. If a valid certificate already exists, enrolment is skipped. The Wi-Fi profile is always recreated and retested.
