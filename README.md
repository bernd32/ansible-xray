# VPS & Xray Core Setup (Ansible)

An automated Ansible playbook for initial configuration and deploying of [Xray-core](https://github.com/XTLS/Xray-core) on a VPS, with a VLESS + REALITY configuration.

## Prerequisites
- **Ansible Core** ≥ 2.14
- **Target OS:** Debian / Ubuntu (systemd-based)
- **Architecture:** `x86_64`  
- **Access:** SSH root access

## Setup & Configuration
- Edit inventory/inventory.ini with your VPS details
- Create the encrypted variables file: 
```
ansible-vault create group_vars/vps_servers.yml
```
Insert your configuration:
```yaml
xray_port: 443
xray_config_path: "/usr/local/etc/xray"

# VLESS / REALITY parameters
client_id: "your-uuid-here"
domain: "example"
private_key: "your-reality-private-key"
short_id: "your-short-id"
```
- Run the Playbook: 
```
ansible-playbook xray_setup.yml --ask-vault-pass
```