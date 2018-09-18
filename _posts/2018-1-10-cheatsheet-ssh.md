---
layout: post
tags: ssh
title: SSH Cheatsheet
---
### 1.1 SSH general

##### - Install ssh server	

   `apt-get install openssh`
   
##### - Run ssh command	

  ` ssh -o StrictHostKeyChecking=no -p 2702 root@172.17.0.8 date`
   
##### - SSH with verbose ouptut	

   `ssh -vvv -p 2702 root@45.33.87.74 date 2>&1`
   
##### - SSH passwordless login	

   `ssh-copy-id <username>@<ssh_host>, Or manually update ~/.ssh/authorized_keys`
     
##### - Remove an entry from known_hosts file	
     
   `ssh-keygen -f ~/.ssh/known_hosts -R github.com`
   
##### - Diff local file with remote one	
   
   `diff local_file.txt <(ssh <username>@<ssh_host> 'cat remote_file.txt')`
     
##### - Diff two remote ssh files	
    
   `diff <(ssh user@remote_host 'cat file1.txt') <(ssh user2@remote_host2 'cat file2.txt')`
    
##### - Upload with timestamps/permissions kept	
   
   `scp -rp /tmp/abc/ ec2-user@<ssh-host>:/root/`
   
##### - SSH agent load key	

   `exec ssh-agent bash && ssh-keygen, ssh-add`
   
##### - Emacs read remote file with tramp	

   `emacs /ssh:<username>@<ssh_host>:/path/to/file`
     
     
### 1.2 SCP

##### - Download a remote folder	

`scp -r ec2-user@<ssh-host>:/home/letsencrypt-20180825 ./`

##### - Upload a file	

`scp -i <ssh-keyfile> /tmp/hosts ec2-user@<ssh-host>:/root/`

##### - Upload a folder	

`scp -r /tmp/abc/ ec2-user@<ssh-host>:/root/`

##### - Upload with timestamps/permissions kept	

`scp -rp /tmp/abc/ ec2-user@<ssh-host>:/root/`

##### - Mount remote directory as local folder	

`sshfs name@server:/path/remote_folder /path/local_folder`
