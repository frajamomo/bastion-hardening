# Bastion Hardening

Ansible playbook for hardening Linux bastion hosts. Supports Fedora laptops and Raspberry Pi (Debian/Raspbian) servers.

## Roles

| Role | Description |
|------|-------------|
| `bootstrap` | Validates connectivity, SSH key auth, and sudo access |
| `ssh_hardening` | Disables password/root login, restricts allowed users, hardens SSH config |
| `firewall` | Configures firewalld (Fedora) or iptables (Debian) with allowlisted ports |
| `kernel_hardening` | Applies sysctl security tunables (network stack, filesystem, kernel self-protection) |
| `selinux` | Ensures SELinux is in enforcing mode (RedHat family only) |
| `fail2ban` | Installs and configures fail2ban for SSH brute-force protection |
| `auditd` | Configures audit rules for file access, privilege escalation, and system calls |
| `auto_updates` | Enables automatic security updates (dnf-automatic or unattended-upgrades) |
| `aide` | Installs and configures AIDE file integrity monitoring with scheduled checks |
| `additional_hardening` | Password quality, shell timeout, cron/at restrictions, su access, security tools |

## Host Groups

- **laptops** — Fedora workstations (firewalld, SELinux, GRUB hardening)
- **servers** — Raspberry Pi / Debian hosts (iptables, Docker port allowlisting, OpenVPN NAT)

## Prerequisites

- Ansible 2.14+
- SSH key-based access to target hosts
- Sudo privileges on target hosts (passwordless)
- `ansible-vault` password file at `~/.ansible-vault.txt`

## Setup

1. Clone the repository:
   ```bash
   git clone https://github.com/frajamomo/bastion-hardening.git
   cd bastion-hardening
   ```

2. Create and encrypt the vault file:
   ```bash
   cp inventory/group_vars/all/vault.yml.example inventory/group_vars/all/vault.yml
   vi inventory/group_vars/all/vault.yml    # fill in your values
   ansible-vault encrypt inventory/group_vars/all/vault.yml
   ```

3. Create a vault password file:
   ```bash
   echo 'your-vault-password' > ~/.ansible-vault.txt
   chmod 600 ~/.ansible-vault.txt
   ```

## Usage

Run the full playbook (bootstrap + hardening) against all hosts:
```bash
ansible-playbook playbooks/site.yml --vault-password-file ~/.ansible-vault.txt
```

Run hardening only:
```bash
ansible-playbook playbooks/harden.yml --vault-password-file ~/.ansible-vault.txt
```

Target a single host:
```bash
ansible-playbook playbooks/harden.yml --vault-password-file ~/.ansible-vault.txt --limit raspberrypi
```

Run specific roles:
```bash
ansible-playbook playbooks/harden.yml --vault-password-file ~/.ansible-vault.txt --tags ssh,firewall
```

## Project Structure

```
bastion-hardening/
├── ansible.cfg
├── inventory/
│   ├── hosts.yml
│   └── group_vars/
│       ├── all/
│       │   ├── all.yml              # Shared variables
│       │   ├── vault.yml            # Encrypted secrets (gitignored)
│       │   └── vault.yml.example    # Template for vault.yml
│       ├── laptops.yml              # Fedora-specific variables
│       └── servers.yml              # Raspberry Pi-specific variables
├── playbooks/
│   ├── site.yml                     # Bootstrap + hardening
│   ├── bootstrap.yml                # Connectivity validation
│   └── harden.yml                   # All hardening roles
└── roles/
    ├── additional_hardening/
    ├── aide/
    ├── auditd/
    ├── auto_updates/
    ├── bootstrap/
    ├── fail2ban/
    ├── firewall/
    ├── kernel_hardening/
    ├── selinux/
    └── ssh_hardening/
```
