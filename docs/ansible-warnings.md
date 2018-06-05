# ansible-warnings.md

```shell
ok: [bionic64] => {"append": true, "changed": false, "comment": "Ansible deployment user", "group": 878, "groups": "deploy", "home": "/home/deploy", "move_home": false, "name": "deploy", "shell": "/bin/bash", "state": "present", "uid": 878}

TASK [../../ : Delete sudoers.d group file when state is set to absent] ********
task path: /home/users/csteel/projects/ace-dev/ace/roles/deployer/tasks/main.yml:147
 [WARNING]: when statements should not include jinja2 templating delimiters
such as {{ }} or {% %}. Found: '{{ deployment_user_sudo_group_state }}' ==
'absent'
```
