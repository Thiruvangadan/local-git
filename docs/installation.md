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

## 4. Download and Install Gitea

Gitea is distributed as a precompiled binary. Instead of building the application from source code, we download the executable directly from the official Gitea website.

### 4.1 Download the Gitea Binary

```bash
wget -O gitea https://dl.gitea.com/gitea/1.23.4/gitea-1.23.4-linux-amd64
```

**Why do we use wget?**

`wget` is a command-line utility used to download files from the internet using URLs.

Command explanation:

- `wget`: Downloads a file from the specified URL.
- `-O`: Specifies the output filename.
- `gitea`: Name assigned to the downloaded file.
- URL: Official Gitea download location.

**Why download a binary?**

A binary is an executable file containing a compiled program. Gitea is written in Go and distributed as a precompiled binary, allowing us to run it without manually compiling its source code.

### 4.2 Make the Binary Executable

```bash
chmod +x gitea
```

**Why do we use chmod?**

Linux uses file permissions to control access to files and directories.

- `chmod`: Changes file permissions.
- `+x`: Adds executable permission.

This allows Linux to execute the downloaded Gitea binary.

### 4.3 Verify the Gitea Version

```bash
./gitea --version
```

**Why do we use this command?**

- `./`: Refers to a file in the current directory.
- `--version`: Displays the application version.

This helps verify that the downloaded binary is working and that the expected version was downloaded.

### 4.4 Move Gitea to the System Binary Directory

```bash
sudo mv gitea /usr/local/bin/gitea
```

**Why do we use /usr/local/bin?**

Linux organizes executable programs into different directories.

- `/usr/bin` generally contains executables managed by the system package manager.
- `/usr/local/bin` is commonly used for locally installed software.

Moving Gitea into `/usr/local/bin` allows us to execute it from different directories without specifying its full file path.

Command explanation:

- `sudo`: Executes the command with elevated privileges.
- `mv`: Moves or renames files.
- `/usr/local/bin/gitea`: Destination path of the executable.

### 4.5 Verify the Installation

```bash
gitea --version
```

**Why do we verify again?**

After moving the binary, this command checks whether Gitea can be executed from the system PATH.

**Expected Output:**

```text
Gitea version 1.23.4
```

This confirms that the Gitea executable is installed correctly and accessible through the system PATH.

### 4.6 Key Concepts

| Concept | Explanation |
|---|---|
| Binary | A compiled executable program |
| wget | Downloads files from URLs |
| chmod | Changes file permissions |
| +x | Adds executable permission |
| mv | Moves or renames files |
| sudo | Executes commands with elevated privileges |
| /usr/local/bin | Common location for locally installed executables |
| PATH | Environment variable containing directories searched for executable commands |

**Installation Result:** Gitea v1.23.4 was successfully downloaded, made executable, moved to `/usr/local/bin/`, and verified.

## 5. Create a Dedicated Gitea System User

For security and better permission management, Gitea runs under a dedicated Linux system user named `git` instead of running directly as the root user.

### 5.1 Create the Gitea User

```bash
sudo adduser --system --group --disabled-password --shell /bin/bash --home /home/git git
```

**Why do we create a separate user?**

Running applications as root provides them with extensive administrative privileges. This can introduce unnecessary security risks.

Creating a dedicated system user:

- Separates Gitea from other system processes.
- Limits the permissions available to the application.
- Provides a separate account for managing Gitea files and directories.
- Follows the principle of least privilege.

**Command explanation:**

| Option | Purpose |
|---|---|
| `sudo` | Executes the command with elevated privileges |
| `adduser` | Creates a Linux user |
| `--system` | Creates a system account |
| `--group` | Creates a group with the same name |
| `--disabled-password` | Disables password-based authentication for this account |
| `--shell /bin/bash` | Sets the user's login shell |
| `--home /home/git` | Specifies the user's home directory |
| `git` | Name of the user being created |

### 5.2 Verify the User

```bash
id git
```

**Why do we use this command?**

The `id` command displays information about a Linux user, including:

- UID (User ID): Unique numeric identifier assigned to the user.
- GID (Group ID): Numeric identifier assigned to the user's primary group.
- Groups: Groups to which the user belongs.

**Expected Output:**

```text
uid=... (git) gid=... (git) groups=...
```

The actual UID and GID values may differ depending on the system.

### 5.3 Understanding Linux Users and Groups

| Concept | Explanation |
|---|---|
| User | An account used to access the Linux system and run processes |
| Group | A collection of users used to manage permissions |
| UID | Unique numeric identifier of a user |
| GID | Numeric identifier of a group |
| Root | The superuser with extensive administrative privileges |
| System User | An account generally created to run system services and applications |

**Installation Result:** A dedicated Linux system user named `git` and its corresponding group were created successfully. This account will be used to run the Gitea application.

## 6. Create Gitea Directory Structure

Gitea requires separate directories to store application data, configuration files, customizations and logs.

### 6.1 Create Required Directories

```bash
sudo mkdir -p /var/lib/gitea/{custom,data,log}
sudo mkdir -p /etc/gitea
```

**Why do we create these directories?**

Linux follows a filesystem hierarchy where directories are organized according to their purpose.

| Directory | Purpose |
|---|---|
| `/var/lib/gitea/custom` | Stores Gitea customizations |
| `/var/lib/gitea/data` | Stores application data, including the SQLite database |
| `/var/lib/gitea/log` | Stores application log files |
| `/etc/gitea` | Stores Gitea configuration files |

**Command explanation:**

- `mkdir`: Creates directories.
- `-p`: Creates parent directories when necessary and avoids errors if they already exist.
- `{custom,data,log}`: Bash brace expansion creates multiple directories in one command.

**Why `/var/lib`?**

The `/var/lib` directory is conventionally used to store persistent application-specific data.

**Why `/etc`?**

The `/etc` directory is conventionally used to store system-wide application configuration files.

### 6.2 Assign Directory Ownership

```bash
sudo chown -R git:git /var/lib/gitea /etc/gitea
```

**Why do we use chown?**

Linux files and directories have owners and associated groups.

Since Gitea runs under the dedicated `git` user, it needs appropriate permissions to access its application directories.

Command explanation:

- `chown`: Changes file or directory ownership.
- `-R`: Applies the ownership change recursively.
- `git:git`: Sets the owner and group to `git`.

### 6.3 Configure Directory Permissions

```bash
sudo chmod 750 /etc/gitea
```

After creating the Gitea configuration file, set its permissions using:

```bash
sudo chmod 640 /etc/gitea/app.ini
```

**Why do we use chmod?**

`chmod` controls read, write and execute permissions for users and groups.

| Permission | Meaning |
|---|---|
| 7 | Read + Write + Execute |
| 6 | Read + Write |
| 5 | Read + Execute |
| 4 | Read only |
| 0 | No permissions |

For the `/etc/gitea` directory, `750` allows the owner full access, the group read and directory access, and denies access to others.

For `app.ini`, `640` allows the owner to read and write, the group to read, and denies access to others.

### 6.4 Verify the Directory Structure

```bash
ls -ld /var/lib/gitea /var/lib/gitea/custom /var/lib/gitea/data /var/lib/gitea/log /etc/gitea
```

**Why do we use this command?**

- `ls`: Lists files and directories.
- `-l`: Displays detailed information, including ownership and permissions.
- `-d`: Displays information about the directories themselves rather than listing their contents.

**Installation Result:** The required Gitea directories were created, and ownership was assigned to the dedicated `git` user and group.

## 7. Configure Gitea as a systemd Service

After installing the Gitea binary and preparing its directories, the next step is to configure Gitea as a systemd service.

### 7.1 What is systemd?

`systemd` is the service manager used by Ubuntu to manage system services and background processes.

We configure Gitea as a systemd service so that:

- Gitea can run in the background without keeping a terminal open.
- The application can start automatically when Ubuntu boots.
- We can start, stop and restart Gitea using systemctl commands.
- We can monitor the application's status and logs.

### 7.2 Create the Gitea Service File

Systemd service files are commonly stored in:

```text
/etc/systemd/system/
```

Create the Gitea service file:

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

Save the file.

**Why do we use this configuration?**

It tells systemd how to run the Gitea application, which Linux user should run it, where its working directory is located and what command should be executed.

### 7.3 Understanding the Service Configuration

| Setting | Purpose |
|---|---|
| `[Unit]` | Defines service information and dependencies |
| `Description` | Describes the service |
| `After` | Specifies startup ordering dependencies |
| `[Service]` | Defines how the application runs |
| `Type=simple` | Runs Gitea as a foreground process managed by systemd |
| `User=git` | Runs Gitea using the dedicated git user |
| `Group=git` | Runs Gitea using the git group |
| `WorkingDirectory` | Specifies the application's working directory |
| `ExecStart` | Specifies the command used to start Gitea |
| `Restart=always` | Configures automatic restarting if the process stops |
| `RestartSec=2s` | Waits two seconds before restarting |
| `Environment` | Defines environment variables for the service |
| `[Install]` | Defines how the service is enabled |
| `WantedBy=multi-user.target` | Allows the service to start during normal multi-user system startup |

### 7.4 Reload systemd Configuration

```bash
sudo systemctl daemon-reload
```

**Why do we use this command?**

After creating or modifying a service file, systemd needs to reload its configuration so it recognizes the changes.

### 7.5 Enable Gitea Service

```bash
sudo systemctl enable gitea
```

**Why?**

Enables automatic startup of Gitea when Ubuntu boots.

### 7.6 Start Gitea Service

```bash
sudo systemctl start gitea
```

**Why?**

Starts Gitea immediately in the background without requiring a system reboot.

### 7.7 Verify Service Status

```bash
sudo systemctl status gitea
```

The expected service state is:

```text
Active: active (running)
```

This indicates that systemd successfully started the Gitea process.

### 7.8 Useful systemctl Commands

| Command | Purpose |
|---|---|
| `systemctl start gitea` | Starts Gitea |
| `systemctl stop gitea` | Stops Gitea |
| `systemctl restart gitea` | Restarts Gitea |
| `systemctl enable gitea` | Enables automatic startup |
| `systemctl disable gitea` | Disables automatic startup |
| `systemctl status gitea` | Checks service status |
| `journalctl -u gitea` | Displays Gitea service logs |

**Installation Result:** Gitea was configured as a systemd-managed service, allowing it to run in the background and start automatically during Ubuntu startup.

## 8. Initial Gitea Installation and Configuration

After configuring and starting Gitea as a systemd service, the initial installation was completed through Gitea's web interface.

### 8.1 Access the Gitea Web Interface

First, obtain the IP address of the Ubuntu virtual machine:

```bash
hostname -I
```

From the Windows host machine, open a browser and access:

```text
http://<UBUNTU_IP>:3000
```

Replace `<UBUNTU_IP>` with the actual IP address of the Ubuntu virtual machine.

The Gitea initial installation page should appear.

### 8.2 Configure SQLite3

Gitea requires a database to store application information such as user accounts, repository metadata, settings and issues.

For this project, SQLite3 was selected through Gitea's browser-based installation page.

**Why SQLite3?**

- Lightweight and easy to configure.
- Does not require a separate database server.
- Stores information in a local database file.
- Suitable for a small personal Gitea deployment.

Other database options include PostgreSQL and MySQL/MariaDB, which can be considered for larger deployments.

The following settings were selected:

| Setting | Configuration |
|---|---|
| Database Type | SQLite3 |
| Database Path | `/var/lib/gitea/data/gitea.db` |

The database file is created and managed by Gitea during initialization.

### 8.3 Configure the Server

The following server settings were configured during the initial installation:

| Setting | Configuration |
|---|---|
| Server Domain | `<UBUNTU_IP>` |
| HTTP Listen Port | `3000` |
| Application Base URL | `http://<UBUNTU_IP>:3000/` |

The HTTP port is used to access the Gitea web interface from the browser.

### 8.4 Create the Administrator Account

An administrator account was created through the initial installation page.

This account provides access to Gitea administration, user management and repository management.

### 8.5 Understand the Gitea Configuration File

Gitea uses a configuration file named `app.ini` to store application settings.

**Configuration file location:**

```text
/etc/gitea/app.ini
```

This file contains database, server, web interface and SSH configuration settings.

Example database configuration:

```ini
[database]
DB_TYPE = sqlite3
PATH = /var/lib/gitea/data/gitea.db
```

Example server configuration:

```ini
[server]
DOMAIN = <UBUNTU_IP>
HTTP_PORT = 3000
ROOT_URL = http://<UBUNTU_IP>:3000/
```

Replace `<UBUNTU_IP>` with the actual IP address of the Ubuntu virtual machine.

| Setting | Purpose |
|---|---|
| `DB_TYPE` | Specifies the database backend used by Gitea. |
| `PATH` | Defines the location of the SQLite database file. |
| `DOMAIN` | Defines the hostname or IP address used to access Gitea. |
| `HTTP_PORT` | Specifies the port used by the web interface. |
| `ROOT_URL` | Defines the base URL used when generating application links. |

The browser-based installer generates the initial configuration. Additional settings can be adjusted in `app.ini` when required.

### 8.6 Set Configuration File Permissions

After the initial installation creates `app.ini`, configure its ownership and permissions:

```bash
sudo chown -R git:git /etc/gitea
sudo chmod 750 /etc/gitea
sudo chmod 640 /etc/gitea/app.ini
```

**Why are these commands used?**

- `chown`: Assigns ownership of the configuration directory and its contents to the Gitea user and group.
- `chmod 750`: Allows the owner full access, the group read and execute access, and denies access to other users.
- `chmod 640`: Allows the owner to read and modify the configuration file while the group can read it.

These permissions help protect Gitea's configuration from unauthorized access.

### 8.7 Verify the SQLite Database

Check whether the SQLite database file was created successfully:

```bash
sudo ls -lh /var/lib/gitea/data/gitea.db
```

The database file should be present at the specified location.

### 8.8 Important Gitea File Locations

| Component | Location |
|---|---|
| Gitea configuration | `/etc/gitea/app.ini` |
| SQLite database | `/var/lib/gitea/data/gitea.db` |
| Application data | `/var/lib/gitea/data/` |
| Customizations | `/var/lib/gitea/custom/` |
| Application logs | `/var/lib/gitea/log/` |

### Installation Result

Gitea was successfully initialized using SQLite3 through the browser-based installation page. The administrator account was created, and the application configuration and file permissions were verified.

The Gitea web interface was accessible from the Windows host machine through the Ubuntu virtual machine's IP address and port `3000`.
:
## 9. Configure Gitea Built-in SSH Server

Gitea supports SSH-based Git operations, allowing users to clone, push and pull repositories securely without relying on HTTP authentication for Git transport.

For this project, Gitea's built-in SSH server was configured to use port `2222`.

### 9.1 Why Use a Separate SSH Port?

Ubuntu already runs its OpenSSH service on port `22`, which is used for remote Linux administration.

To avoid a port conflict, Gitea's built-in SSH server was configured to use port `2222`.

| Service | Port | Purpose |
|---|---:|---|
| Ubuntu OpenSSH | `22` | Remote administration |
| Gitea SSH Server | `2222` | Git repository operations |

### 9.2 Update Gitea Configuration

The Gitea configuration file was opened:

```bash
sudo nano /etc/gitea/app.ini
```

The following settings were added under the `[server]` section:

```ini
[server]
DISABLE_SSH = false
START_SSH_SERVER = true
SSH_PORT = 2222
```
The existing server IP configuration was retained.

**Configuration explanation:**
- `DISABLE_SSH = false: Allows SSH functionality in Gitea.
- `START_SSH_SERVER`: Enables Gitea's built-in SSH server.
- `SSH_PORT`: Defines the SSH port displayed in repository clone instructions.

### 9.3 Restart the Gitea Service

After updating the configuration, the Gitea service was restarted:

```bash
sudo systemctl restart gitea
sudo systemctl status gitea
```

The service status was checked to confirm that Gitea restarted successfully.

### 9.4 Verify the SSH Port

The following command was used to check whether Gitea was listening on port `2222`:

```bash
sudo ss -tulpn | grep 2222
```

If UFW is enabled, the port must also be allowed through the firewall:

```bash
sudo ufw allow 2222/tcp
```

### Result

Gitea's built-in SSH server was configured to listen on port `2222`, while Ubuntu's OpenSSH service continued using port `22`.

This configuration allows Git clients to communicate with Gitea over SSH without interfering with remote Ubuntu administration.
