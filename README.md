# Ansible Bootstrap Playbook

A personal Ansible project to automate post-installation/bootstrap system configuration on Ubuntu and Debian machines.

## Project Structure

```text
ansible-bootstrap/
├── ansible-bootstrap.yml  # Main entrypoint playbook
├── README.md              # Project documentation
├── vars/                  # Variable files (modular package definition)
│   ├── common.yml         # Packages installed on all PC types
│   ├── office-pc.yml      # Packages for office PCs
│   ├── gaming-ai.yml      # Packages for gaming/AI PCs
│   └── home-pc.yml        # Packages for home/persoanl laptop PCs
└── tasks/                 # Custom tasks per PC type
    ├── office-pc.yml
    ├── gaming-ai.yml
    └── home-pc.yml
```

## Features

- **Input Validation**: Playbook asserts that the specified `pc_type` is defined and valid. It will fail immediately if not specified.
- **Dynamic Configuration**: Loads common and PC-specific packages, merging them dynamically.
- **Modularity**: Keeps variables and custom tasks decoupled from the main play.
- **Privilege Separation**: Package installation runs as root (`become: yes`), while configuration (Starship, Vimrc) runs as standard user (`become: no`).

## Supported PC Types

1. `home-pc`
2. `office-pc`
3. `gaming-ai`

## Requirements

> **⚠️ WARNING**: This playbook is developed, tested, and validated **ONLY on Ubuntu 26.04 (Plucky/Series 16)**. Running on other versions of Ubuntu or Debian may lead to package resolution or compatibility issues.

- **OS**: Ubuntu 26.04.
- **Ansible** and standard modules installed:
  ```bash
  sudo apt update && sudo apt install -y ansible
  ansible-galaxy collection install community.general
  ```

## Usage

You MUST specify the `pc_type` variable when running the playbook.

> **⚠️ CRITICAL NOTE FOR UBUNTU 26.04 (and modern sudo 1.9.16+)**: Due to a known issue with Ansible's local connection privilege escalation parser on systems with `sudo` >= 1.9.16 (where prompt wrapping breaks Ansible's internal regex), running with `--ask-become-pass` (or `-K`) will hang and timeout.
>
> **Do NOT run the entire playbook directly with `sudo` (e.g. `sudo ansible-playbook ...`)**, as this will run user-level tasks (Vimrc, Starship, LazyVim) as root, corrupting file permissions in your home directory (`~`).
>
> Instead, use the temporary passwordless `sudo` workaround to let Ansible escalate privileges seamlessly:

### Workaround & Execution Steps

1. **Allow temporary passwordless sudo**:
   ```bash
   echo "$USER ALL=(ALL) NOPASSWD:ALL" | sudo tee /etc/sudoers.d/ansible-temp
   ```

2. **Run the playbook (without `-K` or `--ask-become-pass`)**:
   - For Home PC:
     ```bash
     ansible-playbook ansible-bootstrap.yml -e "pc_type=home-pc"
     ```
   - For Office PC:
     ```bash
     ansible-playbook ansible-bootstrap.yml -e "pc_type=office-pc"
     ```
   - For Gaming AI:
     ```bash
     ansible-playbook ansible-bootstrap.yml -e "pc_type=gaming-ai"
     ```

3. **Clean up the temporary file**:
   ```bash
   sudo rm -f /etc/sudoers.d/ansible-temp
   ```

---
## Docker Services
1. **GPU Acceleration (NVIDIA)**:
   If you have an NVIDIA GPU, install the **NVIDIA Container Toolkit** on the host:
   ```bash
   sudo apt-get update && sudo apt-get install -y nvidia-container-toolkit
   sudo systemctl restart docker

2. **Generating Custom Secret Keys**:
   For security, generate unique 32-byte hexadecimal secrets for Arcane:
   ```bash
   docker run --rm ghcr.io/getarcaneapp/arcane:latest /app/arcane generate secret
   ```

---

## 🔑 Crucial Post-Installation Manual Steps

Please ensure you **manually copy** your SSH keys, SSH config, and AI agent skills from your secure backup, as these are not automated by the playbook for security and privacy reasons.
