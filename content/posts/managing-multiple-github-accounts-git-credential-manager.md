---
author: "CodeGenos"
title: "Managing Multiple Github Accounts with Git Credential Manager"
date: 2023-08-13T17:53:26+03:00
description: "How to manage multiple github accounts with git credential manager?"
tags: ["Github", "Git", "Git Credential Manager"]
categories: ["Github"]
ShowToc: false
ShowBreadCrumbs: true
draft: false
---

If you have more than one github account and want to contribute your projects from one computer, you can manage accounts using `git credential manager`.

## Step 1
---

Install `git credential manager`. (You can read [install instructions](https://github.com/git-ecosystem/git-credential-manager/blob/release/docs/install.md))

For debian users, download the latest [.deb package](https://github.com/git-ecosystem/git-credential-manager/releases/latest), and run the following:

```bash
sudo dpkg -i <path-to-package>
```

## Step 2
---

Configure `git credential manager`:

```bash
git-credential-manager configure
```

###### Output
>```bash
>Configuring component 'Git Credential Manager'...
>Configuring component 'Azure Repos provider'...
>```

## Step 3
---

To configure Git to cache credentials for the full remote URL of each repository you access on GitHub, enter the following command.

```bash
git config --global credential.https://github.com.useHttpPath true
```

## Step 4
---

Set credential store type. I use cache credential store which uses Git's built-in credential cache.

```bash
git config --global credential.credentialStore cache
```

## Step 5
---

Set user name and email for each of your git repositories.

```bash
git config user.name "Your Name"
git config user.email "youremail@yourdomain.com"
```

## Result

Now when you push your commits to the github repository "Connect to Github" popup appears. 

![Connect to Github Popup](/posts/images/connect-to-github-popup.jpg)

In the "Connect to Github" popup, when you click "Sign in with your browser" button, the github login page will be opened in your default web browser. 

![Sign in to Github to continue to Git Credential Manager](/posts/images/sign-in-to-github-gcm-browser.jpg)

After you enter username and password, the authentication will be succeeded and then you will be able to push the commits successfully.

![Git Credential Manager Github Authentication succeeded](/posts/images/gcm-github-auth-success.jpg)

## How to Open Github Login Page in New Private Window for firefox?
When you are already logged in to github from your default browser with another user other than the git repository's user, you cannot authenticate. You can force git credential manager to open your default browser in private window (incognito window). Git credential manages uses `xdg-open` to open the github login page in the browser. You can see the source code [here](https://github.com/git-ecosystem/git-credential-manager/blob/main/src/shared/Core/BrowserUtils.cs#L75).

>`xdg-open` is a command-line utility that is used in Linux and other Unix-like operating systems to open files and URLs using the default applications set by the user's desktop environment. It is part of the XDG (Desktop Entry Specification) framework, which aims to standardize file and URL handling across different desktop environments.

I'm using firefox (flatpak) so I'll show you how to make `xdg-open` open URL's in new firefox private window. In order to do that:

### 1. Create org.mozilla.firefox-private.desktop

```bash
sudo cp /var/lib/flatpak/app/org.mozilla.firefox/current/active/export/share/applications/org.mozilla.firefox.desktop /usr/share/applications/org.mozilla.firefox-private.desktop
```

Edit `/usr/share/applications/org.mozilla.firefox-private.desktop`:
- add `--private-window` to `Exec=`
- set `NoDisplay=true`
- set `Hidden=true`.

`/usr/share/applications/org.mozilla.firefox-private.desktop` content should be:
```
Exec=/usr/bin/flatpak run --branch=stable --arch=x86_64 --command=firefox --file-forwarding org.mozilla.firefox --private-window @@u %u @@
Icon=org.mozilla.firefox
Terminal=false
Type=Application
MimeType=text/html;text/xml;application/xhtml+xml;application/vnd.mozilla.xul+xml;text/mml;x-scheme-handler/http;x-scheme-handler/https;
StartupNotify=true
Categories=Network;WebBrowser;
StartupWMClass=firefox
X-Flatpak=org.mozilla.firefox
NoDisplay=true
Hidden=true
```

>**NoDisplay:** The NoDisplay entry is used to indicate whether an application should be displayed in menus and application launchers. If `NoDisplay=true` is set in a `.desktop` file, the application won't appear in user-facing menus or lists. This is typically used for applications that are meant to be run in the background or that don't have a direct user interface.
>**Hidden:** The Hidden entry is used to indicate that an application should be hidden from the user interface entirely. Unlike NoDisplay, which still allows the application to be launched if you know its name, ´Hidden=true´ ensures that the application is entirely hidden from the user.

### 2. Set xdg-mime default for x-scheme-handler/https and x-scheme-handler/http

```bash
xdg-mime default org.mozilla.firefox-private.desktop x-scheme-handler/https
```

```bash
xdg-mime default org.mozilla.firefox-private.desktop x-scheme-handler/http
```

#### Note
>Other programs using `xdg-open` to open Urls will open the urls in new private window after you make these changes in `xdg-mime`. 
