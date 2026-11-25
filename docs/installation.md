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
