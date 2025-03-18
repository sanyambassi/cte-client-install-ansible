# Ansible CTE Client Installation Demo

This repository contains an Ansible playbook to automate the installation of the CipherTrust Transparent Encryption (CTE) client on Red Hat, Ubuntu, and Windows systems. It leverages a role-based structure for modularity, supports multi-OS environments, and secures sensitive credentials using Ansible Vault.

## Table of Contents

- [Overview](#overview)
- [Prerequisites](#prerequisites)
- [Directory Structure](#directory-structure)
- [Setup Instructions](#setup-instructions)
- [Usage](#usage)
- [Inventory Configuration](#inventory-configuration)
- [Contributing](#contributing)
- [License](#license)

## Overview

This Ansible playbook automates the deployment of the CTE client across heterogeneous operating systems:

- **Red Hat**: Installs prerequisites (libnsl2) and the CTE agent using an answerfile.
- **Ubuntu**: Installs prerequisites (libnsl2) and the CTE agent using an answerfile.
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

- **Ansible**: Version 2.9 or later (install via pip install ansible or your package manager).
- **Python**: Required on the control node (where Ansible runs) and Linux targets (for SSH).
- **WinRM**: Python pywinrm module for Windows hosts (pip install pywinrm).

### Target Host Requirements

- **Red Hat/Ubuntu**:
  - SSH server running (sudo systemctl enable --now sshd).
  - User with sudo privileges (password or public key authentication).
  - Network access to the control node and CipherTrust Manager.
- **Windows**:
  - WinRM configured for HTTP (port 5985) or HTTPS (port 5986) with NTLM authentication.
  - Administrator account with known password.
  - Firewall rule allowing connectivity port 5985 or 5986 , inbound to windows hosts

### Files Needed

- CTE installer binaries: For example. 
  - vee-fs-7.7.0-100-rh9-x86_64.bin (Red Hat)
  - vee-fs-7.7.0-100-ubuntu22-x86_64.bin (Ubuntu)
  - vee-fs-7.7.0-104-win64.msi (Windows)
- Place these in the files/ directory (not included in this repo due to licensing).

### CipherTrust Manager resources
- Create a client profile under Access Management --> Client Profiles on the CipherTrust manager and save the ID.
- Create a CTE profile under Transparent Encryption --> Setttings --> Profiles on the CipherTrust manager and save the name. 

## Directory Structure
```bash
cte-client-install-ansible/
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
```

**Note**: The files/ directory is a placeholder. Users must supply their own CTE installers.

## Setup Instructions

### 1. Clone the Repository
```bash
git clone https://github.com/sanyambassi/cte-client-install-ansible.git
```
### 2. Configure the Inventory

Edit inventory.ini to match your environment:

```
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
windows-host1 ansible_host=<windows-ip> ansible_connection=winrm ansible_winrm_transport=ntlm

[windows:vars]
ansible_user={{ ansible_winrm_user }}
ansible_password={{ ansible_winrm_password }}
ansible_port=5985
ansible_winrm_server_cert_validation=ignore

[cte_clients:children]
redhat
ubuntu
windows
```
- Replace \<redhat-ip\>, \<ubuntu-ip\>, \<windows-ip\> with your target host IPs (e.g., 10.10.10.18 for Windows).
- Update ansible_user and ansible_ssh_private_key_file for Linux hosts.
- Ensure the Windows host IP matches your setup.

### 3. Configure Non-Sensitive Variables

Edit vars/cte_config.yml:

```
server_hostname: "your-ciphertrust-manager-host"
client_profile_default: "id-of-the-client-profile"
cte_client_profile_default: "name-of-cte-client-profile"
lifetime: "1d"
max_clients: 1
name_prefix: "ansible_token"
enable_ldt: 1
host_desc_default: "Client Installed with Ansible"
```

### 4. Configure Sensitive Variables

Create or edit `vars/vault.yml` with encrypted credentials:
```bash
ansible-vault create vars/vault.yml
```
- Enter a password and confirm password when prompted.
- In the editor, add the following contents:
```
cm_username: "cm-username"
cm_password: "cm-password"
cm_domain: "cte-domain"
ansible_become_password: "your-sudo-password"
ansible_winrm_user: "your-administrator-user"
ansible_winrm_password: "your-windows-password"
```
- Note: if using a domain user for Windows hosts, escapte the \ separator. For example, "domain\\\username"

### 5. Add Installer Files

Place the CTE installers in files/: For example:

- vee-fs-7.7.0-100-rh9-x86_64.bin
- vee-fs-7.7.0-100-ubuntu22-x86_64.bin
- vee-fs-7.7.0-104-win64.msi

These can be downloaded from the Thales support portal. Update the **playbook.yml** with these filenames.

### 6. Configure Target Hosts

- **Linux (Red Hat/Ubuntu)**:
  - Enable SSH and allow port 22 in the firewall.
  - Ensure the user has sudo privileges as the CTE client can only be installed as the root user.
- **Windows**:
  - Configure WinRM for NTLM and ensure the user has Administrator rights on the system. 

## Usage

### Run the Playbook

From the `cte-client-install-ansible/` directory:
```bash
ansible-playbook -i inventory.ini playbook.yml --ask-vault-pass -v
```
- where
```
-i inventory.ini: Specifies the inventory file.
--ask-vault-pass: Prompts for the vault password.
-v: Verbose output for debugging (or use -vvvv).
```

#### What It Does
- Authenticates with CipherTrust Manager. 
- Creates a registration token for CTE clients.
- Gathers OS facts to determine the target system.
- Copies the appropriate installer and answerfile (Linux only).
- Installs the CTE client using OS-specific roles.
- Deletes the installer and the answerfile.

**Example Output**
```
PLAY [Install CipherTrust Transparent Encryption (CTE) Client]
TASK [Gathering Facts] *****************************************
ok: [windows-host1]
TASK [Authenticate to CipherTrust Manager and obtain JWT token]
ok: [windows-host1 -> localhost]
...

## Contributing

If you would like to contribute to this project, please open an issue or submit a pull request with your changes.

## License

This project is licensed under the MIT License. See the [LICENSE](LICENSE) file for details.
