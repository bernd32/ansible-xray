# Ansible Xray Provisioning

An Ansible playbook for provisioning a basic [Xray](https://github.com/XTLS/Xray-core) VLESS + REALITY setup on a server and a client(s).

It installs Xray, generates the required identity material, renders JSON configs from Jinja2 templates, and enables the systemd service on both hosts.

## What it provisions

* **Server**: Xray inbound on `443/tcp` using `vless` + `reality`
* **Client**: local SOCKS inbound on `127.0.0.1:10808`
* **Transport**: TCP + REALITY
* **Auth**: VLESS UUID + X25519 key material + shortId

## Requirements

* Ansible 2.14+
* SSH access to both hosts
* Root or sudo access on the managed nodes
* Debian/Ubuntu-like distro on both server and client
* Python 3 on the remote hosts

# Quick Start

## 1. Prepare the hosts

You need:

* One Linux VPS/server
* One or more Linux clients
* SSH access to all hosts
* A working SSH keypair on your local machine

Example:

```bash
~/.ssh/id_ed25519
~/.ssh/id_ed25519.pub
```

Copy your public key to every target host:

```bash
ssh-copy-id admin@YOUR_VPS_IP
ssh-copy-id root@YOUR_CLIENT_IP
```

Verify SSH access works without a password:

```bash
ssh admin@YOUR_VPS_IP
ssh root@YOUR_CLIENT_IP
```

---

## 2. Clone the repository

```bash
git clone https://github.com/YOUR_USERNAME/ansible-xray.git
cd ansible-xray
```

---

## 3. Install required Ansible collections

The playbook uses the `community.general` collection for UFW management.

Install it:

```bash
ansible-galaxy collection install community.general
```

---

## 4. Configure inventory

Edit `inventory.ini`.

Example:

```ini
[server]
vps ansible_host=203.0.113.10 ansible_user=admin

[clients]
client_1 ansible_host=192.168.1.50 ansible_user=root

[all:vars]
ansible_python_interpreter=/usr/bin/python3
ansible_ssh_private_key_file=~/.ssh/id_ed25519
```

Notes:

* `server` must contain exactly one host
* `clients` may contain multiple hosts
* `ansible_user` must have sudo privileges

---

## 5. Adjust server settings

Open `playbook.yml` and review:

```yaml
server_name: "microsoft.com"
xray_port: 443
```

Choose a realistic SNI target for `server_name`.

---

## 6. Run the playbook

Execute:

```bash
ansible-playbook -i inventory.ini playbook.yml
```

The playbook will:

* install Xray (on both client and server)
* generate REALITY keys
* generate UUID and shortId
* deploy configs
* enable/start the Xray service
* configure UFW on the server

---

## 7. Verify Xray service

On the server:

```bash
systemctl status xray
```

On the client:

```bash
systemctl status xray
```

Check logs:

```bash
journalctl -u xray -f
```

Check if proxy is working: 

```bash
curl --proxy socks5h://127.0.0.1:10808 https://ifconfig.me
```

# 8. Configure the firewall (optional)

Open the TCP 443 port on a server, e.g.: 

```bash
 ufw allow 443/tcp
```


# Additional information 

Specify a tag option to limit the configuration to VPS/Server or client part only. 

To install and configure Xray only on a VPS/server (skip the client part): 

```bash
ansible-playbook -i inventory.ini playbook.yml --tags server
```

To install and configure Xray only on a client (skip the server part): 

```bash
ansible-playbook -i inventory.ini playbook.yml --tags client
```