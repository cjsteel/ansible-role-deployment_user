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
    system_01 ansible_ssh_user=some_preexisting_admin



Role Variables
--------------

### defaults/main.yml example

    ---
    # defaults file for ansible-role-deployment_user
    
    deployment_user                 : 'deploy'
    deployment_user_uid             : '990'
    
    deployment_user_system_groups   : [ "{{ deployment_user }}" ]
    deployment_user_home_gid        : '990'
    deployment_user_home_group     : '{{ deployment_user_system_groups[0] }}'
    deployment_user_sudo_group     : '{{ deployment_user }}'
    
    deployment_user_home            : '{{ "/home/" + deployment_user }}'
    deployment_user_home_mode       : '0750'
    
    deployment_user_shell           : '/bin/bash'
    deployment_user_comment         : 'Ansible deployment user'
    
    deployment_users_public_sshkeys : [ '{{ lookup("pipe","ssh-add -L | grep ^ssh || cat ~/.ssh/id_rsa.pub || true") }}' ]

    deployment_sudoers_d_files:
    
      etc_sudoers.d:
    
        src: 'etc/sudoers.d/{{ deployment_user_sudo_group }}'
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
