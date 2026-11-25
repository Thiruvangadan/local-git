# SSH Configuration and Authentication

## Overview

SSH (Secure Shell) is used in this project for secure communication between the Windows host machine and the Ubuntu virtual machine.

Two separate SSH services are configured:

1. Ubuntu OpenSSH Server — used for remote administration of the Ubuntu virtual machine.
2. Gitea Built-in SSH Server — used for Git repository operations such as cloning, pushing, and pulling.

Both services operate independently on different TCP ports.

## SSH Services Architecture

| SSH Service | Port | Purpose |
|---|---:|---|
| Ubuntu OpenSSH | 22 | Remote Linux administration |
| Gitea Built-in SSH | 2222 | Git repository authentication and operations |

Ubuntu OpenSSH handles connections to Linux user accounts, whereas Gitea's built-in SSH server handles Git operations using keys registered within Gitea.

The two services are configured separately to avoid port conflicts and maintain independent responsibilities.

## 1. Ubuntu OpenSSH Server

Ubuntu's OpenSSH server provides remote access to the virtual machine.

It runs on TCP port `22` and allows authorized users to connect to the Ubuntu operating system.

**Primary use cases:**

- Remote Linux administration.
- Executing commands on the Ubuntu server.
- Managing server files and services.
- Monitoring Gitea and other system processes.

The SSH service can be managed using systemd:

```bash
# Start SSH service
sudo systemctl start ssh

# Enable SSH during system boot
sudo systemctl enable ssh

# Check SSH service status
sudo systemctl status ssh
```

This service is independent of Gitea's built-in SSH server.

## 2. Gitea Built-in SSH Server

Gitea provides its own built-in SSH server, which handles Git operations over SSH.

In this project, the built-in SSH server listens on TCP port `2222`.

The following settings are configured under the `[server]` section of `/etc/gitea/app.ini`:

```ini
[server]
DISABLE_SSH = false
START_SSH_SERVER = true
SSH_PORT = 2222
```

**Configuration explanation:**

- `DISABLE_SSH = false` — enables SSH functionality in Gitea.
- `START_SSH_SERVER = true` — starts Gitea's built-in SSH server.
- `SSH_PORT = 2222` — specifies the SSH port advertised by Gitea for Git operations.

The Gitea service manages this built-in SSH server.

The listening port can be checked on Ubuntu using:

```bash
sudo ss -tulpn | grep 2222
```

The output should show the Gitea process listening on port `2222`.

Unlike Ubuntu OpenSSH, Gitea's built-in SSH server does not provide interactive shell access. It is intended for Git operations.

## 3. SSH Key Authentication

SSH public-key authentication is used to authenticate the Windows Git client with Gitea.

An Ed25519 key pair was generated specifically for Gitea because an existing SSH key was already being used for GitHub.

The key pair consists of:

| File | Purpose |
|---|---|
| `id_ed25519_gitea` | Private key stored on Windows |
| `id_ed25519_gitea.pub` | Public key registered in Gitea |

The private key must remain securely stored on the Windows machine. Only the public key is uploaded to Gitea.

### Authentication Workflow

1. The Windows client initiates an SSH connection to the Gitea server.
2. The SSH client uses the configured private key to authenticate.
3. Gitea checks the corresponding public key registered in the user's account.
4. If authentication succeeds, Gitea allows the requested Git operation.

The public key is registered through:

**Gitea → User Settings → SSH / GPG Keys → Add Key**

This registration is separate from Ubuntu's `~/.ssh/authorized_keys`, which is used for authenticating Linux account access through Ubuntu OpenSSH.

## 4. Windows SSH Client Configuration

To simplify SSH connections, a host alias named `gitea` is configured in the Windows SSH client configuration file.

Configuration file location:

```text
C:\Users\<YOUR_USERNAME>\.ssh\config
```

Configuration:

```sshconfig
Host gitea
    HostName <UBUNTU_IP>
    User git
    Port 2222
    IdentityFile ~/.ssh/id_ed25519_gitea
    IdentitiesOnly yes
```

**Configuration explanation:**

- `Host` — defines a shortcut name for the SSH connection.
- `HostName` — specifies the Ubuntu virtual machine's IP address.
- `User` — specifies the Gitea system user.
- `Port` — specifies Gitea's built-in SSH port.
- `IdentityFile` — specifies the private key used for authentication.
- `IdentitiesOnly yes` — limits SSH authentication to the explicitly configured identity.

The host alias allows the SSH client to reuse these connection details without requiring them to be entered every time.

### Alternative Without an SSH Config File

The SSH configuration file is optional. The connection details can also be supplied directly:

```powershell
ssh -i "$env:USERPROFILE\.ssh\id_ed25519_gitea" -p 2222 -T git@<UBUNTU_IP>
```

This provides an alternative when a host alias has not been configured.

## 5. Testing SSH Authentication

After registering the public key and configuring the SSH client, authentication can be tested using:

```bash
ssh -T gitea
```

A successful connection returns a message similar to:

```text
Hi there, <USERNAME>! You've successfully authenticated with the key named <KEY_NAME>, but Gitea does not provide shell access.
```

This confirms that the SSH connection was established and the registered key was accepted by Gitea.

To inspect which key is being offered and accepted, use verbose mode:

```bash
ssh -vT gitea
```

Look for messages indicating that the expected private key was offered and accepted.

## 6. Git Operations Using SSH

Once SSH authentication is configured, Git repositories can be accessed using the Gitea SSH host alias.

### Clone a Repository

```bash
git clone gitea:<GITEA_USERNAME>/<REPOSITORY_NAME>.git
```

### Push Changes

```bash
git add .
git commit -m "Your commit message"
git push origin main
```

### Pull Changes

```bash
git pull
```

The SSH client automatically uses the connection details specified in the `gitea` host configuration.

### Alternative: Full SSH URL

Git operations can also use the complete SSH URL:

```bash
git clone ssh://git@<UBUNTU_IP>:2222/<GITEA_USERNAME>/<REPOSITORY_NAME>.git
```

This method does not require the `gitea` host alias because the server address, username, and port are specified directly.

## 7. SSH Configuration Summary

The project uses two independent SSH services for different purposes.

| Feature | Ubuntu OpenSSH | Gitea Built-in SSH |
|---|---|---|
| Port | 22 | 2222 |
| Service purpose | Linux administration | Git operations |
| Authentication target | Ubuntu user account | Gitea account |
| Public key location | Ubuntu `authorized_keys`, when using key-based login | Gitea account SSH keys |
| Interactive shell | Yes, for authorized accounts | No |

This separation allows remote Ubuntu administration and secure Git repository access to operate independently within the same virtual machine.

## Conclusion

SSH provides the secure communication mechanism used by this self-hosted Git platform.

Ubuntu OpenSSH handles remote server administration, while Gitea's built-in SSH server handles Git authentication and repository operations.

Using a dedicated SSH key pair and a configured host alias simplifies repository access from Windows while keeping the two SSH services separate.
