# https://github.com/CSC-IT-Center-for-Science/fgci-ansible/tree/openstack

This section is mostly out of date but it might work - See the above link for an updated way of using openstack with this role

ansible-role-slurm
------------------

[![Build Status](https://travis-ci.org/CSC-IT-Center-for-Science/ansible-role-slurm.svg?branch=master)](https://travis-ci.org/CSC-IT-Center-for-Science/ansible-role-slurm)

# Creates a slurm cluster

Tested with slurm versions:
 - 14.11.0
 - 14.11.3
 - 15.08.x

Tested with these linux distributions:
 - CentOS 6
  - Only 14.11.x
 - CentOS 7
  - Only 15.08.x

## How-To

### Initial playbook configuration

*Playbook variables:*

All variables should be defined in defaults/main.yml

All nodes run munge. Nodes which are part of the slurm\_compute host
group will additionally run slurmd. Nodes which are part of the
slurm\_service host group will additionally runs slurmctld and
slurmdbd.

You also need to add a mysql\_slurm\_password: "PASSWORD" string
somewhere. This will be used to set a password for the slurm mysql
user.


### Below instructions are for running slurm and perhaps multiple slurm instances in openstack

These instructions no longer work, files have been removed. A playbook that uses this role: https://github.com/CSC-IT-Center-for-Science/fgci-ansible

#### Initial configuration of the instances and your workstation:

 - First yum -y install nc on the bastion host.
 - Then setup SSH config so you don't have to have a public IP on each instance. Change the Hostname in "Host bastion" to the service node.

Put this in ~/.ssh/config :

<pre>
# http://edgeofsanity.net/article/2012/10/15/ssh-leap-frog.html
# This applies to all hosts in your ssh config.
ControlMaster auto
ControlPath ~/.ssh/ssh_control_%h_%p_%r

# Always ssh as cloud-user and don't save hostkeys
Host bastion
  User cloud-user
  StrictHostKeyChecking no
  UserKnownHostsFile /dev/null
  Hostname 86.50.168.39
     
Host slurm* 
   User cloud-user
   StrictHostKeyChecking no
   UserKnownHostsFile /dev/null
   ForwardAgent yes
   ProxyCommand ssh bastion nc %h %p
</pre>

 - Second update the /etc/hosts on the service node (playbook should update the others):

<pre>
# For second cluster
192.168.36.126 slurm2-compute1
192.168.36.125 slurm2-service
192.168.36.124 slurm2-login

# For first cluster
192.168.36.129 slurm-compute1
192.168.36.128 slurm-service
192.168.36.127 slurm-login
</pre>

### Then we can finally run the slurm configuration playbooks:

#### Description of the playbooks:

 - site.yml - calls the slurm*.yml playbooks
 - slurm_*.yml # The playbooks that configure the servers
  - set slurm_version - this is used to determine which version to download from schedmd.com

#### Run them in this order:

 - Update stage to have the right IP addresses and hostnames.
 - configuring 1st slurm: ansible-playbook site.yml

 - Update stage to have the right IP addresses and hostnames.
 - configuring 2st slurm: ansible-playbook site.yml

#### Add cloud-user to slurm

<pre>
sacctmgr create account name=csc
sacctmgr create user name=cloud-user account=csc
</pre>

### Make changes to slurm.conf and distribute it to nodes and restart/reconfigure:

 - Would be nice with a role /tag where one could just run ansible-playbook site.yml --tag new-slur-config and it pushes new config and restarts/reconfigs as necessary.

# Authors / Contributors:

 - Marco Passerini (original author)
 - https://github.com/martbhell
 - https://github.com/tiggi
 - https://github.com/A1ve5
