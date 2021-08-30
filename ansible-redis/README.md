# ansible-redis

[![Build Status](https://travis-ci.org/DavidWittman/ansible-redis.svg?branch=master)](https://travis-ci.org/DavidWittman/ansible-redis) [![Ansible Galaxy](https://img.shields.io/badge/galaxy-DavidWittman.redis-blue.svg?style=flat)](https://galaxy.ansible.com/davidwittman/redis)

 - Ansible 2.4+
 - Compatible with most versions of Ubuntu/Debian and RHEL/CentOS 6.x

## Contents

 1. [Installation](#installation)
 2. [Getting Started](#getting-started)
  1. [Single Redis node](#single-redis-node)
  2. [Master-Slave Replication](#master-slave-replication)
  3. [Redis Sentinel](#redis-sentinel)
 3. [Advanced Options](#advanced-options)
  1. [Verifying checksums](#verifying-checksums)
  2. [Install from local tarball](#install-from-local-tarball)
  3. [Building 32-bit binaries](#building-32-bit-binaries)
 4. [Role Variables](#role-variables)

## Installation

``` bash
$ ansible-galaxy install davidwittman.redis
```

## Getting started

Below are a few example playbooks and configurations for deploying a variety of Redis architectures.

This role expects to be run as root or as a user with sudo privileges.

### Clone git repositry
```bash
git clone git@codeup.aliyun.com:60e566f6f00dfa2fa4cabdd6/ansible-redis.git /etc/ansible/roles/ansible-redis
```

### Single Redis node

Deploying a single Redis server node is pretty trivial; just add the role to your playbook and go. Here's an example which we'll make a little more exciting by setting the bind address to 127.0.0.1:

``` yml
---
- hosts: redis-test.sjhz.tk
  vars:
    - redis_bind: 10.230.7.8
  roles:
    - ansible-redis
```

``` bash
$ ansible-playbook -i redis-test.sjhz.tk, redis.yml
```

**Note:** You may have noticed above that I just passed a hostname in as the Ansible inventory file. This is an easy way to run Ansible without first having to create an inventory file, you just need to suffix the hostname with a comma so Ansible knows what to do with it.

That's it! You'll now have a Redis server listening on 127.0.0.1 on redis01.example.com. By default, the Redis binaries are installed under /opt/redis, though this can be overridden by setting the `redis_install_dir` variable.

**uninstall**
```
systemctl disable --now redis_6379
rm -f /etc/systemd/system/redis_6379.service
rm -rf /opt/redis
```

### Redis Sentinel

#### Introduction

Using Master-Slave replication is great for durability and distributing reads and writes, but not so much for high availability. If the master node fails, a slave must be manually promoted to master, and connections will need to be redirected to the new master. The solution for this problem is [Redis Sentinel](http://redis.io/topics/sentinel), a distributed system which uses Redis itself to communicate and handle automatic failover in a Redis cluster.

Sentinel itself uses the same redis-server binary that Redis uses, but runs with the `--sentinel` flag and with a different configuration file. All of this, of course, is abstracted with this Ansible role, but it's still good to know.

#### Configuration

ansible version = 5.0.13  
auth pass = 123456 
configure file is in ansible-redis/defaults/main.yml

To add a Sentinel node to an existing deployment, assign this same `redis` role to it, and set the variable `redis_sentinel` to True on that particular host. This can be done in any number of ways, and for the purposes of this example I'll extend on the inventory file used above in the Master/Slave configuration:

``` ini
[redis-master]
redis-master.sjhz.tk

[redis-slave]
redis-slave0[1:2].sjhz.tk

[redis-sentinel]
redis-sentinel0[1:3].sjhz.tk redis_sentinel=True
```

Above, we've added three more hosts in the **redis-sentinel** group (though this group serves no purpose within the role, it's merely an identifier), and set the `redis_sentinel` variable inline within the inventory file.

Now, all we need to do is set the `redis_sentinel_monitors` variable to define the Redis masters which Sentinel should monitor. In this case, I'm going to do this within the playbook:

``` yml
- name: configure the master redis server
  hosts: redis-master
  roles:
    - ansible-redis

- name: configure redis slaves
  hosts: redis-slave
  vars:
    - redis_slaveof: redis-master.sjhz.tk 6379
  roles:
    - ansible-redis

- name: configure redis sentinel nodes
  hosts: redis-sentinel
  vars:
    - redis_sentinel_monitors:
      - name: mymaster
        host: redis-master.sjhz.tk
        port: 6379
  roles:
    - ansible-redis
```

## sing redis with only ip address config
``` yml
---
- hosts: 10.230.7.14
  vars:
    - redis_bind: 10.230.7.14
  roles:
    - ansible-redis
```

``` bash
$ ansible-playbook -i 10.230.7.14, redis.yml  
```

## redis-sentinel with only ip address config


hosts.redis-sentinel
```
$ cat /etc/ansible/hosts
[redis-master]
192.168.177.40

[redis-slave]
192.168.177.41

[redis-sentinel]
192.168.177.42 redis_sentinel=True
```

redis.yml

```
~]# cat redis.yml
- name: configure the master redis server
  hosts: redis-master
  roles:
    - ansible-redis

- name: configure redis slaves
  hosts: redis-slave
  vars:
    - redis_slaveof: 192.168.177.40 6379
  roles:
    - ansible-redis

- name: configure redis sentinel nodes
  hosts: redis-sentinel
  vars:
    - redis_sentinel_monitors:
      - name: mymaster
        host: 192.168.177.40
        port: 6379
  roles:
    - ansible-redis
```

install
```
ansible-playbook -i hosts.redis-sentinel redis.yml 
```
