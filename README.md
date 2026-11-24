# Self-Hosted Git Platform using Gitea

A lightweight, self-hosted Git hosting platform deployed on an Ubuntu virtual machine using VMware Workstation. The platform uses Gitea to manage Git repositories, users, and collaboration through a web interface, with SSH-based authentication for Git operations.

## Project Overview

The objective of this project was to understand how Git hosting platforms work by deploying and managing a self-hosted alternative to GitHub.

Gitea provides the repository management interface, while Git and SSH handle version control and secure communication between client machines and the server.

The server runs inside an Ubuntu virtual machine, making it possible to host and access repositories from multiple machines on a local network.

## Architecture

**Server:**
- Ubuntu Linux running inside VMware Workstation
- Gitea as the Git hosting platform
- SQLite3 as the database
- Gitea's built-in SSH server for Git operations
- OpenSSH Server for remote Ubuntu administration

**Client:**
- Windows machine with Git installed
- SSH key-based authentication using Ed25519
- Windows SSH configuration for custom key filenames and connection settings

### Network Architecture

| Component | Port | Purpose |
|---|---:|---|
| Ubuntu OpenSSH | 22 | Remote server administration |
| Gitea Web Interface | 3000 | Repository and user management |
| Gitea Built-in SSH | 2222 | Git clone, push, and pull |

## Technology Stack

| Component | Technology | Purpose |
|---|---|---|
| Virtualization | VMware Workstation | Runs the Ubuntu server |
| Operating System | Ubuntu Linux x86_64 | Server environment |
| Git Hosting | Gitea v1.23.4 | Self-hosted Git platform |
| Version Control | Git | Source code management |
| Database | SQLite3 | Stores Gitea application data |
| Authentication | SSH / Ed25519 | Key-based Git authentication |
| Remote Administration | OpenSSH | Secure Ubuntu terminal access |

## Features

- Self-hosted Git repository management
- Web-based repository interface
- SSH-based Git authentication
- Repository cloning, pushing, and pulling
- User and repository management
- Private repository support
- Multiple client machine support
- Lightweight SQLite database integration

## Architecture and Setup

### 1. VMware Network Configuration

Create an Ubuntu virtual machine using a 64-bit x86_64 Ubuntu ISO.

Configure the VMware network adapter:

- NAT: Allows communication between the Windows host and the Ubuntu virtual machine.
- Bridged: Allows the VM to participate directly in the local physical network.

For this setup, NAT networking is used.

Verify the architecture and network address:

```bash
uname -m
ip -4 addr show
```

Expected architecture:

```text
x86_64
```

Note the Ubuntu VM's IPv4 address. This is referred to as `<UBUNTU_IP>` throughout this documentation.

### 2. Install Required Packages

Update the package list and install Git, OpenSSH Server, and wget:

```bash
sudo apt update
sudo apt install -y openssh-server git wget
```

Enable and start OpenSSH:

```bash
sudo systemctl enable --now ssh
```

Verify the service:

```bash
sudo systemctl status ssh
```

If UFW is active, allow the required ports:

```bash
sudo ufw allow 22/tcp
sudo ufw allow 3000/tcp
sudo ufw allow 2222/tcp
```

### 3. Install Gitea

Download the Gitea Linux AMD64 binary:

```bash
wget -O gitea https://dl.gitea.com/gitea/1.23.4/gitea-1.23.4-linux-amd64
```

Make the binary executable:

```bash
chmod +x gitea
```

Move it to the system executable directory:

```bash
sudo mv gitea /usr/local/bin/gitea
```

### 4. Create a Dedicated Git User

Create a dedicated system account for running Gitea:

```bash
sudo adduser --system --group --disabled-password --shell /bin/bash --home /home/git git
```

Create the required directories:

```bash
sudo mkdir -p /var/lib/gitea/{custom,data,log}
sudo mkdir -p /etc/gitea
```

Set ownership:

```bash
sudo chown -R git:git /var/lib/gitea /etc/gitea
```

### 5. Configure Gitea as a Systemd Service

Create the service file:

```bash
sudo nano /etc/systemd/system/gitea.service
```

Add the following configuration:

```ini
[Unit]
Description=Gitea (Git with a cup of tea)
After=syslog.target
After=network.target

[Service]
RestartSec=2s
Type=simple
User=git
Group=git
WorkingDirectory=/var/lib/gitea/
ExecStart=/usr/local/bin/gitea web --config /etc/gitea/app.ini
Restart=always
Environment=USER=git HOME=/home/git GITEA_WORK_DIR=/var/lib/gitea

[Install]
WantedBy=multi-user.target
```

Reload systemd and enable Gitea:

```bash
sudo systemctl daemon-reload
sudo systemctl enable --now gitea
```

Check the service status:

```bash
sudo systemctl status gitea
```

### 6. Initial Web Configuration

Open the following address in a browser on the Windows host:

```text
http://<UBUNTU_IP>:3000
```

Complete the initial Gitea configuration.

Database:
- Database Type: SQLite3

Server:
- Server Domain: `<UBUNTU_IP>`
- HTTP Port: `3000`
- Gitea Base URL: `http://<UBUNTU_IP>:3000/`

Administrator:
- Create an administrator account.
- Configure a username and password.

Complete the installation through the Gitea web interface.

After installation, restrict configuration file permissions:

```bash
sudo chmod 750 /etc/gitea
sudo chmod 640 /etc/gitea/app.ini
```

### 7. Configure Gitea Built-in SSH Server

Ubuntu OpenSSH already uses port 22. To avoid a port conflict, Gitea's built-in SSH server is configured to use port 2222.

Edit the Gitea configuration:

```bash
sudo nano /etc/gitea/app.ini
```

Configure the server section:

```ini
[server]
START_SSH_SERVER = true
SSH_DOMAIN = <UBUNTU_IP>
SSH_PORT = 2222
SSH_LISTEN_PORT = 2222
```

Restart Gitea:

```bash
sudo systemctl restart gitea
```

Verify that Gitea is listening on port 2222:

```bash
sudo ss -tlnp | grep 2222
```

The two SSH services now have separate responsibilities:

- Port 22: Ubuntu remote administration.
- Port 2222: Gitea Git operations.

### 8. Configure SSH Authentication on Windows

Generate an Ed25519 SSH key pair on the Windows client:

```powershell
ssh-keygen -t ed25519 -C "minigit-windows"
```

A custom filename can be provided when prompted.

For example:

```text
C:\Users\<WINDOWS_USER>\.ssh\minigit_key
C:\Users\<WINDOWS_USER>\.ssh\minigit_key.pub
```

The private key stays on the client machine. Only the public key is registered in Gitea.

Copy the public key:

```powershell
Get-Content "$env:USERPROFILE\.ssh\minigit_key.pub"
```

In the Gitea web interface:

1. Open User Settings.
2. Select SSH / GPG Keys.
3. Choose Add Key.
4. Provide a title.
5. Paste the public key.
6. Save the key.

### 9. Configure the Windows SSH Client

Since a custom SSH key filename is used and Gitea listens on port 2222, configure an SSH alias.

Open the Windows SSH configuration file:

```powershell
notepad $env:USERPROFILE\.ssh\config
```

Add:

```sshconfig
Host minigit
    HostName <UBUNTU_IP>
    User git
    Port 2222
    IdentityFile ~/.ssh/minigit_key
    IdentitiesOnly yes
```

Replace `<UBUNTU_IP>` and the key filename with the actual values used in your environment.

This configuration allows SSH to automatically select the correct host, port, username, and private key.

Test the connection:

```powershell
ssh -T minigit
```

A successful authentication response should identify the Gitea account associated with the registered public key.

### 10. Repository Operations

Create a repository through the Gitea web interface.

Clone the repository:

```bash
git clone git@minigit:<GITEA_USERNAME>/<REPOSITORY_NAME>.git
```

Enter the repository:

```bash
cd <REPOSITORY_NAME>
```

Create a file or make a change:

```bash
echo "Hello from my self-hosted Git platform" > test.txt
```

Stage the changes:

```bash
git add .
```

Commit:

```bash
git commit -m "Add initial test file"
```

Push to Gitea:

```bash
git push origin main
```

Verify the commit and updated files through the Gitea web interface.

Other supported operations include:

```bash
git pull
git fetch
git branch
git log --oneline
```

## Challenges and Solutions

### SSH Port Conflict

**Problem:** Ubuntu OpenSSH was already using port 22, which prevented Gitea's built-in SSH server from using the same port.

**Solution:** Enabled Gitea's built-in SSH server using `START_SSH_SERVER = true` and configured it to listen on port 2222.

### Custom SSH Key Filename

**Problem:** The SSH key was generated using a custom filename rather than the default `id_ed25519`, so SSH did not automatically select it.

**Solution:** Created a Windows SSH configuration file specifying the server IP, username, port, and private key location.

### Client-to-Server Git Connectivity

**Problem:** Git operations needed to work between Windows and the Ubuntu virtual machine.

**Solution:** Configured Gitea SSH authentication and verified repository cloning and pushing through the custom SSH connection.

## Future Improvements

- Multi-user repository collaboration
- Repository access permissions
- Automated repository backups
- HTTPS configuration
- Monitoring using Prometheus and Grafana
- Automated installation and configuration using Bash
- Improved deployment and infrastructure management

## Learning Outcomes

This project provided practical experience with:

- Git version control and remote repositories
- Linux server administration
- VMware virtualization and NAT networking
- SSH authentication and key management
- Gitea deployment and configuration
- SQLite database integration
- Linux systemd service management
- Troubleshooting networking and SSH configuration

## Project Status

Initial Gitea deployment and SSH-based Git operations are working. Further improvements and multi-user functionality are planned.

---

**Note:** This is a local lab deployment. The IP address, SSH configuration, and network settings depend on the local environment. Replace all placeholders with the corresponding values from your setup.
