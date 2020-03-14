<!-- markdown-toc start - Don't edit this section. Run M-x markdown-toc-refresh-toc -->
**Table of Contents**

- [Inventory](#inventory)
- [Playbooks and Roles](#playbooks-and-roles)
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
- design: bastion, not localhost
  - awx can't do sudo on tower host
  - set all and delegate to exploit --limit
- todo: don't start my-unbound always
- todo: instead, test start and commit for fast startup
- todo: exclusive control for git repo and container
- todo: not git private key, do it inprior by yourself
