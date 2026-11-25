# Commands Explained

This document explains the important Linux, system administration, SSH, and Git commands used while setting up and managing the self-hosted Gitea platform.

The commands are grouped by purpose rather than installation order, making this document useful as a quick reference when maintaining or troubleshooting the platform.

## 1. System Information

Commands used to inspect the Ubuntu environment and installed software versions.

### Check Ubuntu Version

```bash
lsb_release -a
```

Displays the Ubuntu distribution, release number, and codename.

### Check Git Version

```bash
git --version
```

Displays the installed Git version.

### Check SQLite Version

```bash
sqlite3 --version
```

Displays the installed SQLite version.

### Check OpenSSH Version

```bash
ssh -V
```

Displays the installed OpenSSH client version.

### Check Gitea Version

```bash
gitea --version
```

Displays the installed Gitea version and build information.

### Check Ubuntu IP Address

```bash
hostname -I
```

Displays the IP addresses assigned to the Ubuntu machine. The VM's IP address is needed when accessing Gitea from the Windows host.

## 2. Package Management

Commands used to update Ubuntu and install the required packages.

### Update Package Lists

```bash
sudo apt update
```

Downloads the latest package information from the configured repositories. It does not install or upgrade packages.

### Upgrade Installed Packages

```bash
sudo apt upgrade -y
```

Upgrades installed packages to their available newer versions.

The `-y` option automatically confirms the installation prompts.

### Install Required Packages

```bash
sudo apt install wget git curl openssh-server sqlite3 -y
```

Installs the packages required for downloading Gitea, performing Git operations, establishing SSH connections, and using SQLite.

- `wget` – Downloads files from the internet.
- `git` – Provides version control functionality.
- `curl` – Transfers data to or from servers.
- `openssh-server` – Allows SSH connections to the Ubuntu machine.
- `sqlite3` – Provides the SQLite command-line utility.

## 3. Gitea Installation and User Management

Commands used to download Gitea, install its executable, and create a dedicated system user.

### Download Gitea

```bash
wget -O gitea https://dl.gitea.com/gitea/1.23.4/gitea-1.23.4-linux-amd64
```

Downloads the Gitea 1.23.4 binary for 64-bit Linux.

The `-O` option specifies the output filename as `gitea`.

### Make the Binary Executable

```bash
chmod +x gitea
```

Adds executable permission to the downloaded Gitea binary.

### Check the Downloaded Binary

```bash
./gitea --version
```

Runs the Gitea executable from the current directory and displays its version.

The `./` prefix specifies that the executable is in the current directory.

### Move Gitea to the System Binary Directory

```bash
sudo mv gitea /usr/local/bin/gitea
```

Moves the Gitea executable to `/usr/local/bin`, allowing it to be executed from different directories.

### Create a Dedicated Gitea System User

```bash
sudo adduser --system --group --disabled-password --shell /bin/bash --home /home/git git
```

Creates a dedicated system user named `git` to run the Gitea service.

- `--system` – Creates a system account.
- `--group` – Creates a group with the same name.
- `--disabled-password` – Prevents password-based login for the account.
- `--shell /bin/bash` – Sets the user's login shell.
- `--home /home/git` – Specifies the home directory.
- `git` – The name of the account.

### Verify the Gitea User

```bash
id git
```

Displays the user ID, group ID, and group memberships of the `git` account.

## 4. Directory and Permission Management

Commands used to create Gitea directories and configure their ownership and permissions.

### Create Gitea Directories

```bash
sudo mkdir -p /var/lib/gitea/{custom,data,log}
```

Creates the required Gitea directories.

The `-p` option creates parent directories when necessary and does not report an error if the directories already exist.

The brace expansion creates the three directories:

- `/var/lib/gitea/custom`
- `/var/lib/gitea/data`
- `/var/lib/gitea/log`

Create the configuration directory separately:

```bash
sudo mkdir -p /etc/gitea
```

### Change Directory Ownership

```bash
sudo chown -R git:git /var/lib/gitea /etc/gitea
```

Changes the ownership of the specified directories and their contents to the `git` user and group.

- `chown` – Changes ownership.
- `-R` – Applies the change recursively.
- `git:git` – Specifies the owner and group.

### Set Directory Permissions

```bash
sudo chmod 750 /etc/gitea
```

Sets the permissions of the Gitea configuration directory.

Permission `750` means:

- Owner: Read, write, and execute.
- Group: Read and execute.
- Others: No permissions.

### Set Configuration File Permissions

```bash
sudo chmod 640 /etc/gitea/app.ini
```

Sets the permissions of the Gitea configuration file.

Permission `640` means:

- Owner: Read and write.
- Group: Read only.
- Others: No permissions.

These permissions help restrict access to Gitea's configuration, which may contain sensitive settings.

## 5. Service Management with systemd

Commands used to register, start, stop, and inspect the Gitea and OpenSSH services.

### Reload systemd Configuration

```bash
sudo systemctl daemon-reload
```

Instructs systemd to reload service unit files after creating or modifying a service configuration.

### Enable Gitea at Startup

```bash
sudo systemctl enable gitea
```

Configures Gitea to start automatically when Ubuntu boots.

### Start Gitea

```bash
sudo systemctl start gitea
```

Starts the Gitea service immediately.

### Check Gitea Service Status

```bash
sudo systemctl status gitea
```

Displays the current status of the Gitea service, including whether it is active, its process information, and recent log messages.

### Restart Gitea

```bash
sudo systemctl restart gitea
```

Stops and starts the Gitea service again. Useful after changing configuration settings.

### Check OpenSSH Service

```bash
sudo systemctl status ssh
```

Displays the status of the Ubuntu OpenSSH server service.

### Enable OpenSSH at Startup

```bash
sudo systemctl enable ssh
```

Configures the OpenSSH server to start automatically when Ubuntu boots.

### Start OpenSSH

```bash
sudo systemctl start ssh
```

Starts the OpenSSH server immediately.

## 6. Network and Port Verification

Commands used to inspect network connectivity and verify that Gitea is listening on its configured ports.

### Check Listening Ports

```bash
sudo ss -tulpn
```

Displays listening TCP and UDP sockets along with process information.

Options:

- `-t` – Display TCP sockets.
- `-u` – Display UDP sockets.
- `-l` – Display listening sockets.
- `-p` – Display associated processes.
- `-n` – Display numeric addresses and port numbers.

### Check the Gitea SSH Port

```bash
sudo ss -tulpn | grep 2222
```

Filters the socket information to display entries associated with port `2222`.

This was used to verify that Gitea's built-in SSH server was listening on the configured port.

### Check the Gitea Web Port

```bash
sudo ss -tulpn | grep 3000
```

Checks whether a service is listening on port `3000`, which is used by the Gitea web interface.

### Check SQLite Database File

```bash
sudo ls -lh /var/lib/gitea/data/gitea.db
```

Displays the SQLite database file's permissions, ownership, size, and modification information.

## 7. SSH Key Management and Authentication

Commands used to generate an SSH key pair, inspect the public key, and test authentication with Gitea.

### Generate an Ed25519 SSH Key

```powershell
ssh-keygen -t ed25519 -C "gitea"
```

Generates a new Ed25519 SSH key pair.

- `-t ed25519` – Specifies the Ed25519 key algorithm.
- `-C "gitea"` – Adds a comment to identify the key.

The private and public keys are saved in the location specified during key generation.

### Display the Public Key

```powershell
Get-Content "$env:USERPROFILE\.ssh\id_ed25519_gitea.pub"
```

Displays the contents of the public key file in Windows PowerShell.

The public key is copied into the SSH keys section of the Gitea user account.

**Important:** Only the public key (`.pub`) should be added to Gitea. Never share the private key.

### Test SSH Authentication

```powershell
ssh -T gitea
```

Tests SSH authentication using the `gitea` host alias configured in the Windows SSH config file.

A successful authentication message confirms that the SSH key was accepted by Gitea.

### Enable Verbose SSH Output

```powershell
ssh -vT gitea
```

Runs the SSH authentication test with verbose output.

The `-v` option provides additional debugging information, including which SSH keys are offered and whether authentication succeeds.

### Connect Using a Specific SSH Key

```powershell
ssh -i "$env:USERPROFILE\.ssh\id_ed25519_gitea" -p 2222 -T git@<UBUNTU_IP>
```

Tests SSH authentication by explicitly specifying the private key, server IP address, and port.

- `-i` – Specifies the private key file.
- `-p 2222` – Specifies the SSH port.
- `-T` – Disables pseudo-terminal allocation.
- `git@<UBUNTU_IP>` – Connects as the Gitea system user.

## 8. Git Operations

Commands used to interact with repositories hosted on the Gitea server.

### Clone a Repository

```bash
git clone gitea:<GITEA_USERNAME>/<REPOSITORY_NAME>.git
```

Downloads a remote repository to the local machine using the `gitea` SSH host alias.

The alias uses the SSH configuration defined on the Windows client.

An alternative is to use the complete SSH URL:

```bash
git clone ssh://git@<UBUNTU_IP>:2222/<GITEA_USERNAME>/<REPOSITORY_NAME>.git
```

Both commands connect to the same Gitea SSH service. Use either the configured alias or the complete SSH URL.

### Check Repository Status

```bash
git status
```

Displays the current branch and the status of modified, staged, and untracked files.

### Stage a File

```bash
git add README.md
```

Stages the specified file so it can be included in the next commit.

To stage all changes in the current directory:

```bash
git add .
```

### Create a Commit

```bash
git commit -m "Add initial README"
```

Records the staged changes in the local repository.

The `-m` option specifies the commit message.

### Push Changes to Gitea

```bash
git push origin main
```

Uploads the local `main` branch commits to the remote repository named `origin`.

### Pull Changes from Gitea

```bash
git pull
```

Fetches and integrates changes from the configured upstream branch into the current local branch.

### Check the Current Branch

```bash
git branch --show-current
```

Displays the name of the currently checked-out branch.

This is useful when checking whether the local branch is named `main` or `master`.

---

## 9. Important Notes

- Commands beginning with `sudo` require administrative privileges.
- Linux commands are intended to be executed inside the Ubuntu VM unless specified otherwise.
- Commands labelled PowerShell are intended to be executed on the Windows host.
- Replace placeholders such as `<UBUNTU_IP>`, `<GITEA_USERNAME>`, and `<REPOSITORY_NAME>` with the actual values from your environment.
- The Gitea web interface uses port `3000`, while its built-in SSH server uses port `2222`.
- Ubuntu's OpenSSH server uses port `22` for SSH administration. It is separate from Gitea's built-in SSH server.
- The commands in this document are references to the commands used during the project. They are not intended to be executed all at once.

