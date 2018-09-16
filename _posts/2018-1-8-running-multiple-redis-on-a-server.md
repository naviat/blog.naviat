---
layout: post
tags: Redis
title: How to setup and run multiple Redis server instances on a Linux host
---
### How to setup and run multiple Redis server instances on a Linux host

#### Environment

Redis

Linux

#### Purpose

Users can easily setup and run multiple Redis instances on the same Linux host. We have a Redis instance running on Ubuntu 14.04, and we want to setup a second Redis instance. 

##### 1.  Existing Redis Server1 instance running on port 6379 using

```
/etc/redis/redis_6379.conf configuration file 
/etc/init.d/redis_6379 script
```
##### 2.  Create a new (2nd) Redis Server2 instance running on port 6380 using
```
/etc/redis/redis_6380.conf configuration file
/etc/init.d/redis_6380 script 
```
#### Instructions

###### 1)  Setup a second Redis instance configuration file
```
cp /etc/redis/redis_6379.conf /etc/redis/redis_6380.conf
```
###### 2)  Change the following lines in /etc/redis/redis_6380.conf
```
pidfile /var/run/redis_6379.pid
port 6379
logfile /var/log/redis/redis_6379.log
dir /var/lib/redis/6379
```
To this:
```
pidfile /var/run/redis_6380.pid
port 6380
logfile /var/log/redis/redis_6380.log
dir /var/lib/redis/6380
```
###### 3)  Create a second Redis instance working directory
```
mkdir /var/lib/redis/6380
```
###### 4) Change the following line in /etc/init.d/redis_6379 
```
> EXEC=/usr/local/bin/redis-server        (no change here)
CLIEXEC=/usr/local/bin/redis-cli         (no change here)
PIDFILE=/var/run/redis_6379.pid
CONF="/etc/redis/6379.conf";
REDISPORT="6379"
```
To this:
```
NAME=`basename ${0}`
EXEC=/usr/local/bin/redis-server       (no change here)
CLIEXEC=/usr/local/bin/redis-cli        (no change here)
PIDFILE=/var/run/${NAME}.pid
CONF="/etc/redis/${NAME}.conf"
REDISPORT="${NAME#*_}"
```
###### 5) Create a symlink script for the second server instance
```
ln -s /etc/init.d/redis_6379 /etc/init.d/redis_6380
```
###### 6) Now we can start up and use both server instances
```
/etc/init.d/redis_6379 start
/etc/init.d/redis_6380 start
> redis-cli -p 6379 info server | egrep "process_id|tcp_port|config_file"
process_id:8329
tcp_port:6379
config_file:/etc/redis/redis_6379.conf

> redis-cli -p 6380 info server | egrep "process_id|tcp_port|config_file"
process_id:5809
tcp_port:6380
config_file:/etc/redis/redis_6380.conf
```
