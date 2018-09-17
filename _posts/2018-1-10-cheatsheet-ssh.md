---
layout: post
tags: SSH
title: SSH Cheatsheet
---
### 1.1 SSH GENERAL

- Install SSH Server 
```diff
- $apt-get install openssh  
```
- Run ssh command	 
```diff
- $ssh -o StrictHostKeyChecking=no -p 2702 root@172.17.0.8 date  
```
- SSH with verbose ouptut		 
```diff
- $ssh -vvv -p 2702 root@45.33.87.74 date 2>&1  
```
- SSH passwordless login		 
```diff
- $ssh-copy-id <username>@<ssh_host>
```
, Or manually update 
```diff
- $~/.ssh/authorized_keys
```
