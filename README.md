# Multi-Platform Linux Automation Lab

This repository is a collection of Infrastructure-as-Code (IaC) projects using Ansible to manage diverse Linux environments, ranging from enterprise-grade Red Hat systems to Debian-based distributions.



## Supported Environments
- **Enterprise Linux (RHEL/Rocky 9):** Focused on high-availability, security hardening, and LAMP stack deployment.
- **Debian Systems (Tested on Ubuntu 22.04/24.04):** Focused on developer environments and modern web services.

##  Project Highlights


### 1. RHEL Web & Database Deployment (`rhel-academy/`)
* **Goal:** Deploy a secure, authenticated web server environment with automated verification.
* **Key Skills:** * **Security & Hardening:** Configuring `mod_ssl` for HTTPS, setting up Basic Authentication (`.htaccess`/`htpasswd`), and managing file ownership/permissions.
    * **Service Management:** Orchestrating `httpd` and `firewalld` services with immediate and permanent port opening.
    * **Dynamic Configuration:** Generating content using **Ansible Facts** to reflect host-specific data.
    * **Automated Testing:** Verifying deployment success from the control node using the `uri` module to test authentication and HTTP 200 status codes.

### 2. Rocky Linux System Hardening (`homelab-rocky/`)
* **Goal:** Applying enterprise-grade security baselines to RHEL-compatible environments.
* **Why Rocky?:** As a binary-compatible downstream of RHEL, Rocky Linux utilizes **SELinux** by default. This project demonstrates the ability to configure granular access controls (MAC) rather than disabling them. This is a critical requirement for government and financial sector roles.
* **Key Skills:** * Managing SELinux Booleans (`httpd_can_network_connect`).
    * Persistent File Contexts (`semanage fcontext`).
    * Automated Audit Log analysis.

### 3. Ubuntu Workstation Continuity (`homelab-ubuntu/`)
* **Goal:** Eliminating configuration drift by codifying the development environment. This playbook transforms a fresh Ubuntu installation into a production-ready workstation in under 15 minutes.
* **Justification:**
    * **Disaster Recovery:** Removes the downtime associated with OS re-installation or hardware failure.
    * **State Management:** Enforces consistent versions for tools (Docker, Python, Node.js) across machines, preventing "it works on my machine" errors.
    * **Repository Management:** Automates the handling of third-party PPAs, GPG keys, and Snap/Flatpak preferences that are tedious to manage manually.
* **Key Skills:** `apt_repository` management, dotfile synchronization, and local user configuration.

These automations are developed and tested on a local workstation environment to ensure performance and reproducibility.

* **System:** Dell Mobile Workstation (Precision series)
* **Processor:** Intel Core i7-11850H (8 Cores @ 2.50GHz)
* **Memory:** 32GB RAM (Simulating multi-node production setups)
* **Graphics:** NVIDIA T1200 Laptop GPU + Intel UHD (Hybrid Graphics)
* **Storage:** 1TB NVMe SSD

##  Usage
All playbooks are structured to be run via `ansible-navigator` or `ansible-playbook`.

```bash
# Example: Running the RHEL web stack
ansible-navigator run -m stdout rhel-academy/internet.yml

