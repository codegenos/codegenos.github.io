---
author: "CodeGenos"
title: "How to Transfer Files with Scp Tool on Linux"
date: 2023-09-08T21:28:35+03:00
description: "In this article i'll show you how to transfer files and folders using `scp` command-line tool on Linux"
tags: ["Linux", "ssh", "scp"]
categories: ["Linux", "ssh"]
ShowToc: false
ShowBreadCrumbs: true
draft: false
---

The other day I needed to copy files from my local machine to my `Raspberry Pi` with `DietPi` without desktop installed. My only method of accessing the Raspberry Pi was via `ssh`. I wanted to transfer files in the simplest way on terminal. I searched how to make this work and found the `scp` tool. 

In this article i'll show you how to transfer files and folders using `scp` command-line tool.

## SCP command
`scp` (Secure Copy Protocol) is a command-line tool that allows you to securely transfer files and folders between different machines over an `SSH` (Secure Shell) connection. 

It uses the same authentication and security as SSH to ensure that your data is transferred securely.

You can both transfer between two remote hosts and from a local machine to remote host or from remote host to local machine.

## Copy files with SCP command
SCP lets you securely transfer files between two remote hosts or a remote machine.

### Copy file from Local Machine to Remote Server
Here's how you can copy a file from your local machine to a remote server:

```bash
scp /path/to/local/file.zip username@remote_server:/path/to/destination/folder
```

- `/path/to/local/file.zip`: Replace this with the path to the local file you want to copy.
- `username`: Replace this with your username on the remote server.
- `remote_server`: Replace this with the hostname or IP address of the remote server.
- `/path/to/destination/folder`: Replace this with the path to the destination folder on the remote server.

You will be prompted for the password of the remote user, and once you enter it correctly, the file will be securely copied to the remote server folder destination.

### Copy file from Remote Server to Local Machine
If you want to copy a file from the remote server to your local machine, you can reverse the source and destination paths in the scp command:

```bash
scp username@remote_server:/path/to/destination/file.zip /path/to/local/folder
```

### Copy file from Remote Server to Remote Server
You can also copy from Remote Server to Remote Server:

```bash
scp username1@remote_server1:/path/to/destination/file.zip username2@remote_server2:/path/to/destination/folder
```

## Copy folders with SCP command
You can also copy the contents inside a folder instead of copying all files one by one.

### Copy folder from Local Machine to Remote Server
Here's how you can copy a folder from your local machine to a remote server:

```bash
scp -r /path/to/local/folder username@remote_server:/path/to/destination/folder
```

- `-r`: This option is used to recursively copy the entire folder and its contents.
- `/path/to/local/folder`: Replace this with the path to the local folder you want to copy.
- `username`: Replace this with your username on the remote server.
- `remote_server`: Replace this with the hostname or IP address of the remote server.
- `/path/to/destination/folder`: Replace this with the path to the destination folder on the remote server.

You will be prompted for the password of the remote user, and once you enter it correctly, the folder and its contents will be securely copied to the remote server folder destination.

### Copy folder from Remote Server to Local Machine
If you want to copy a folder from the remote server to your local machine, you can reverse the source and destination paths in the scp command:

```bash
scp -r username@remote_server:/path/to/remote/folder /path/to/local/folder
```

### Copy folder from Remote Server to Remote Server
You can also copy from Remote Server to Remote Server:

```bash
scp -r username1@remote_server1:/path/to/remote/folder username2@remote_server2:/path/to/remote/folder
```

## Install scp on Ubuntu
To use `scp` on Ubuntu, you don't need to install it separately because it is included by default with the `OpenSSH` package, which is a standard component of most Linux distributions, including Ubuntu. 

However, if for some reason you don't have `scp` installed, you can install it along with the full `OpenSSH` suite using the following steps:

1. **Open a Terminal**
2. **Update the Package List**
```bash
sudo apt update
```
3. **Install OpenSSH Client**
```bash
sudo apt install openssh-client
```

This command will install the `scp` tool along with other SSH client utilities.

4. **Verify the Installation**

```bash
scp --version
```

Thanks for reading.

Happy coding!