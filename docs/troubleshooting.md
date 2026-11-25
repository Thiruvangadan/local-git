# Troubleshooting

## SSH Port Conflict

Problem:
Ubuntu OpenSSH was already listening on port 22.

Solution:
Enabled Gitea's built-in SSH server and configured it to
use port 2222.

## Custom SSH Key Filename

Problem:
SSH was not automatically identifying the custom private key.

Solution:
Configured the Windows SSH config file with the
correct IdentityFile, HostName, User and Port.

## Git Authentication

Problem:
Git operations required authentication through Gitea.

Solution:
Generated an Ed25519 key pair and registered the
public key in Gitea's SSH/GPG Keys settings.

## Repository Connectivity

Problem:
The Windows client needed to communicate with the
Gitea SSH server inside the Ubuntu VM.

Solution:
Configured the correct VMware network address and
Gitea SSH port.
