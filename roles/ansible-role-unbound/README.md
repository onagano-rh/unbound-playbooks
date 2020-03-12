ansible-role-unbound
=========

Requirements
------------

RHEL 8

Role Variables
--------------

    unbound_listen_address: 127.0.0.1
    unbound_network_cidr_to_allow: 192.168.122.0/24
    unbound_default_nameserver: 192.168.122.1


Example Playbook
----------------

    - name: Provision Unbound servers
      hosts: unbound_servers
      become: true
      roles:
        - role: ansible-role-unbound
          vars:
            unbound_listen_address: 0.0.0.0
            unbound_network_cidr_to_allow: 0.0.0.0/0
            unbound_default_nameserver: 192.168.122.1
