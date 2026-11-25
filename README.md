# Self-Hosted Git Platform using Gitea

A lightweight, self-hosted Git hosting platform deployed on an Ubuntu virtual machine using VMware Workstation. Gitea provides a web-based interface for managing Git repositories, users, and collaboration, while SSH handles secure Git operations between the client and server.

## Project Overview

The objective of this project was to understand how Git hosting platforms work by building and managing a self-hosted alternative to GitHub.

The platform runs on an Ubuntu virtual machine and uses Gitea for repository management, SQLite3 for application data, and SSH key-based authentication for Git operations.

This project provided hands-on experience with Linux server administration, virtualization, networking, SSH, Git, and systemd service management.

## Architecture

**Server**
- Ubuntu Linux running inside VMware Workstation
- Gitea v1.23.4 as the Git hosting platform
- SQLite3 database
- Gitea built-in SSH server for Git operations
- OpenSSH Server for remote Ubuntu administration

**Client**
- Windows host with Git installed
- Ed25519 SSH key-based authentication
- Windows SSH client configuration

### Network Ports

| Component | Port | Purpose |
|---|---:|---|
| Ubuntu OpenSSH | 22 | Remote server administration |
| Gitea Web Interface | 3000 | Repository and user management |
| Gitea Built-in SSH | 2222 | Git clone, push, and pull |

The VMware virtual machine uses NAT networking. This allows the Windows host to communicate with the Ubuntu VM while the VM accesses external networks through VMware's NAT connection.

## Technology Stack

| Component | Technology | Purpose |
|---|---|---|
| Virtualization | VMware Workstation | Runs the Ubuntu VM |
| Operating System | Ubuntu Linux x86_64 | Server environment |
| Git Hosting | Gitea v1.23.4 | Self-hosted Git platform |
| Version Control | Git | Source code management |
| Database | SQLite3 | Stores Gitea application data |
| Authentication | SSH / Ed25519 | Key-based authentication |
| Remote Administration | OpenSSH | Secure server access |

## Features

- Self-hosted Git repository management
- Web-based repository interface
- SSH key-based authentication
- Repository cloning, pushing, and pulling
- User and repository management
- Private repository support
- Dedicated Linux system user for running Gitea
- Gitea configured as a systemd service
- Automatic service startup after Ubuntu reboot

## Git Operations

The following operations were tested successfully from the Windows client:

Clone a repository:

```bash
git clone git@gitea:<GITEA_USERNAME>/<REPOSITORY_NAME>.git
```

Check repository status:

```bash
git status
```

Stage and commit changes:

```bash
git add .
git commit -m "Add project files"
```

Push changes:

```bash
git push
```

Pull remote changes:

```bash
git pull
```

The SSH alias `gitea` uses the configured Ubuntu IP address, port 2222, and the selected private key.

## Challenges and Solutions

### SSH Port Conflict

**Problem:** Ubuntu OpenSSH was already using port 22, which would conflict with Gitea's built-in SSH server if both attempted to use the same port.

**Solution:** Enabled Gitea's built-in SSH server and configured it to use port 2222, keeping Ubuntu OpenSSH on port 22.

### Existing SSH Key

**Problem:** The default Ed25519 SSH key filename was already being used for another service.

**Solution:** Generated a separate key pair named `id_ed25519_gitea` and registered its public key in Gitea.

### SSH Client Configuration

**Problem:** Git needed to use the correct SSH key, username, server address, and port when connecting to Gitea.

**Solution:** Configured a Windows SSH client alias named `gitea` to specify the required connection settings.

### Client-to-Server Git Connectivity

**Problem:** Git operations needed to work between the Windows client and the Ubuntu virtual machine.

**Solution:** Configured SSH authentication and verified repository cloning, pushing, and pulling.

## Documentation

Detailed implementation steps and supporting information are available in the `docs/` directory.

| File | Description |
|---|---|
| [Installation Guide](docs/installation.md) | Step-by-step Gitea installation and configuration |
| [Architecture](docs/architecture.md) | System components and network architecture |
| [Tools and Technologies](docs/tools-and-technologies.md) | Tools, technologies, and their roles |
| [Commands Explained](docs/commands-explained.md) | Linux, Git, and service management commands |
| [SSH Configuration](docs/ssh-configuration.md) | SSH keys, client configuration, and authentication |
| [Troubleshooting](docs/troubleshooting.md) | Problems encountered and their solutions |

## Future Improvements

- Configure HTTPS for secure web access
- Implement automated repository and database backups
- Explore multi-user collaboration and repository permissions
- Automate installation and configuration using Bash
- Add monitoring using Prometheus and Grafana
- Explore infrastructure automation and management

## Learning Outcomes

Through this project, I gained practical experience with:

- Git version control and remote repositories
- Linux server administration
- VMware virtualization and NAT networking
- SSH authentication and key management
- Gitea deployment and configuration
- SQLite database integration
- Linux systemd service management
- Networking and troubleshooting

## Project Status

The initial Gitea deployment is complete. SSH-based authentication and Git operations have been tested successfully, and the service has been verified to start automatically after an Ubuntu reboot.

This is a local lab deployment intended for learning and experimentation.

---

**Note:** Replace placeholders such as `<UBUNTU_IP>`, `<GITEA_USERNAME>`, and `<REPOSITORY_NAME>` with the values from your environment when following the documentation.
