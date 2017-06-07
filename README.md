nginx https only
================

Install nginx webserver with following configurations:
* http to https redirect
* strong crypto settings
* automatic SSL certificates from let's encrypt.


**The script [dehydrated](https://github.com/lukas2511/dehydrated)
used by this role. This script is working with your private keys so be careful
and review the code. For more details take a look at the
[ansible dehydrated role](https://github.com/martin-v/ansible-dehydrated).**


Requirements
------------

Installs a nginx and some small packages from dependent role (e.g. openssl and git)

This role need for a appropriate usage an additional webserver configuration
with specification of the served content.

TODO add example


Role Variables
--------------

### Required Variables:

TODO write documentation

### Optional Variables:

TODO write documentation

There are also some more advanced variables for super user who need more control,
for details look at `defaults/main.yml`


Dependencies
------------

This role depends on [martin-v.dehydrated](https://github.com/martin-v/ansible-dehydrated).


Example Playbook
----------------

    - hosts: servers
      roles:
        - { role: martin-v.letsencryptsh }
        - { role: martin-v.nginx_https_only }
      vars_files:
        - group_vars/letsencryptsh.yml
        - group_vars/nginx_https_only.yml


Example Variables file
----------------------

TODO


Open tasks
----------

* Implement safe usage of ssl_session_ticket_key
** Refresh key every day
* Add more http security header
* Write documentation in README.md
* Example documentation for usage
* Extend test to deploy a https sites


License
-------

MIT

Author Information
------------------

This role was created in 2016 by [Martin V](https://github.com/martin-v).
