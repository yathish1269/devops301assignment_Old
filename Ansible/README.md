# Installing Ansible on Ubuntu 16.04
```
# apt-get update 

# apt-get install sshpass ansible -y 
```

```
# ansible --version
ansible 2.0.0.2
  config file = /etc/ansible/ansible.cfg
  configured module search path = Default w/o overrides
```


# Setting up Docker Container as ansible remote servers

## Create a Ubuntu 16.04 Container.
```
docker run -itd --name ansible-R1 ubuntu:16.04 /bin/bash
docker attach ansible-R1
```

## Install Required Packages for Ansible Remote Server ( Python & OpenSSH Server/Clinet ) 
```
apt-get update ; apt-get install python -y
apt-get install openssh-server openssh-client -y
```

## Setup the Root Password for remote Login as redhat
```
passwd 
```

## Configure SSH Service to allow Password Based Auth via Root User. 
```
sed -i "s/#PasswordAuthentication yes/PasswordAuthentication yes/g" /etc/ssh/sshd_config
sed -i "s/PermitRootLogin prohibit-password/PermitRootLogin yes/g" /etc/ssh/sshd_config
```
```
grep -i "PasswordAuthentication" /etc/ssh/sshd_config
grep -i "PermitRootLogin" /etc/ssh/sshd_config
```

## Start the Openssh Service.
```
service ssh restart
```

## Make Sure your SSH Service will come up after a Restart.
```
echo "service ssh restart" >> /root/.bashrc
```

## Now Come out of the container  & Test the connection 
```
docker inspect ansible-R1
ssh root@<ip>
```

## Commit the Container & get the ansible ready image. 
```
docker commit -p -m  "SSH ReadyImage" ansible-R1 ansible-ready-image:v1
```

## Now you can start multiple containers. 
```
docker run -itd --name ansible-R2 ansible-ready-image:v1 /bin/bash
docker run -itd --name ansible-R3 ansible-ready-image:v1 /bin/bash
```

## Update your inventory with container IP Address & Variable Password. ( I.E Below ) 
```
# cat inventory
...
[docker-container]
172.17.0.2
172.17.0.3
172.17.0.4


[docker-container:vars]
ansible_ssh_user=root
ansible_ssh_pass=redhat
```

