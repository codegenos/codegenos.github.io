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

When you are already logged in to github from your default browser with another user other than the git repository's user, you cannot authenticate. You can force git credential manager to open your default browser in private mode.
