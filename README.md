# Ansible CTE Client Installation Demo

This repository contains an Ansible playbook to automate the installation of the CipherTrust Transparent Encryption (CTE) client on Red Hat, Ubuntu, and Windows systems. It leverages a role-based structure for modularity, supports multi-OS environments, and secures sensitive credentials using Ansible Vault.

## Table of Contents

- [Overview](#overview)
- [Prerequisites](#prerequisites)
- [Directory Structure](#directory-structure)
- [Setup Instructions](#setup-instructions)
- [Usage](#usage)
- [Inventory Configuration](#inventory-configuration)
- [Troubleshooting](#troubleshooting)
- [License](#license)

## Overview

This Ansible playbook automates the deployment of the CTE client across heterogeneous operating systems:

- **Red Hat**: Installs prerequisites (`libnsl2`) and the CTE agent using an answerfile.
- **Ubuntu**: Installs prerequisites (`libnsl2`) and the CTE agent using an answerfile.
- **Windows**: Installs the CTE agent via MSI with reboot handling.

Key features:

- Authenticates with CipherTrust Manager to obtain a JWT token.
- Generates a registration token for CTE clients.
- Uses roles to separate OS-specific tasks.
- Secures sensitive data (e.g., passwords) with Ansible Vault.
- Cleans up temporary files post-installation.

## Prerequisites

Before using this playbook, ensure the following:

### Software Requirements

- **Ansible**: Version 2.9 or later (install via `pip install ansible` or your package manager).
- **Python**: Required on the control node (where Ansible runs) and Linux targets (for SSH).
- **WinRM**: Python `pywinrm` module for Windows hosts (`pip install pywinrm`).

### Target Host Requirements

- **Red Hat/Ubuntu**:
  - SSH server running (`sudo systemctl enable --now sshd`).
  - User with sudo privileges (password or passwordless).
  - Network access to the control node and CipherTrust Manager.
- **Windows**:
  - WinRM configured for HTTP (port 5985) with NTLM authentication.
  - Administrator account with known password.
  - Firewall rule allowing port 5985 (`netsh advfirewall firewall add rule name="WinRM HTTP" dir=in action=allow protocol=TCP localport=5985`).

### Files Needed

- CTE installer binaries:
  - `vee-fs-7.7.0-100-rh9-x86_64.bin` (Red Hat)
  - `vee-fs-7.7.0-100-ubuntu22-x86_64.bin` (Ubuntu)
  - `vee-fs-7.7.0-104-win64.msi` (Windows)
- Place these in the `files/` directory (not included in this repo due to licensing).

## Directory Structure
cte_install/
├── README.md              # This documentation file
├── playbook.yml           # Main playbook orchestrating the installation
├── inventory.ini          # Inventory file defining target hosts
├── vars/
│   ├── cte_config.yml     # Non-sensitive configuration variables
│   └── vault.yml          # Encrypted sensitive variables (Ansible Vault)
├── files/
│   ├── vee-fs-7.7.0-100-rh9-x86_64.bin    # Red Hat installer (placeholder)
│   ├── vee-fs-7.7.0-100-ubuntu22-x86_64.bin  # Ubuntu installer (placeholder)
│   └── vee-fs-7.7.0-104-win64.msi         # Windows installer (placeholder)
└── roles/
├── redhat/
│   └── tasks/
│       └── main.yml   # Red Hat-specific installation tasks
├── ubuntu/
│   └── tasks/
│       └── main.yml   # Ubuntu-specific installation tasks
└── windows/
└── tasks/
└── main.yml   # Windows-specific installation tasks


**Note**: The `files/` directory is a placeholder. Users must supply their own CTE installers.

## Setup Instructions

### 1. Clone the Repository

If downloading manually from GitHub:

- Click "Code" > "Download ZIP".
- Extract to a local directory (e.g., `cte_install/`).

### 2. Configure the Inventory

Edit `inventory.ini` to match your environment:

```ini
[redhat]
redhat-host1 ansible_host=<redhat-ip> ansible_user=<redhat-user> ansible_ssh_private_key_file=/path/to/redhat_key

[redhat:vars]
ansible_become=true
ansible_become_method=sudo
ansible_become_password={{ ansible_become_password }}

[ubuntu]
ubuntu-host1 ansible_host=<ubuntu-ip> ansible_user=<ubuntu-user> ansible_ssh_private_key_file=/path/to/ubuntu_key

[ubuntu:vars]
ansible_become=true
ansible_become_method=sudo
ansible_become_password={{ ansible_become_password }}

[windows]
windows-host1 ansible_host=<windows-ip> ansible_connection=winrm ansible_winrm_transport=ntlm ansible_port=5985 ansible_winrm_server_cert_validation=ignore

[cte_clients:children]
redhat
ubuntu
windows
```

- Replace <redhat-ip>, <ubuntu-ip>, <windows-ip> with your target host IPs (e.g., 10.10.10.18 for Windows).
- Update ansible_user and ansible_ssh_private_key_file for Linux hosts.
- Ensure the Windows host IP matches your setup.
