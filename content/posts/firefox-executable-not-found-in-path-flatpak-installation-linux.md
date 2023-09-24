---
author: "CodeGenos"
title: "Firefox Executable File not Found in $PATH for Flatpak on Linux"
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
But when i want to authenticate my command-line sessions with **GitHub** using **Github CLI Tool** command `gh auth login` with `HTTPS` protocol and `Login with a web browser` options, i get this error:

```text
! Failed opening a web browser at https://github.com/login/device
  exec: "firefox": executable file not found in $PATH
  Please try entering the URL in your browser manually
```

> **Note**: At first the firefox browser was installed using Snap packages. I uninstalled the Snap package and installed the firefox browser Flatpak package. I think i am getting this error because of this.

I get this error because github CLI tool is trying to run `firefox <url>` command and cannot find executable file not in $PATH. 

When i looked at the github CLI source code and the [browser_linux.go](https://github.com/cli/browser/blob/main/browser_linux.go) file, is saw that the `OpenBrowser` method is calling the first available "xdg-open", "x-www-browser", "www-browser", "wslview" command to open "https://github.com/login/device" url. I think xdg-open command is running the `firefox` command.

Flatpak stores installed applications in `/var/lib/flatpak/app` folder [Lubuntu 23.04]. However, this folder is not included in the `PATH` when you install the applications with flatpak. So you cannot run firefox using `firefox` command.

## Fix

In order to fix this issue we will create a bash file in a folder that is listed in the `PATH`.

You can see the folders that are included in `PATH` using  `echo $PATH` command in linux:

```console
echo $PATH
```

the output is:

```console
/usr/local/sbin:/usr/local/bin:/usr/sbin:/usr/bin:/sbin:/bin
```

The `PATH` has `/usr/local/sbin:/usr/local/bin:/usr/sbin:/usr/bin:/sbin:/bin` folders. So i decided to use `/usr/local/bin` folder.

We will create the `firefox` bash file that will execute `flatpak run org.mozilla.firefox` command in the `/usr/local/bin` folder (the bash file will be: `/usr/local/bin/firefox`).

### 1. Go to the folder

Go to the `/usr/local/bin` folder:

```console
cd /usr/local/bin
```

### 2. Create firefox file using nano:

Create the firefox bash file using nano:

```console
sudo nano firefox
```

And then paste the following content then save and exit:

```bash
#!/bin/bash

flatpak run org.mozilla.firefox $@
```

>`$@` is a special variable that represents all the positional parameters passed to the script. This gets all the arguments passed to `firefox` and passed all the arguments to `flatpak run org.mozilla.firefox` command. `gh auth login` opens the authentication page with this command: `firefox "https://github.com/login/device"`. So `$@` will be used for passing the URL argument

### 3. Make the file executable

You should make the firefox bash file executable otherwise you will not be able to run it. To make a file executable in Linux, you need to use the `chmod` command, which stands for "change mode." The chmod command allows you to change the permissions of a file. Here is how you can make a file executable:

```console
sudo chmod +x firefox
```

The `+x` is an option used to grant execute permission to a file. The `+` symbol indicates that you are adding permission. The `x` stands for the execute permission.

Now you can use `firefox` command to open Firefox from terminal. The second parameter is the URL that you want to open in browser.

```console
firefox "https://github.com"
```

If this command works successfully then the `gh auth login` with `Login with a web browser` options will work successfully too.

### Finally the problem is fixed

Run `gh auth login` command. Then choose `HTTPS` protocol and `Login with a web browser` options. And then "Press Enter to open github.com in your browser" will be prompted. When you press enter, "https://github.com/login/device" URL will be opened in your firefox browser. When you enter the `one-time code` which is prompted in terminal in "Enter the code displayed on your device" textbox in the "Device Activation" Github page, press "Authorize Github" button and login with your github password, then your device will be connected to github. 

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

Thanks for reading.

Happy coding!

