<!-- markdown-toc start - Don't edit this section. Run M-x markdown-toc-refresh-toc -->
**Table of Contents**

- [Inventory](#inventory)
- [Playbooks and Roles](#playbooks-and-roles)
- [Pre-commit using yamllint and ansible-lint](#pre-commit-using-yamllint-and-ansible-lint)
- [Tasks](#tasks)

<!-- markdown-toc end -->


Inventory
================

Install "netaddr" Python package because "ipv4" Ansible filter is used.


Playbooks and Roles
================

Pre-commit using yamllint and ansible-lint
================

To install:

    pip install pre-commit yamllint ansible-lint

Or ("molecule" has them as dependencies):

    pip install molecule

Configuraiton:

    pre-commit sample-config > .pre-commit-config.yaml
    vi .pre-commit-config.yaml

- For yamllint
  - https://yamllint.readthedocs.io/en/stable/integration.html#integration-with-pre-commit
- For ansible-lint
  - https://github.com/ansible/ansible-lint#pre-commit-setup

To run against all files for test:

    pre-commit run --all-files

To run before every Git commit automatically:

    pre-commit install

To disable the automatic execution:

    pre-commit uninstall


Tasks
================

- limit: unbound group hierarchy
  - unbound and bastion are hard coded
  - proj1 etc. are free as long as this limit
- limit: zero or single limit tag
- limit: add and remove are orderd
  - add is tested, remove is not
  - remove deletes all entries, after that you can add
- limit: resolv.conf is not modified
- design: bastion, not localhost
  - awx can't do sudo on tower host
  - set all and delegate to exploit --limit
  - role shouldn't depends on inventory in general
- todo: don't start my-unbound always
- deploy playbook
  - todo: instead, test start and commit for fast startup
  - todo: install bind-utils on tower
  - todo: exclusive control for git repo and container
- todo: not git private key, do it inprior by yourself
- todo: unbound container role
- todo: customize local-zone type (static or nxdomain?)
- todo: customize git commit log
  - include job id link to tower
