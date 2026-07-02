# Ansible Role — Tailscale

Installs Tailscale from official repositories, configures the daemon, and connects the node to your tailnet using an auth key. Supports Debian/Ubuntu and RHEL/Fedora.

## Requirements

- Ansible ≥ 2.9
- Target hosts running Debian/Ubuntu or RHEL/Fedora
- A Tailscale auth key ([generate one here](https://login.tailscale.com/admin/settings/keys))

## Quick Start

```bash
# 1. Add hosts to inventory
vim inventory.ini

# 2. Run the playbook with your auth key
ansible-playbook -i inventory.ini site.yaml \
  -e tailscale_authkey=tskey-auth-XXXXXXXXXXXX
```

> **Tip:** Store `tailscale_authkey` in an Ansible Vault file instead of passing it on the command line.

## Role Variables

### Defaults (`defaults/main.yaml`)

| Variable | Default | Description |
|---|---|---|
| `tailscale_state` | `present` | `present` to install, `absent` to uninstall |
| `tailscale_authkey` | `""` | Auth key for headless login — **use Ansible Vault** |
| `tailscale_up_skip` | `false` | Skip `tailscale up` (install and configure only) |
| `tailscale_hostname` | `{{ inventory_hostname }}` | Hostname registered in the tailnet |
| `tailscale_extra_args` | `""` | Extra arguments for `tailscale up` |
| `tailscale_force_reauth` | `false` | Force re-authentication even if already connected |

### Common `tailscale_extra_args` Examples

```yaml
# Enable Tailscale SSH
tailscale_extra_args: "--ssh"

# Advertise as exit node
tailscale_extra_args: "--advertise-exit-node"

# Accept subnet routes and enable SSH
tailscale_extra_args: "--accept-routes --ssh"

# Apply ACL tags
tailscale_extra_args: "--advertise-tags=tag:server"
```

## Tags

| Tag | Controls |
|---|---|
| `install` | Repository setup and package installation |
| `configure` | Enable and start tailscaled service |
| `connect` | Authentication and tailnet connection |
| `uninstall` | Full removal (disconnect, uninstall, clean repos) |

```bash
# Install only, don't connect
ansible-playbook -i inventory.ini site.yaml --tags install,configure

# Re-authenticate an already installed node
ansible-playbook -i inventory.ini site.yaml --tags connect \
  -e tailscale_force_reauth=true \
  -e tailscale_authkey=tskey-auth-XXXXXXXXXXXX
```

## Uninstalling

Set `tailscale_state` to `absent`:

```bash
ansible-playbook -i inventory.ini site.yaml -e tailscale_state=absent
```

This will:
1. Disconnect from the tailnet (`tailscale down`)
2. Logout the node (`tailscale logout`)
3. Stop and disable the `tailscaled` service
4. Remove the package and repository

## File Structure

```
roles/tailscale/
├── defaults/
│   └── main.yaml          ← role defaults
└── tasks/
    ├── main.yaml           ← orchestrator
    ├── install.yaml        ← add repo + install package
    ├── configure.yaml      ← enable/start tailscaled
    ├── connect.yaml        ← tailscale up (idempotent)
    └── uninstall.yaml      ← full teardown
```

## Task Execution Order

### Install (`tailscale_state: present`)

1. **Install** — add Tailscale GPG key, repository, and package
2. **Configure** — enable and start `tailscaled` systemd service
3. **Connect** — check status, authenticate with auth key if not connected

### Uninstall (`tailscale_state: absent`)

1. **Disconnect** — `tailscale down`
2. **Logout** — `tailscale logout`
3. **Stop service** — disable and stop `tailscaled`
4. **Remove** — uninstall package, remove repo and GPG key

## Supported Platforms

| OS Family | Tested On |
|---|---|
| Debian | Ubuntu 20.04+, Debian 11+ |
| RedHat | RHEL 8+, Fedora 38+, CentOS Stream 9+ |

## License

See the project-level [LICENSE](../../LICENSE) if applicable.
