Prerequisites
================

Edit `roles/requirements.yml` to adjust the GitLab URL.

    - name: ansible-role-unbound
      version: master
      src: git+https://gitlab.example.com/test-user/ansible-role-unbound.git

And install the role:

    ansible-galaxy role install -r roles/requirements.yml -p roles
    

Run Playbooks
================

Option 1
----------

    ansible-playbook 01_site.yml -e target_domain=proj1.example.com

- Using the following files as zone data.
  - zones/proj1.example.com.conf.j2
- Using 'local-data' directive for all servers.

Option 2
----------

    ansible-playbook 02_site.yml -e target_domain=proj2.example.com

- Using the following files as zone data.
  - zones/proj2.example.com.zone.j2
  - zones/proj2.example.com.conf.j2
- Using 'auth-zone' directive for all servers.

Option 3 (Not working)
----------

    ansible-playbook 03_site.yml -e target_domain=proj3.example.com

- Using the following files as zone data.
  - zones/proj2.example.com.zone.j2
  - zones/proj2.example.com.conf.j2
  - zones/proj2.example.com.slave.conf.j2
- Using zone transfer (AXFR) from master to slave.
  - Unbound doesn't implement it as a sender-side, only as a receiver-side.
    - https://github.com/NLnetLabs/unbound/blob/release-1.7.3/daemon/worker.c#L1198
