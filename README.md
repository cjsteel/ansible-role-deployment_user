ansible-role-deployment_user
============================

An Ansible role to setup a (system) deployment user and (system) deployment group.

Requirements
------------

Items required on the target OS for the Ansible **user** module:

    useradd
    userdel
    usermod

Role Playbook
-------------

You can copy the testing example here:

    cp roles/deployment_user/files/deployment_user.yml .

```yaml
---
- hosts: deployment_user
  user: {{ deployment_user }}
  become: true
  gather_facts: true
  roles:
    - deployment_user
```

* You can set the deployment user var here or in group_vars/deployment_user/defaults.yml as shown in the **Role Variables** section.

Role Variables
--------------

### group_vars/deployment_user/defaults.yml

* Create a directory to hold group_vars

    mkdir -p group_vars/deployment_user

* Copy the roles group_vars example file.

    cp roles/deployment_user/files/group_vars/deployment_user/defaults.yml group_vars/deployment_user/.

* Edit the roles group_vars file to match your setup.

    nano group_vars/deployment_user/defaults

#### Minimal content example

```yaml
    # file: group_vars/deployment_user/deployment_user
    #
    # Used to overide role/deployment_user/defaults/main.yml vars
    # so that our deployment user is not commited to the roles repository.
    
    deployment_user         : 'your_deployment_user'
    deployment_user_uid		: '2001'
    deployment_user_state   : 'present'
```

Ansible playbook
----------------

### Main Playbook

    nano system.yml

#### Content example

```yaml
---
- hosts: all
  become: false
    
- include: deployment_user.yml
```

SSH
---

### vagrant ssh-config

From the vagrant vm directory run:

    vagrant ssh-config

#### Output example

    Host web
      HostName 127.0.0.1
      User vagrant
      Port 2222
      UserKnownHostsFile /dev/null
      StrictHostKeyChecking no
      PasswordAuthentication no
      IdentityFile /home/controller_user/projects/ace_testing/ace_test_vms/.vagrant/machines/web/virtualbox/private_key
      IdentitiesOnly yes
      LogLevel FATAL
    
    Host db
      HostName 127.0.0.1
      User vagrant
      Port 2200
      UserKnownHostsFile /dev/null
      StrictHostKeyChecking no
      PasswordAuthentication no
      IdentityFile /home/controller_user/projects/ace_testing/ace_test_vms/.vagrant/machines/db/virtualbox/private_key
      IdentitiesOnly yes
      LogLevel FATAL

### Build your ssh command

Using the output from `vagrant ssh-config` as a reference, build your initial ssh connection command(s).

    ssh vagrant@localhost -p 2222 -i /home/controller_user/projects/ace_testing/ace_test_vms/.vagrant/machines/web/virtualbox/private_key
    ssh vagrant@localhost -p 2200 -i /home/controller_user/projects/ace_testing/ace_test_vms/.vagrant/machines/db/virtualbox/private_key

    The authenticity of host '[localhost]:2222 ([127.0.0.1]:2222)' can't be established.
    ECDSA key fingerprint is 3a:3e:03:b9:70:71:cd:64:c4:be:f7:59:4f:ba:5c:bb.
    Are you sure you want to continue connecting (yes/no)? yes

If your have created a new VM you may need to remove stale host keys from the controllers ~/.ssh/known_hosts files using `ssh-keygen -f` before connecting:

    ssh-keygen -f "/home/controller_user/.ssh/known_hosts" -R [localhost]:2222
    ssh-keygen -f "/home/controller_user/.ssh/known_hosts" -R [localhost]:2200

Ansible Command
---------------
Once you have entries for all your target hosts in your controllers known_hosts file you are ready to run your `ansible-playbook` command and create your deployment user.

    ansible-playbook testing_parc.yml -i inventory_dev

### Confirm Connectivity

Now that the new deployment user has been setup for usage with the controllers public ssh key you can connect via ssh with commands similar to this from now on:

    ssh deploy@127.0.0.1 -p 2222
    ssh deploy@127.0.0.1 -p 2200

#### Usage as a dependancy

You could probably use this role as a dependancy, I have not tried this yet.

##### deployment_user/meta.yml
Add something like this to the dependencies section of the roles meta/main.yml file. I usually move dependencies section to the top of the file so that it is readily noticable.

    dependencies:

      - { role: deployment_user, deployment_user: 'deploy', deployment_user_uid: '879', deployment_user_state : 'present' }
      - { role: deployment_user, deployment_user: 'ubuntu', deployment_user_state: 'absent' }
      - { role: deployment_user, deployment_user: 'vagrant', deployment_user_state: 'absent' }
 
### defaults/main.yml example

All of the follwoing variables are avaialbe in defaults/main.yml. The first three 

    ---
    # defaults file for ansible-role-deployment_user
    
    deployment_user                 : 'deploy'
    deployment_user_uid             : '1001'
    deployment_user_state           : 'present'
    
    deployment_user_system_groups       : [ "{{ deployment_user }}" ]
    # this probably shoud be skipped for now
    deployment_user_system_groups_state : '{{ deployment_user_state }}'
    
    deployment_user_home_gid       : '{{ deployment_user_uid }}'
    deployment_user_home_group     : '{{ deployment_user_system_groups[0] }}'
    
    deployment_user_sudo_group        : '{{ deployment_user }}'
    deployment_user_sudo_group_state  : '{{ deployment_user_state }}'
    
    # !!! DANGER in most cases home directories should only be removed after archiving!!!
    deployment_user_home            : '{{ "/home/" + deployment_user }}'
    deployment_user_home_mode       : '0750'
    # deployment_user_home_state      : 'absent' # not implemented
    
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
        mode  : 0400

Templates
---------

`roles/deployment_user/templates` currently has the following templates:

    Ubuntu/12.04/etc/sudoers.d/sudoers_group.j2 
    CentOS/6/etc/sudoers.d/sudoers_group.j2
    CentOS/7/etc/sudoers.d/sudoers_group.j2

Dependencies
------------

Does not depend on any other to roles. See **Requirements** for target OS requirements for the Ansible **user** modules.

License
-------

GNU

Author
------

This role is a minimal interpretation of the Debops ansible-bootstrap role which can be found here > https://github.com/debops/ansible-bootstrap
