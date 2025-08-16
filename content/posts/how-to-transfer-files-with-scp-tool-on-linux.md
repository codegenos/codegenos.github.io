---
author: "CodeGenos"
title: "How to Transfer Files with scp on Linux"
date: 2023-09-08T21:28:35+03:00
description: "Learn how to use the scp command to securely copy files and folders on Linux, with Ubuntu install steps, ports, SSH keys, and practical examples."
tags: ["Linux", "ssh", "scp", "Ubuntu", "OpenSSH"]
categories: ["Linux", "ssh"]
ShowToc: true
ShowBreadCrumbs: true
draft: false
---

Need to copy files to a headless Raspberry Pi over SSH? The `scp` command lets you securely transfer files and folders between machines on Linux. This guide covers local↔remote and remote↔remote copies, plus useful flags.

## What is the scp command?
`scp` (secure copy) securely transfers files over an `SSH` connection. It uses the same authentication and encryption as SSH.

In plain terms, scp is like copying a file from one folder to another, except the other “folder” is a computer you can reach over SSH. It’s simple, fast to use for everyday tasks, and available on most Linux and macOS systems by default.

- What you might use it for: sending project files to a server, backing up a directory, fetching logs, or moving a build artifact to a remote host.
- Why people choose it: straightforward commands, secure by default, no extra setup when SSH already works.

Note: In modern OpenSSH releases, `scp` uses the SFTP protocol under the hood by default for safer behavior. You can force legacy scp/rcp behavior with `-O` if needed.

## Cheat sheet
- Local → Remote: `scp ./file.zip user@host:/path/`
- Remote → Local: `scp user@host:/path/file.zip ./`
- Remote → Remote: `scp user1@host1:/path/file.zip user2@host2:/dest/`
- Copy directory: `scp -r ./dir user@host:/dest/`
- Custom port: `scp -P 2222 file.zip user@host:/path/`
- SSH key: `scp -i ~/.ssh/id_ed25519 file.zip user@host:/path/`
- Compression: `scp -C file.zip user@host:/path/`
- Preserve times/modes: `scp -p file.zip user@host:/path/`
- Limit bandwidth (Kb/s): `scp -l 5000 file.zip user@host:/path/`

## Copy files with scp
SCP lets you securely transfer files between a local machine and a remote server, or even between two remote servers.

### How do I copy a file from local to remote?
```bash
scp /path/to/local/file.zip username@remote_server:/path/to/destination/folder
```
- `/path/to/local/file.zip`: local file to copy
- `username`: your username on the remote server
- `remote_server`: hostname or IP address
- `/path/to/destination/folder`: destination path on the remote server

You will be prompted for the remote user password unless using SSH keys.

### How do I copy a file from remote to local?
```bash
scp username@remote_server:/path/to/remote/file.zip /path/to/local/folder
```

### How do I copy from one remote server to another?
```bash
scp username1@remote_server1:/path/to/remote/file.zip username2@remote_server2:/path/to/destination/folder
```

## Copy folders with scp
You can copy an entire directory recursively.

### Copy a folder from local to remote
```bash
scp -r /path/to/local/folder username@remote_server:/path/to/destination/folder
```
- `-r`: recursively copy the folder and its contents

### Copy a folder from remote to local
```bash
scp -r username@remote_server:/path/to/remote/folder /path/to/local/folder
```

### Copy a folder between two remote servers
```bash
scp -r username1@remote_server1:/path/to/remote/folder username2@remote_server2:/path/to/remote/folder
```

## Useful options and tips
- Use a custom port (uppercase P):
  ```bash
  scp -P 2222 file.zip user@host:/path/
  ```
- Use an SSH key (identity file):
  ```bash
  scp -i ~/.ssh/id_ed25519 file.zip user@host:/path/
  ```
- Enable compression (helpful over slow links):
  ```bash
  scp -C file.zip user@host:/path/
  ```
- Preserve timestamps and modes:
  ```bash
  scp -p file.zip user@host:/path/
  ```
- Limit bandwidth (Kb/s):
  ```bash
  scp -l 5000 file.zip user@host:/path/
  ```
- Quote paths with spaces:
  ```bash
  scp "My File.zip" user@host:"/path with space/"
  ```

## Install scp on Ubuntu
`scp` is included with the `OpenSSH` client on Ubuntu. If it’s missing:

1. Open a terminal
2. Update package list
   ```bash
   sudo apt update
   ```
3. Install OpenSSH client
   ```bash
   sudo apt install openssh-client
   ```
4. Verify installation
   ```bash
   command -v scp && ssh -V
   ```

## Common errors and fixes
- Permission denied (publickey): ensure your private key permissions are strict:
  ```bash
  chmod 600 ~/.ssh/id_*
  ```
- Connection times out or refused: check firewall/security groups and confirm the SSH port, then use `-P PORT`.
- Host key verification failed: verify the server’s fingerprint and update `~/.ssh/known_hosts` if the server legitimately changed (e.g., `ssh-keygen -R host && ssh host`).

## References
- OpenSSH scp manual: https://man.openbsd.org/scp