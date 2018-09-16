---
layout: post
tags: Redis
title: How to setup Redis master and slave replication
---
### How to setup Redis master and slave replication
#### PURPOSE
It's very simple to setup a master-slave Redis replication that allows the slave Redis server to keep an exact backup copy of a master server database. For more details on Redis replication, refer here. 

For this demo, we have two Redis instances running the same Ubuntu 14.0.4 host, and we are going to configure redis_6379 server as a master and redis_6380 server as a slave. If you don't have Redis servers install yet, see How to setup and run multiple Redis server instances on a Linux host.

##### 1.  Redis Server1 (Master): Running on port 6379 using
```
/etc/redis/redis_6379.conf configuration file
/etc/init.d/redis_6379 service script
```
##### 2.  Redis Server2 (Slave): Running on port 6380 using
```
/etc/redis/redis_6380.conf configuration file
/etc/init.d/redis_6380 service script
```
#### CAUSE

#### PROCEDURE

##### 1)  Add the following lines
```
/etc/redis/redis_6380.conf  this Redis server an exact copy of a master server
# slaveof <masterip> <masterport>
slaveof localhost 6379
# setting a slave to authenicate to a master
masterauth mypass
```
##### 2) Setup a password required for login the master server in
```
/etc/redis/redis/redis_6379.conf
requirepass mypass
```
##### 3)  Restart the instances
```
/etc/init.d/redis_6379 stop/start
/etc/init.d/redis_6380 stop/start
```
####                                                        OR

You can use redis-cli with config set masterauth/requirepass and SLAVEOF commands without restarting

- Set up a required password login on the master
```
redis-cli -p 6379 config set requirepass mypass
```
- Set up a master password login on the salve
```
redis-cli -p 6380 config set masterauth mypass
```
- Making it a slave of redis_6379 master server
```
redis-cli -p 6380 SLAVEOF localhost 6379
```
- Get the master/salve replication information
```
redis-cli -p 6379 -a mypass info replication
# Replication
role:master
connected_slaves:1
slave0:ip=::1,port=6380,state=online,offset=29,lag=0
master_repl_offset:29
repl_backlog_active:1
repl_backlog_size:1048576
repl_backlog_first_byte_offset:2
repl_backlog_histlen:28

redis-cli -p 6380 info replication
# Replication
role:slave
master_host:localhost
master_port:6379
master_link_status:up
master_last_io_seconds_ago:7
master_sync_in_progress:0
slave_repl_offset:113
slave_priority:100
slave_read_only:1
connected_slaves:0
master_repl_offset:0
repl_backlog_active:0
repl_backlog_size:1048576
repl_backlog_first_byte_offset:0
repl_backlog_histlen:0
```
- Stop replication and turning a slave into master
```
​$redis-cli -p 6380 slaveof no one OK redis-cli -p 6380 info replication # Replication role:master connected_slaves:0 master_repl_offset:211 repl_backlog_active:0 repl_backlog_size:1048576 repl_backlog_first_byte_offset:0 repl_backlog_histlen:0
```
