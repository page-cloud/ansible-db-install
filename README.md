## Updated June 28th 2017
Updated ad Oct 30 2024

# Ansible Install of SQL Server for Linux Always-On with READ SCALING ONLY


The purpose of the ansible sql server project is to install the latest version of SQL Server for Linux with Always-On on a CentOS7 server. 

## Running Playbooks

- Install sqlcmd
``` ansible-playbook dbclient.yml ```

- Install MSSQL server + sql server Agent + setup firewall
```ansible-playbook dbserver.yml```

- Install MSSQL Server HA components + setup firewall for 5022
```ansible-playbook dbserver_ha.yml```

- Create AlwaysOn Availability Group NoCluster ReadOnly mirror between two servers
```ansible-playbook dbserver_ag.yml```

The result of these four plays is a functional mirror between a principal and secondary server using SQL Servers AlwaysOn Technology. 
You can install dbclient and dbserver if you just want to have the latest version of Microsoft's Linux version of SQL Server. 
Some assumptions are made, such as using key based ssh authentification between hosts. We also assume that you will be using the 
ansible server for sqlcmd commands. 

## ANNEX

Vagrant + virtualbox was used as the environment for the SQL Server ansible project. 
You don't have to use vagrant, and this project is about installing SQL Server for Linux and not vagrant nor virtualbox.

## Install Vagrant + VirtualBox
Notice that the VagrantFile passes the id_rsa.pub to all hosts. This avoids having to add the ssh key to authorized_keys before running ansible.
Vagrant was also used to enable key authentification between servers.  
This version of ansible-sqlserver assumes that ssh is setup and using key authentification. 

This is the version of Ansible and Python we used:

```
ansible 2.3.0.0
  config file = 
  configured module search path = Default w/o overrides
  python version = 2.7.5 (default, Nov  6 2016, 00:28:07) [GCC 4.8.5 20150623 (Red Hat 4.8.5-11)]
```


To install the latest version of ansible please follow this link:http://docs.ansible.com/ansible/intro_installation.html#latest-releases-via-pip

Here is what we did to install the lastest version of ansible. 

### INSTALL PYTHON
1. ```yum install yum-utils```
2. ```yum-config-manager --add-repo=https://dl.fedoraproject.org/pub/epel/7/x86_64/```
3. ```yum install wget```
4. ```wget -O /etc/pki/rpm-gpg/RPM-GPG-KEY-EPEL-7 https://dl.fedoraproject.org/pub/epel/RPM-GPG-KEY-EPEL-7```
5. ```rpm --import /etc/pki/rpm-gpg/RPM-GPG-KEY-EPEL-7```
6. ```yum install -y python-devel libffi-devel openssl-devel gcc python-pip redhat-rpm-config```

### INSTALL ANSIBLE
1. ```pip install --upgrade pip```
2. ```pip install paramiko```
3. ```pip install ansible```


### PASSWORD STORE
1. ```yum install pass```
2. ```gpg2 --gen-key (will take a long time to salt encryption key if this is the first time you are using)```
3. ```gpg --list-keys```
4. ```rm -rf ~/.password-store (option to reinitalize password store)```

Now create the passwordstore database and insert a SA password for the principal and mirror servers.  
In this version of ansible-sqlserver, db1 is required to be the principal.
1. pass init "key_id_from_list-keys"
2. pass insert MSSQL/db1/SA
3. pass insert MSSQL/db2/SA


Now install ansible-sqlserver

ansible-galaxy install username.rolename

There are several variable files that need to be updated.

dbserver
 vars
   main.yml (sa password) 
dbserver_ag
  vars
   main.yml (sa_password) This is the password for the mirror private key and mirror login.
   
Once you have completed the setup, you should test ansible with the ping module.  

```
ansible-sqlserver]$ ansible -m ping db1
db1 | SUCCESS => {
    "changed": false, 
    "ping": "pong"
}
```
