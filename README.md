Ansible Pull
=========

Configure a remote machine to run `ansible-pull` on a schedule. `ansible` will be installed on the managed node in a virtual environment using `pip`.

Requirements
------------

- cron
- logrotate

Role Variables
--------------

At a minimum, you need to define `ansible_pull_repo` where your Ansible playbook repository lives as well as the `ansible_pull_playbook` to run.

| Name              | Default Value       | Description          |
|-------------------|---------------------|----------------------|
| `ansible_pull_home` | `/var/lib/ansible` | Main directory for ansible-pull configuration and data. |
| `ansible_pull_workdir` | `{{ ansible_pull_home }}/local` | Directory where repository is cloned. |
| `ansible_pull_repo` | `https://github.com/samdoran/demo-playbooks.git` | Remote repository to clone when running `ansible-pull`. |
| `ansible_pull_inventory` | `{{ ansible_pull_workdir }}/hosts` | Inventory file to use with `ansible-pull`. |
| `ansible_pull_playbook` | `{{ ansible_pull_workdir }}/hello.yml` | Playbook to run with `ansible-pull`. |
| `ansible_pull_script_path` | `/usr/local/sbin/ansible-pull` | Where to put the script which executes `ansible-pull` with the configured arguments. |
| `ansible_pull_logfile` | `/var/log/ansible-pull.log` | Where to log output from `ansible-pull`. Also gets rotated. |
| `ansible_pull_vault_password_file` | `{{ ansible_pull_home }}/vault` | File to hold Ansible vault key. **Not recommonded unless you aware of the implications of storing keys in clear text on remote hosts, or you are using a script to get the secret from an external source.** |
| `ansible_pull_vault_password` | `SuperSecretKey` | Vault key, in plain text, that will be inserted into `{{ ansible_pull_vault_password_file }}`. **Not recommonded unless you aware of the implications of storing keys in clear text on remote hosts, or you are using a script to get the secret from an external source.** |
| `ansible_pull_ssh_private_key` | [see `defaults/main.yml`] | Optionally define an SSH private key that will be installed for `{{ ansible_pull_user }}` on the remote host. If this is not defined, a new key will be generated and the public SSH key will be output at the end of the play. |
| `ansible_known_hosts` | `[]` | List of SSH host keys to add to `known_hosts` for `{{ ansible_pull_user }}`. |
| `ansible_pull_cron_job` | [see `defaults/main.yml`] | Configuration for a job that runs `ansible-pull`. The default settings run `ansible-pull` every 30 minutes. |
| `ansible_pull_user` | `ansible` | User that will run `ansible-pull`. |
| `ansible_pull_group` | `ansible` | Group for `{{ ansible_pull_user }}`. |
| `ansible_pull_scheduler_type` | `cron` | The scheduler type to use, can be either `cron` or `systemd`. |
| `ansible_pull_pip_packages` | `['ansible']` | List of Python packages to install in the virtual environment. |
| `ansible_pull_only_if_changed` | `true` | Whether to execute the playbook only if the repository changes. Note that while this saves computing power most of the time, if the play fails, it will not rerun until the repository changes again. |
| `ansible_pull_user_sudoer` | `true` | Whether to give `{{ ansible_pull_users }}` the right to run any command as any user. |


Dependencies
------------

- samdoran.repo_epel

Example Playbook
----------------

Here is a playbook using an internal GitLab server with the `pull.yml` playbook. We also set the SSH key of the internal GitLab server to avoid any problems.

    - name: Setup Ansible Pull
      hosts: all
      become: True

      vars:
        ansible_pull_playbook: "{{ ansible_pull_workdir }}/playbooks/pull.yml"
        ansible_pull_repo: "git@gitlab.acme.com/internal.git"
        ansible_pull_known_hosts:
          - name: "gitlab.acme.com"
            state: present
            key: "gitlab.acme.com ssh-rsa AAAAB3NzaC1yc2EAAAADAQABAAABAQCEPm0nPQBk+W4FBWSuI2wP0vO2W5cfDQV3B65WayiQPCh5kQIaTfDaRXIHACu9GcZRx5mhTsXYt+jY2egvLwazX5xvvQqDZX7wLw+qJXnpb1pqS7koINnAopGspp5v/+KPk7e3SRbLdNDk8O/g7uXb1PwaryebQM2+eluDebh1zbDd2QgKHf1/p4gZ66m4QJ9s17+Qzj3AJO+5fNr9z0MxPkYkf3jLvJ8PmAqGT+6AYlAh889yCrrC+yGj7VH/H6P3dEakj2xEx3Ib4g42EjKOpumoCVLY6dKrtSlkyOVBEOkf7G3liIV2ZNm6smWsJsnCTMPy4o9ioxF+x5GG1nsL"

      roles:
        - samdoran.repo_epel
        - samdoran.ansible_pull

License
-------

Apache 2.0
