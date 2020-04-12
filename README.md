# Ansible environment setup with Docker

1. Create a ```docker-compose.yml``` file with (Ref: https://hub.docker.com/repository/docker/shahjalalh/ubuntu-ssh)
    1. Ansible Controller (Ubuntu 16.04) (install openssh-server and ansible)
    2. Target Server 1 (Ubuntu 16.04) (target1)
    3. Target Server 2 (Ubuntu 16.04) (target2)
    4. Target Server 3 (Ubuntu 16.04) (target3)
2. Create volume of "Ansible Controller" in local computer

**Note:** ```shahjalalh/ubuntu-ssh``` is prebuilt container openssh-server and Ansible installed

**Note:** only controller needs Ansible and openssh-server both installed. target1, target2 and target3 need only openssh-server installed. Because **Ansible is agent-less**. 

Updating known host
```
$ ssh-keygen -R controller-ip-address
```

Inspecting IP address of the container
```
$ docker inspect 512

...
...
"Networks": {
    "ansible-with-docker_net": {
        "IPAMConfig": null,
        "Links": null,
        "Aliases": [
            "512ed9316196",
            "controller"
        ],
        "NetworkID": "20cbaf946ef9fb37a1e570ef81093d5701487bf29391e5f37cea6927febeb26d",
        "EndpointID": "e279a02304c797497ec20486c0a93d0006971c820b6b5a4696e0655c4f907f5d",
        "Gateway": "172.20.0.1",
        "IPAddress": "172.20.0.2",
        "IPPrefixLen": 16,
        "IPv6Gateway": "",
        "GlobalIPv6Address": "",
        "GlobalIPv6PrefixLen": 0,
        "MacAddress": "02:42:ac:14:00:02",
        "DriverOpts": null
    }

```

```"Gateway": "172.20.0.1", "IPAddress": "172.20.0.2",``` in Network section. Now ssh to the ip
```
$ ssh root@172.20.0.2
```

Running all together
```
$ docker-compose up
$ docker-compose ps
 Name          Command        State   Ports 
--------------------------------------------
ansicon   /usr/sbin/sshd -D   Up      22/tcp
target1   /usr/sbin/sshd -D   Up      22/tcp
target2   /usr/sbin/sshd -D   Up      22/tcp
target3   /usr/sbin/sshd -D   Up      22/tcp

$ docker ps
CONTAINER ID        IMAGE                       COMMAND               CREATED             STATUS              PORTS               NAMES
1c9df5b4a4fc        shahjalalh/ubuntu-ssh:1.0   "/usr/sbin/sshd -D"   2 minutes ago       Up 2 minutes        22/tcp              target3
4db1bc6f7d3f        shahjalalh/ubuntu-ssh:1.0   "/usr/sbin/sshd -D"   2 minutes ago       Up 2 minutes        22/tcp              target1
a49c57cbb1f1        shahjalalh/ubuntu-ssh:1.0   "/usr/sbin/sshd -D"   2 minutes ago       Up 2 minutes        22/tcp              ansicon
ba3ab0bc1d73        shahjalalh/ubuntu-ssh:1.0   "/usr/sbin/sshd -D"   2 minutes ago       Up 2 minutes        22/tcp              target2
$ docker inspect target3
...
"IPAddress": "172.19.0.2",
...
$ docker inspect target1
...
"IPAddress": "172.19.0.3",
...
$ docker inspect ansicon
...
"IPAddress": "172.19.0.4",
...
$ docker inspect target2
...
"IPAddress": "172.19.0.5",
...

```

Now try ssh to each one:
```
$ ssh root@172.19.0.2
$ ssh root@172.19.0.3
$ ssh root@172.19.0.4
$ ssh root@172.19.0.5
```

Running ansible play-book inside controller container - 
```
root@1fbcf9ee4ee7:/home/workspace# ansible target* -m ping -i inventory.txt 
[DEPRECATION WARNING]: Distribution Ubuntu 16.04 on host target3 should use /usr/bin/python3, but is using /usr/bin/python for backward compatibility with prior 
Ansible releases. A future Ansible release will default to using the discovered platform python for this host. See 
https://docs.ansible.com/ansible/2.9/reference_appendices/interpreter_discovery.html for more information. This feature will be removed in version 2.12. Deprecation 
warnings can be disabled by setting deprecation_warnings=False in ansible.cfg.
target3 | SUCCESS => {
    "ansible_facts": {
        "discovered_interpreter_python": "/usr/bin/python"
    }, 
    "changed": false, 
    "ping": "pong"
}
[DEPRECATION WARNING]: Distribution Ubuntu 16.04 on host target1 should use /usr/bin/python3, but is using /usr/bin/python for backward compatibility with prior 
Ansible releases. A future Ansible release will default to using the discovered platform python for this host. See 
https://docs.ansible.com/ansible/2.9/reference_appendices/interpreter_discovery.html for more information. This feature will be removed in version 2.12. Deprecation 
warnings can be disabled by setting deprecation_warnings=False in ansible.cfg.
target1 | SUCCESS => {
    "ansible_facts": {
        "discovered_interpreter_python": "/usr/bin/python"
    }, 
    "changed": false, 
    "ping": "pong"
}
[DEPRECATION WARNING]: Distribution Ubuntu 16.04 on host target2 should use /usr/bin/python3, but is using /usr/bin/python for backward compatibility with prior 
Ansible releases. A future Ansible release will default to using the discovered platform python for this host. See 
https://docs.ansible.com/ansible/2.9/reference_appendices/interpreter_discovery.html for more information. This feature will be removed in version 2.12. Deprecation 
warnings can be disabled by setting deprecation_warnings=False in ansible.cfg.
target2 | SUCCESS => {
    "ansible_facts": {
        "discovered_interpreter_python": "/usr/bin/python"
    }, 
    "changed": false, 
    "ping": "pong"
}
root@1fbcf9ee4ee7:/home/workspace#
```

