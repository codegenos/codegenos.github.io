---
author: "CodeGenos"
title: "Managing Multiple Github Accounts with Git Credential Manager"
date: 2023-08-13T17:53:26+03:00
description: "How to manage multiple github accounts with git credential manager?"
tags: ["Github", "Git", "Git Credential Manager"]
categories: ["Github"]
ShowToc: true
ShowBreadCrumbs: true
draft: false
---

If you have more than one github account and want to contribute your projects from one computer, you can manage accounts using `git credential manager`.

## Step 1
---

Install `git credential manager`. (You can read <a href="https://github.com/git-ecosystem/git-credential-manager/blob/release/docs/install.md" target="_blank">install instructions</a>)


For debian users, download the latest <a href="https://github.com/git-ecosystem/git-credential-manager/releases/latest" target="_blank">.deb package</a>, and run the following:

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
>`Configuring component 'Git Credential Manager'...`
>`Configuring component 'Azure Repos provider'...`

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
git config user.name "youremail@yourdomain.com"
git config user.email "youremail@yourdomain.com"
```

## Result

Now when you push your commits to the github repository "Connect to Github" popup appears. In the "Connect to Github" popup, when you click button, the github login page will be opened in your default web browser. After you enter username and password, you will be able to push the commits successfully.