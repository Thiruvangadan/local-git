# Tools and Technologies

## Overview

This project uses a combination of virtualization, Linux administration, Git hosting, networking, SSH authentication, and database technologies to build a self-hosted Git platform.

The following tools and technologies were used during implementation.

## Technology Stack

| Technology | Version / Configuration | Purpose |
|---|---|---|
| Windows | Host Operating System | Acts as the primary client machine |
| VMware Workstation Pro | 26H1u1 (Build 25688693) | Runs the Ubuntu virtual machine |
| Ubuntu | Linux x86_64 / AMD64 | Server operating system hosting Gitea |
| Gitea | v1.23.4 | Self-hosted Git repository management platform |
| SQLite3 | SQLite database backend | Stores Gitea application data |
| Git | Git client | Performs repository operations |
| OpenSSH | Ubuntu SSH service | Provides remote Linux administration |
| Gitea Built-in SSH | TCP port 2222 | Handles Git operations over SSH |
| systemd | Ubuntu service manager | Manages Gitea as a background service |
| VMware NAT (VMnet8) | NAT networking | Provides network connectivity between the host and VM |

## Tools and Their Roles

### 1. VMware Workstation Pro

VMware Workstation Pro provides the virtualization environment used to run Ubuntu on the Windows host machine.

It allows the Gitea server to operate inside an isolated virtual machine without requiring a separate physical server.

**Why it was used:**
- Provides a dedicated Linux server environment.
- Allows the virtual machine to run independently of the host operating system.
- Supports NAT networking through VMnet8.
- Makes it easier to experiment with server administration and networking.

### 2. Ubuntu Linux

Ubuntu is the server operating system used to host the Gitea application.

It provides the environment required to install, configure, and manage Gitea and its supporting services.

**Why it was used:**
- Provides a Linux environment for server administration.
- Supports systemd service management.
- Provides SSH and networking utilities.
- Offers package management and command-line administration tools.

### 3. Gitea

Gitea is a lightweight, self-hosted Git hosting platform.

It provides a web interface for managing repositories, users, SSH keys, and Git operations.

**Why it was used:**
- Provides a self-hosted alternative to cloud-based Git hosting.
- Supports repository creation and management.
- Supports SSH-based Git authentication.
- Offers a web interface for browsing repositories and managing users.
- Can run as a standalone application without requiring Docker.

### 4. Git

Git is the distributed version control system used to manage source code and interact with repositories hosted on Gitea.

**Why it was used:**
- Tracks changes to files.
- Supports commits and version history.
- Enables repository cloning and synchronization.
- Provides push and pull operations for remote repositories.

### 5. SQLite3

SQLite3 is the database backend selected during Gitea's initial installation.

It stores Gitea application information in a local database file.

**Why it was used:**
- Lightweight and simple to configure.
- Does not require a separate database server.
- Suitable for a small self-hosted learning environment.
- Keeps database management within the Ubuntu virtual machine.

### 6. OpenSSH

OpenSSH provides secure remote access to the Ubuntu virtual machine.

In this project, Ubuntu's OpenSSH service runs on TCP port `22` and is used for Linux administration.

**Why it was used:**
- Allows remote access to the Ubuntu server.
- Supports SSH-based authentication.
- Separates server administration from Git repository access.

### 7. Gitea Built-in SSH Server

Gitea's built-in SSH server handles SSH-based Git operations independently of Ubuntu's OpenSSH service.

It listens on TCP port `2222` in this implementation.

**Why it was used:**
- Provides SSH access to Gitea repositories.
- Supports public-key authentication through Gitea account settings.
- Handles Git clone, push, and pull operations.
- Operates independently of Ubuntu's SSH administration service.

### 8. systemd

systemd is the service manager used by Ubuntu to manage background services.

Gitea was configured as a systemd service using a custom service unit.

**Why it was used:**
- Starts Gitea as a background service.
- Allows the service to start automatically during system boot.
- Supports service monitoring and management.
- Provides commands to start, stop, restart, and inspect Gitea.

### 9. VMware NAT Networking

VMware NAT networking provides communication between the Windows host, Ubuntu virtual machine, and external networks.

The VMnet8 virtual network adapter is used for this configuration.

**Why it was used:**
- Provides network connectivity to the Ubuntu VM.
- Allows the Windows host to access Gitea services.
- Allows the Ubuntu VM to access external networks through NAT.
- Avoids requiring a separate physical server or network connection.

## Development and Administration Tools

The following utilities were also used during implementation:

| Tool | Purpose |
|---|---|
| PowerShell | Windows SSH key generation and configuration |
| Git Bash | Git commands and SSH-based repository access |
| Command Prompt | Git operations from the Windows command line |
| Windows Web Browser | Accessing the Gitea web interface |
| wget | Downloading the Gitea binary |
| curl | Command-line network and HTTP testing |
| systemctl | Managing the Gitea and SSH services |
| ss | Checking listening network ports |
| chmod | Managing file permissions |
| chown | Managing file ownership |

## Summary

The project combines VMware virtualization, Ubuntu server administration, Gitea, Git, SQLite3, SSH, and systemd to create a lightweight self-hosted Git hosting environment.

Each technology has a specific responsibility, from providing the virtual server environment to managing repositories, authenticating Git clients, storing application data, and ensuring the Gitea service starts automatically.
