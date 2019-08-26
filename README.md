Ansible Pull
=========
[![Build Status](https://travis-ci.org/samdoran/ansible-role-ansible-pull.svg?branch=master)](https://travis-ci.org/samdoran/ansible-role-ansible-pull)
[![Galaxy](https://img.shields.io/badge/galaxy-samdoran.ansible_pull-blue.svg?style=flat)](https://galaxy.ansible.com/samdoran/ansible_pull)

Use `ansible` in push mode to configure a remote machine to run `ansible-pull` as `root` on a schedule.

Requirements
------------

- cron
- logrotate

Role Variables
--------------

At a minimum, you need to define `ansible_pull_repo` where your Ansible playbook repostiory lives as well as the `ansible_pull_playbook` to run.

| Name              | Default Value       | Description          |
|-------------------|---------------------|----------------------|
| `ansible_pull_workdir` | `/var/lib/ansible/local` | Directory where repository is cloned. |
| `ansible_pull_repo` | `https://github.com/samdoran/` | Remote repository to clone when running `ansible-pull`. |
| `ansible_pull_playbook` | `{{ ansible_pull_workdir }}/site.yml` | Playbook to run with `ansible-pull`. |
| `ansible_pull_logfile` | `/var/log/ansible-pull.log` | Where to log output from `ansible-pull`. Also gets rotated. |
| `ansible_pull_vault_password_file` | `/root/.vault` | File to hold Ansible vault key. **Not recommonded unless you aware of the implications of storing keys in clear text on remote hosts.** |
| `ansible_pull_vault_password` | `SuperSecretKey` | Vault key, in plain text, that will be inserted int `ansible_pull_vault_password_file`. **Not recommonded unless you aware of the implications of storing keys in clear text on remote hosts.** |
| `ansible_pull_ssh_private_key` | [see `defaults/main.yml`] | Optionally define an SSH private key that will be installed for `root` on the remote host. If this is not defined, a new key will be generated and the pubil SSH key will be output at the end of the play. |
| `ansible_known_hosts` | `[]` | List of SSH host keys to add to `/root/.ssh/known_hosts`. |
| `ansible_pull_cron` | [see `defaults/main.yml`] | Cron configuration for job that runs `ansible-pull`. The default settings run `ansible-pull` every ten minutes. |


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
