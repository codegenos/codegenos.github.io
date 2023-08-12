---
author: "CodeGenos"
title: "Firefox Executable File not Found in $PATH for Firefox Flatpak Installation on Linux"
date: 2023-08-08T23:58:39+03:00
description: "This article is about the firefox executable file not found in $PATH error fix when firefox is installed with flatpak."
tags: ["Linux", "Flatpak", "Firefox"]
categories: ["Firefox", "Linux"]
ShowToc: true
ShowBreadCrumbs: true
draft: false
---

## Intro
When an application is installed via `apt` package management tool, you can start running the application when you type the application name in terminal, for example `firefox`.

But when you install the application with Flatpak utility you must run the Flatpak application using this command:

```bash
flatpak run org.mozilla.firefox
```

---

## Problem
But when i want to authenticate my command-line sessions with **GitHub** using **Github CLI Tool** command `gh auth login` with `HTTPS` protocol and `Login with a web browser` options i get this error:

```text
! Failed opening a web browser at https://github.com/login/device
  exec: "firefox": executable file not found in $PATH
  Please try entering the URL in your browser manually
```

Flatpak stores installed applications in `/var/lib/flatpak/app` folder [Lubuntu 23.04]. However, this folder is not included in the `PATH` when you install the applications with flatpak. So you cannot run firefox using `firefox` command.

## Fix

In order to fix this issue you can create a bash file in a folder that is listed in the `PATH`.

```console
echo $PATH
```

the output is:

```console
/usr/local/sbin:/usr/local/bin:/usr/sbin:/usr/bin:/sbin:/bin
```

We will create the `firefox` bash file that will execute `flatpak run org.mozilla.firefox` command in the `/usr/local/bin` folder (the file: `/usr/local/bin/firefox`):

### 1. Go to the folder

```console
cd /usr/local/bin
```

### 2. Create firefox file using nano:

```console
sudo nano firefox
```

paste the following content then save and exit:

```bash
#!/bin/bash

flatpak run org.mozilla.firefox $@
```

>`$@` is a special variable that represents all the positional parameters passed to the script. This gets all the arguments passed to `firefox` and gives them to `flatpak run org.mozilla.firefox`. `gh auth login` opens the authentication page with this command: `firefox "https://github.com/login/device"`. So `$@` will be used for passing the URL argument

### 3. Make the file executable

```console
sudo chmod +x firefox
```

Now you can use `firefox` command to open Firefox from terminal. And also:

```console
firefox "https://github.com"
```

### Finally the problem is fixed

```console
? What account do you want to log into? GitHub.com
? You're already logged into github.com. Do you want to re-authenticate? Yes
? What is your preferred protocol for Git operations? HTTPS
? How would you like to authenticate GitHub CLI? Login with a web browser

! First copy your one-time code: XXXX-XXXX
Press Enter to open github.com in your browser... 
✓ Authentication complete.
- gh config set -h github.com git_protocol https
✓ Configured git protocol
✓ Logged in as xxxxxx
```



