---
title: Switching between multiple ssh keys for git
author: ''
date: '2021-05-10'
slug: switching-between-multiple-ssh-keys-for-git
categories:
  - Guide
tags:
  - Git
  - Gitlab
  - GitHub
subtitle: ''
summary: ''
authors: []
lastmod: '2021-05-10T11:49:03+01:00'
featured: no
image:
  caption: ''
  focal_point: ''
  preview_only: no
projects: []
---


## Problem

Recently my company has switched code repositories, from Gitlab to GitHub. I already have a personal GitHub account so I find myself in a position where I have two GitHub accounts. I authenticate using ssh keys, but GitHub does not authenticate two separate accounts using the same keys. Despite generating a new ssh key for my new corporate account I found myself unable to access my organisations repositories.

After much scouring of [stack](https://stackoverflow.com/a/9680717/10960765) [overflow](https://stackoverflow.com/a/25924462/10960765) I found that the issue was down to the choice of ssh key used to access a remote git server. When you only have the one key, life is simple and there is no choice about which key git uses to connect to any remote server. However, for multiple keys we need to configure aliases for hosts that are associated with specific keys that you have generated.

## Solution

Edit your `~/.ssh/config` file (assuming the default location, where "~" means your home folder) with something that looks similar to below.

```bash
Host github-work
HostName github.com
User mywork_account
IdentityFile ~/.ssh/work_private_key
IdentitiesOnly yes

Host github-home
HostName github.com
User myhome_account
IdentityFile ~/.ssh/home_private_key
IdentitiesOnly yes
```

After editing your configuration file as above you can execute e.g., `git remote add origin git@github-work:path-to-my-repository.git`, to add a remote to your repository that refers to the correct alias. The above configuration is a thorough example but you could omit one alias ("github-home" for example) and any remote that does not include the reference to "github-work" would use the first key you generated (in this example, "home_private_key".)

