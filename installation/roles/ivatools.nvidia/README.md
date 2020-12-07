Role Name
=========

This role installs docker-ce service. 

Requirements
------------

RHEL(ppc64le & x86_64), Ubuntu/x86_64

Role Variables
--------------

force_install: true|false, forces the re-installation when set to true. Default: false

Dependencies
------------

None

Example Playbook
----------------

Including an example of how to use your role (for instance, with variables passed in as parameters) is always nice for users too:

    - hosts: servers
      roles:
         - { role: ivatools.docker-ce, force_install: false }

License
-------

Apache v2.0

Author Information
------------------

Robin Mulkers - IBM Watson - 2020
