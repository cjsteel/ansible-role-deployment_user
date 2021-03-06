
ansible-role-deployment_user
============================

An Ansible role to setup a (system) deployment user and (system) deployment group on targeted systems.

## Description

The **deployment_user** Ansible role prepares nodes for usage with Ansible by setting up a deployment user as well as a public key for the Ansible controller in order to allow the controller to connect to the remote system using SSH keypairs. You may want to consider using a passphrase protected keypair with a strong pass phrase loaded to memory using ssh-agent and the ssh-agent or ssh-add -t (life) option.

    -t life
      Set a default value for the maximum lifetime of identities added to the agent.  The lifetime
      may be specified in seconds or in a time format specified in sshd_config(5).  A lifetime
      specified for an identity with ssh-add(1) overrides this value.  Without this option the
      default maximum lifetime is forever.


Under Development
-----------------

This role is considered to be stable.

## Target system preparations

The deployment_user role Before applying this role 

- Backup any data.
- Disconnect from network.
- Secure boot considerations
  - Third party drivers / signing?
- FFR (FDISK, Format, Reload OS)
- Create an administrative user with a good password

#### 


Requirements
------------

#### users module

The Ansible user module requires the following on the target OS to support the Ansible **user** module:

```shell
useradd
userdel
usermod
```

####  setup firewall on target system(s)

network cable still unplugged

```shell
sudo ufw enable
sudo ufw allow ssh
```

#### install openssh-server

* Plug in network cable
* install opesh-server so that the Ansible controller can connect to the system:

```shell
sudo apt-get update
sudo apt-get install openssh-server -y
sudo ufw enable
sudo ufw allow ssh
```
### Ansible controller configuration

#### /etc/hosts

Add the new workstations DHCP assigned IP address to `/etc/hosts` so our controller can find it.

```shell
sudo nano /etc/hosts
```

#####  Append example

Append an entry to your Ansible controllers hosts file so the controller can find the system. Make sure to separate the IP address from the hostname using a tab.

```txt
192.168.11.22 workstation-001
```

### Setup ssh agent

```shell
eval `/usr/bin/ssh-agent -s`
/usr/bin/ssh-add
```

### Confirm connectivity

#### remove any stale keys

```shell
ssh-keygen -f "$HOME/.ssh/known_hosts" -R workstation-001
ssh-keygen -f "$HOME/.ssh/known_hosts" -R 192.168.11.22
```

#### test connection

```shell
ssh <adminuser>@workstation-001
exit
```

#### copy over ssh public key

```shell
$
ssh-copy-id <adminuser>@workstation-001
exit
```
#### test connection using ssh keys

### Confirm

```shell
ssh <adminuser>@workstation-001
```

### group_vars/deployment_user/defaults.yml

Before creating our role's playbook we will set values for at least the following three vars in `group_vars/deployment_user/defaults.yml`:

```shell
mkdir -p group_vars/deployer
nano group_vars/deployer/defaults.yml
```

Add your deployment user name and uid as desires

    deployment_user_username: deploy
    deployment_user_uid: 888

and

    deployment_user_state: present

#### Create group_vars directory

First we create a directory to hold the group_vars related to our role.

    mkdir -p group_vars/deployment_user

#### Copy the example file.

    cp roles/deployment_user/files/group_vars/deployment_user/defaults.yml group_vars/deployment_user/.

#### Edit and save

Next we will edit the file and set our deployment users username, user id and the users state (present or absent).

    nano group_vars/deployment_user/defaults.yml

#### Minimal content example

```yaml
# file: group_vars/deployment_user/defaults.yml
#
# Used to overide role/deployment_user/defaults/main.yml vars
# so that our deployment user is not commited to the roles repository.

deployment_user_username : 'your_deployment_user'
deployment_user_uid	     : '808'
deployment_user_state    : 'present'
```
## Playbooks

```shell
### Main playbook

Example of a minimal main playbook that contains an include for our roles' playbook:

​```yaml
---
- hosts: all
  become: false

- include: deployment_user.yml
```

Copy and edit included example if desired:

```
cp roles/deployment_user/files/systems.yml .
nano systems.yml
```

### Roles' playbook

Example of our roles' playbook

```yaml
---
- hosts: deployment_user
  user: '{{ deployment_user_username }}'
  become: true
  gather_facts: true
  roles:
    - deployment_user
```

To copy and edit the included example:

    cp roles/deployment_user/files/deployment_user.yml .
    nano deployment_user.yml


Other Variables
---------------

### defaults/main.yml example

All of our variables are located in `defaults/main.yml` so that they can be overidden via precidence via `group_vars` and/or `host_vars` as is the case for the first three.

```yaml
---
# defaults/main.yml
#
#
# three vars most people will change.
# overidden via group_vars/deployment_user/defaults.yml

deployment_user_username : 'deployer'
deployment_user_uid      : '808'
deployment_user_state    : 'absent'

deployment_user_system_groups       : [ "{{ deployment_user_username }}" ]

# perhaps implement later if required
deployment_user_system_groups_state : '{{ deployment_user_state }}'

deployment_user_home_gid            : '{{ deployment_user_uid }}'
deployment_user_home_group          : '{{ deployment_user_system_groups[0] }}'

deployment_user_sudo_group          : '{{ deployment_user_username }}'
deployment_user_sudo_group_state    : '{{ deployment_user_state }}'

# !!! DANGER in most cases home directories should only be removed after archiving!!!
deployment_user_home                : '{{ "/home/" + deployment_user_username }}'
deployment_user_home_mode           : '0750'
# deployment_user_home_state        : 'absent' # not implemented

deployment_user_shell           : '/bin/bash'
deployment_user_comment         : 'Ansible deployment user'

deployment_users_public_sshkeys       : [ '{{ lookup("pipe","ssh-add -L | grep ^ssh || cat ~/.ssh/id_rsa.pub || true") }}' ]
deployment_users_public_sshkeys_state : '{{ deployment_user_state }}'

deployment_sudoers_d_files:

  etc_sudoers.d:

    src   : 'etc/sudoers.d/sudoers_group.j2'
    dest  : '/etc/sudoers.d/{{ deployment_user_sudo_group }}'
    owner : root
    group : root
    mode  : 0440
```


Inventorymkdir -p ~/projects/ace-dev Examples
------------------

### Production

CentOS production example

```yaml
[deployment_user]
system-001 ansible_ssh_user=root
system-002 ansible_ssh_user=root

```


Ansible Command
---------------



```shell
ansible-playbook systems.yml -i inventory/dev --ask-become-pass --limit workstation-001
```

    ansible-playbook systems.yml -i inventory/development --ask-become-pass

Connectivity Test
-----------------

Now you should be able to ssh in using your new deployment user with your ssh keypair for authentication.

    ssh deployment_user@host

### Development

CentOS or Ubuntu example using an initial ansible_ssh_user called `vagrant`

```yaml
[deployment_user]
db ansible_ssh_host=127.0.0.1 ansible_ssh_port=2222 ansible_ssh_private_key_file=/home/controller_user/projects/project_name/vms/.vagrant/machines/db/virtualbox/private_key ansible_ssh_user=vagrant
```


Ansible Command
---------------
Once you have entries for all your target hosts in your controllers known_hosts file you are ready to run your `ansible-playbook` command and create your deployment user.

    ansible-playbook systems.yml -i inventory/dev --ask-become-pass --limit workstation-001

### Testing

To test your new deployment users connectivity you could use any of the following three:

    ssh deployer@db -p 2222
    ssh deployer@127.0.0.1 -p 2222
    ssh deployer@localhost -p 2222


Role Templates
--------------

`roles/deployment_user/templates` currently has the following templates:

    Ubuntu/12.04/etc/sudoers.d/sudoers_group.j2
    Ubuntu/14.04/etc/sudoers.d/sudoers_group.j2
    Ubuntu/16.04/etc/sudoers.d/sudoers_group.j2
    CentOS/6/etc/sudoers.d/sudoers_group.j2
    CentOS/7/etc/sudoers.d/sudoers_group.j2

#### Usage as a dependency

You could probably use this role as a dependency, I have not tried this yet.

##### deployment_user/meta.yml

Add something like this to the dependencies section of the roles meta/main.yml file. I usually move dependencies section to the top of the file so that it is readily noticable.

    dependencies:
    
      - { role: deployment_user, deployment_user_username: 'deploy', deployment_user_uid: '879', deployment_user_state : 'present' }
      - { role: deployment_user, deployment_user_username: 'ubuntu', deployment_user_state: 'absent' }
      - { role: deployment_user, deployment_user_username: 'vagrant', deployment_user_state: 'absent' }


Dependencies
------------

Does not depend on any other to roles. See **Requirements** for target OS requirements for the Ansible **user** modules.


Vagrant and SSH
---------------

If you are running Vagrant this may be helpful...

### vagrant ssh-config

From the vagrant vm directory run:

    vagrant ssh-config

#### Output example

    Host db
      HostName 127.0.0.1
      User vagrant
      Port 2222
      UserKnownHostsFile /dev/null
      StrictHostKeyChecking no
      PasswordAuthentication no
      IdentityFile /home/controller_user/projects/project_name/vms/.vagrant/machines/db/virtualbox/private_key
      IdentitiesOnly yes
      LogLevel FATAL

### Build your ssh commands

Using the output from `vagrant ssh-config` as a reference, build your initial ssh connection command(s).

#### Remove any stale keys

If your have created a new VM you may need to remove stale host keys from the controllers ~/.ssh/known_hosts files using `ssh-keygen -f` before connecting:

    ssh-keygen -f "/home/controller_user/.ssh/known_hosts" -R [127.0.0.1]:2222
    ssh-keygen -f "/home/controller_user/.ssh/known_hosts" -R [localhost]:2222
    ssh-keygen -f "/home/controller_user/.ssh/known_hosts" -R [db]:2222

### Confirm Connectivity

    ssh vagrant@localhost -p 2222 -i /home/controller_user/projects/project_name/vms/.vagrant/machines/db/virtualbox/private_key
    
    The authenticity of host '[localhost]:2222 ([127.0.0.1]:2222)' can't be established.
    ECDSA key fingerprint is ...
    Are you sure you want to continue connecting (yes/no)? yes

### Vagrant inventory example

We specify our host, ssh port, private key location and our ansible ssh user when testing using Vagrant.

```yaml
[deployment_user]
db ansible_ssh_host=127.0.0.1 ansible_ssh_port=2222 ansible_ssh_private_key_file=/home/controller_user/projects/project_name/vms/.vagrant/machines/db/virtualbox/private_key ansible_ssh_user=vagrant
```

### Run the deployment_user role

### confirm your new deployment user

Now that the new deployment user has been setup for usage with the controllers public ssh key you can connect via ssh with commands similar to this from now on:

    ssh deploy@127.0.0.1 -p 2222
    ssh deploy@127.0.0.1 -p 2200


License
-------

GNU


Author
------

This role is a minimal interpretation of the Debops ansible-bootstrap role which can be found here > https://github.com/debops/ansible-bootstrap

## Adaptated by

Christopher Steel
Systems Administrator
McGill Centre for Integrative Neuroscience
Montreal Neurological Institute
McGill University
3801 University Street
Montréal, QC, Canada H3A 2B4
Tel. No. +1 514 398-2494
E-mail: christopherDOTsteel@mcgill.ca
[MCIN](http://mcin-cnim.ca/), [theneuro.ca](http://theneuro.ca)