ansible-role-deployment_user
===========================

An Ansible role to setup a (system) deployment user and (system) deployment group.

Requirements
------------
(on host that executes module)

    useradd
    userdel
    usermod

Inventory Example
-----------------

    [deployment_user]
    system_01 ansible_ssh_user=some_preexisting_admin_user

Role Variables
--------------

### group_vars/deployment_user

You can overiding three default variables via group_vars to get rolling quickly as follows:

    mkdir -p group_vars/deployment_user
    nano -p group_vars/deployment_user/default.yml

#### Minimal content example

    ---
    deployment_user        : 'deploy'
    deployment_user_uid    : '879'
    deployment_user_state  : 'present'

#### Usage as a dependancy

You could probably use this role as a dependancy, I have not tried this yet.

##### deployment_user/meta.yml
Add something like this to the dependencies section of the roles meta/main.yml file. I usually move dependencies section to the top of the file so that it is readily noticable.


    dependencies:

      - { role: deployment_user, deployment_user: 'deploy', deployment_user_uid: '879', deployment_user_state : 'present' }
      - { role: deployment_user, deployment_user: 'ubuntu', deployment_user_state: 'absent' }
      - { role: deployment_user, deployment_user: 'vagrant', deployment_user_state: 'absent' }

 
### defaults/main.yml example

If you want to do a lot of customization you can overide all of the items in defaults/main.yml

    ---
    # defaults file for ansible-role-deployment_user
    
    deployment_user                 : 'deploy'
    deployment_user_uid             : '878'
    deployment_user_state           : 'absent'
    
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

### templates/Ubuntu/12.04/etc/sudoers.d/deploy.js

You will want to rename `deploy.js` template to match your sudo group name if you change it in defaults/main.yml.

Dependencies
------------

Does not depend on other roles nor global variables when used with the provided defaults/main.yml file.

Example Playbook
----------------

    ---
    - hosts: deployment_user
      user: some_preexisting_admin
      become: true
      gather_facts: true
      roles:
        - ansible-role-deployment_user

License
-------

GNU

Author
------

This role is a minimal interpretation of the Debops ansible-bootstrap role which can be found here > https://github.com/debops/ansible-bootstrap
