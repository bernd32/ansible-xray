# Xray Core Ansible Playbook: Usage Guide

This playbook automates VPS hardening (UFW, SSH) and deploys Xray-core with VLESS/REALITY configurations for both server and client nodes.

## Prerequisites
* Ansible >= 2.14
* `community.general` collection (`ansible-galaxy collection install community.general`)
* Target OS: Debian/Ubuntu (systemd-based, `apt` package manager)
* Target Architecture: `x86_64`  

## Configuration

### 1. Inventory
Edit `inventory/inventory.ini`. Define your server and client nodes under their respective groups.
```ini
[vps_servers]
vps_1 ansible_host=<IP> ansible_user=root

[xray_clients]
client_1 ansible_host=<IP> ansible_user=root

[all:vars]
ansible_python_interpreter=/usr/bin/python3
```

### 2. Variables and Secrets
Create and encrypt group variables using Ansible Vault. Do not commit plaintext secrets.

```bash
ansible-vault create group_vars/all.yml
ansible-vault create group_vars/vps_servers.yml
ansible-vault create group_vars/xray_clients.yml
```

**Required variables:**

`group_vars/all.yml`
```yaml
client_id: "<UUID>"
short_id: "<hex-string>"
domain: "<reality-domain>"
vps_ip: "<server-ip>"
```

`group_vars/vps_servers.yml`
```yaml
xray_mode: server
ssh_port: 22
xray_port: 443
private_key: "<reality-private-key>"
```

`group_vars/xray_clients.yml`
```yaml
xray_mode: client
inbound_port: 10808
outbound_port: 443
public_key: "<reality-public-key>"
```

More info about these options (RU/CN): https://github.com/XTLS/Xray-core/discussions/3518  

## Execution

Run the main playbook. Provide vault and SSH credentials as required by your environment.

```bash
ansible-playbook -i inventory/inventory.ini xray_setup.yml --ask-vault-pass -k
```

### Execution Caveats
* **SSH Port Rotation:** If `ssh_port` in `vps_servers.yml` differs from the default SSH port, the playbook will reconfigure `sshd` and reset the connection. For subsequent runs, you must update `inventory.ini` with `ansible_port=<new_port>` or pass it via CLI (`-e ansible_port=<new_port>`).

## Architecture Notes
* **Role Separation:** `vps-prep` handles OS-level hardening and firewall rules. `xray` handles binary deployment, systemd unit management, and Jinja2 templating.
* **Configuration Templating:** The `xray` role dynamically selects the configuration template based on the `xray_mode` variable (`config_server.json.j2` or `config_client.json.j2`).
* **Idempotency:** Xray binary download and extraction are skipped if the target version matches the currently installed version.
* **Customization:** To support ARM/MIPS architectures or alter default paths, modify `roles/xray/defaults/main.yml`.