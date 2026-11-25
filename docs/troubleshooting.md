# Troubleshooting

This document describes the issues encountered while setting up the self-hosted Gitea platform and the solutions used to resolve them.

## 1. SSH Port Configuration

**Problem:**

Ubuntu's OpenSSH server was already using port `22`. Gitea also needed an SSH port for Git operations.

**Solution:**

Instead of changing Ubuntu's existing OpenSSH configuration, Gitea's built-in SSH server was enabled and configured to use port `2222`.

The following settings were configured in `/etc/gitea/app.ini`:

```ini
[server]
DISABLE_SSH = false
START_SSH_SERVER = true
SSH_PORT = 2222
```

This allowed both services to operate independently:

- Port `22` – Ubuntu OpenSSH server for remote administration.
- Port `2222` – Gitea built-in SSH server for Git operations.

**Verification:**

```bash
sudo ss -tulpn | grep 2222
```

This checks whether Gitea is listening on port `2222`.

## 2. Custom SSH Key Filename

**Problem:**

The Windows machine already had a default SSH key named `id_ed25519`, which was being used for GitHub. A separate key was needed for Gitea.

SSH also needed to know which private key to use when connecting to the Gitea server.

**Solution:**

Generated a separate Ed25519 key pair for Gitea and used the filename `id_ed25519_gitea`.

Configured the Windows SSH config file (`~/.ssh/config`) with the appropriate connection details:

```sshconfig
Host gitea
    HostName <UBUNTU_IP>
    User git
    Port 2222
    IdentityFile ~/.ssh/id_ed25519_gitea
    IdentitiesOnly yes
```

The `IdentityFile` setting specifies the private key to use, while `IdentitiesOnly yes` ensures SSH uses the explicitly configured identity for this host.

**Verification:**

```powershell
ssh -T gitea
```

For additional debugging information:

```powershell
ssh -vT gitea
```

## 3. Git Authentication with Gitea

**Problem:**

Git operations over SSH required the Windows client to authenticate with the Gitea server.

**Solution:**

Generated an Ed25519 SSH key pair and added the public key to the Gitea account under **Settings → SSH/GPG Keys**.

The private key remained on the Windows machine and was referenced through the SSH config file.

**Verification:**

```powershell
ssh -T gitea
```

A successful authentication message confirms that Gitea accepted the registered SSH key.

**Note:** Gitea does not provide interactive shell access. Therefore, a message stating that authentication succeeded but shell access is unavailable is expected behavior, not an error.

## 4. Repository Connectivity

**Problem:**

The Windows client needed to communicate with the Gitea SSH server running inside the Ubuntu VMware virtual machine.

**Solution:**

Used VMware NAT networking and the Ubuntu VM's network address to establish connectivity between the Windows host and the virtual machine.

Configured the Gitea SSH connection to use port `2222`.

Repositories could then be cloned using the configured SSH alias:

```bash
git clone gitea:<GITEA_USERNAME>/<REPOSITORY_NAME>.git
```

Alternatively, the complete SSH URL could be used:

```bash
git clone ssh://git@<UBUNTU_IP>:2222/<GITEA_USERNAME>/<REPOSITORY_NAME>.git
```

**Note:** These are two alternative clone formats. The SSH alias should not be combined with the complete SSH URL format.

## 5. Permission Denied While Accessing Gitea Configuration

**Problem:**

Attempting to access `/etc/gitea` using a regular Ubuntu user resulted in a permission-denied error.

```bash
cd /etc/gitea
```

**Solution:**

The Gitea configuration directory has restricted permissions to protect its configuration files.

Use administrative privileges when inspecting or editing protected configuration files.

For example:

```bash
sudo ls -l /etc/gitea
sudo nano /etc/gitea/app.ini
```

Avoid changing the directory permissions to make it accessible to every user.

## 6. Gitea Service Verification

**Problem:**

After configuring Gitea as a systemd service, it was necessary to verify that the service started correctly and would remain available after restarting Ubuntu.

**Solution:**

Checked the service status:

```bash
sudo systemctl status gitea
```

Enabled automatic startup:

```bash
sudo systemctl enable gitea
```

After rebooting Ubuntu, verified that Gitea started automatically and that both the web interface and Git operations were accessible.

## 7. Useful Diagnostic Commands

The following commands can help investigate connectivity and service-related issues in future deployments.

| Purpose | Command |
|---|---|
| Check Ubuntu IP address | `hostname -I` |
| Check Gitea service | `sudo systemctl status gitea` |
| Check OpenSSH service | `sudo systemctl status ssh` |
| Check Gitea web port | `sudo ss -tulpn \| grep 3000` |
| Check Gitea SSH port | `sudo ss -tulpn \| grep 2222` |
| Test SSH authentication | `ssh -T gitea` |
| Debug SSH authentication | `ssh -vT gitea` |
| Check Git repository status | `git status` |

