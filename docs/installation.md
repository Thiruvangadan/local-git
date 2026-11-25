# Installation Guide

This document explains the step-by-step process of deploying a self-hosted Git platform using Gitea on an Ubuntu virtual machine running inside VMware Workstation.

## 1. Environment and Prerequisites

### Project Environment

| Component | Configuration |
|---|---|
| Host Operating System | Windows |
| Virtualization Platform | VMware Workstation |
| Server Operating System | Ubuntu Linux 64-bit |
| Server Architecture | x86_64 / AMD64 |
| Git Platform | Gitea v1.23.4 |
| Database | SQLite3 |
| Web Interface Port | 3000 |
| Gitea SSH Port | 2222 |
| OpenSSH Port | 22 |

### Prerequisites

- Windows computer with VMware Workstation installed.
- Ubuntu ISO image (64-bit).
- Internet connectivity.
- Basic Linux terminal knowledge.
- Git installed on the Windows host.
- SSH client available on Windows.

### Why These Technologies?

- **VMware Workstation:** Used to create and run an Ubuntu virtual machine without replacing the existing Windows operating system.
- **Ubuntu:** Acts as the Linux server that hosts the Gitea application.
- **Gitea:** Provides a self-hosted platform for managing Git repositories through a web interface.
- **SQLite3:** Stores Gitea application data in a lightweight local database.
- **Git:** Used to manage version control and interact with repositories hosted on Gitea.
- **SSH:** Provides secure authentication and communication between the Windows client and Gitea server.

## 2. VMware Network Configuration

### 2.1 Network Mode

The Ubuntu virtual machine was configured to use VMware NAT (Network Address Translation) networking through the VMnet8 virtual network adapter.

### Why NAT?

NAT allows the virtual machine to access external networks using the host machine's network connection. It also allows the Windows host to communicate with the Ubuntu VM through the virtual network.

This configuration is suitable for a local self-hosted Git platform because the Gitea server can be accessed from the Windows host without requiring a separate physical server.

### 2.2 Network Communication

The communication between the components works as follows:

1. Windows acts as the Git client.
2. Ubuntu runs the Gitea server.
3. VMware provides the virtual network connection between Windows and Ubuntu.
4. The Gitea web interface is accessed through port 3000.
5. Git operations use Gitea's built-in SSH server on port 2222.
6. Ubuntu's OpenSSH service uses port 22 for Linux server administration.

### 2.3 Port Configuration

| Port | Service | Purpose |
|---|---|---|
| 22 | OpenSSH | Remote Ubuntu administration |
| 3000 | Gitea HTTP | Accessing the Gitea web interface |
| 2222 | Gitea SSH | Git clone, push and pull operations |

**Note:** Port 22 and port 2222 serve different purposes. Gitea's built-in SSH server runs separately from Ubuntu's OpenSSH service.

### 2.4 NAT vs Bridged Networking

- **NAT:** Allows the VM to share the host's network connection. Suitable for host-to-VM access in this project.
- **Bridged:** Connects the VM directly to the physical network, allowing other devices on the LAN to access it when firewall rules permit.

NAT was selected to keep this project within the local VMware environment.

## 3. Ubuntu Server Preparation

After configuring the Ubuntu virtual machine, the next step is to prepare the operating system for hosting Gitea.

### 3.1 Update Package Lists

```bash
sudo apt update
```

**Why do we use it?**

- Refreshes the local package index.
- Retrieves information about available package versions from configured repositories.
- Helps Ubuntu identify available updates.

### 3.2 Upgrade Installed Packages

```bash
sudo apt upgrade -y
```

**Why do we use it?**

- Installs available updates for existing packages.
- Helps keep the operating system updated.
- The `-y` option automatically confirms installation prompts.

### 3.3 Install Required Utilities

```bash
sudo apt install wget git curl openssh-server sqlite3 -y
```

**Why do we install these packages?**

| Package | Purpose |
|---|---|
| wget | Downloads files from URLs through the terminal |
| git | Provides Git version control commands |
| curl | Transfers data to and from URLs; useful for testing HTTP connections |
| openssh-server | Enables SSH access to the Ubuntu machine |
| sqlite3 | Provides command-line access to SQLite databases |

### 3.4 Enable OpenSSH

```bash
sudo systemctl enable ssh
sudo systemctl start ssh
```

**Why do we use these commands?**

- `systemctl` is used to manage services through systemd.
- `enable` configures SSH to start automatically during system boot.
- `start` starts the SSH service immediately.

Check the service status:

```bash
sudo systemctl status ssh
```

The service should show as active (running).

**Note:** Ubuntu OpenSSH uses port 22 in this project. Gitea's built-in SSH server will use port 2222 for Git operations.
