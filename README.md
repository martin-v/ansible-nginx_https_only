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

The role installs a nginx and some small packages from dependent role (e.g. openssl and git)

This role need for a appropriate usage an additional webserver configuration
with specification of the served content, some examples are listed below.


Role Variables
--------------

### Required Variables:

#### nginx_https_server_name

The domain for the ssl enabled site.

    nginx_https_server_name: example.com


#### dehydrated_contactemail

Address for the letsencrypt account. Mostly for certificate expiration notices,
but should be not happen if the cron job works fine.

    dehydrated_contactemail: certmaster@example.com


#### dehydrated_letsencrypt_agreed_terms

To accept the letsencrypt terms of service set the variable
`dehydrated_letsencrypt_agreed_terms` to the current license url.
You find the actual url at https://acme-v01.api.letsencrypt.org/terms.

    dehydrated_letsencrypt_agreed_terms: https://letsencrypt.org/documents/LE-SA-v1.1.1-August-1-2016.pdf


### Optional Variables:

#### nginx_https_ssl_protocols

This variable set the allowed ssl protokol versions. Default is for better compatibility `TLSv1 TLSv1.1 TLSv1.2`.
For more secure setups use:

    nginx_https_ssl_protocols: TLSv1.2


#### nginx_https_ssl_ciphers

Configure the ssl cipher suite, the default is the recomented cipher suite from bettercrypto.org, but with only
Perfect Forward Secrecy (PFS). For better compatibility use one of the first suites, for more secure setups the last
one.

bettercrypto.org

    nginx_https_ssl_ciphers: EDH+CAMELLIA:EDH+aRSA:EECDH+aRSA+AESGCM:EECDH+aRSA+SHA256:EECDH:+CAMELLIA128:+AES128:+SSLv3:!aNULL:!eNULL:!LOW:!3DES:!MD5:!EXP:!PSK:!DSS:!RC4:!SEED:!IDEA:!ECDSA:kEDH:CAMELLIA128-SHA:AES128-SHA

bettercrypto.org (with 256bit non PFS ciphers)

    nginx_https_ssl_ciphers: EDH+CAMELLIA:EDH+aRSA:EECDH+aRSA+AESGCM:EECDH+aRSA+SHA256:EECDH:+CAMELLIA128:+AES128:+SSLv3:!aNULL:!eNULL:!LOW:!3DES:!MD5:!EXP:!PSK:!DSS:!RC4:!SEED:!IDEA:!ECDSA:kEDH:CAMELLIA256-SHA:AES256-SHA

bettercrypto.org (only PFS ciphers, the default)

    nginx_https_ssl_ciphers: EDH+CAMELLIA:EDH+aRSA:EECDH+aRSA+AESGCM:EECDH+aRSA+SHA256:EECDH:+CAMELLIA128:+AES128:+SSLv3:!aNULL:!eNULL:!LOW:!3DES:!MD5:!EXP:!PSK:!DSS:!RC4:!SEED:!IDEA:!ECDSA

only TLSv1.2 AES256 ciphers

    nginx_https_ssl_ciphers: EDH+aRSA+AES256:EECDH+aRSA+AES256:!SSLv3


#### nginx_https_strict_transport

Per default this role configure a https strict transport header with a three month duration. To change the duration set the following variable:

    nginx_https_strict_transport : max-age=2592000 # 30 days

It is also posible the set additional attributes, like:

    # one year, enforce for subdomains and allow HSTS preload
    nginx_https_strict_transport : "max-age=31536000; includeSubDomains; preload"

#### More variables

There are also some unusual variables for super user who need more control,
for details take look at `defaults/main.yml`


Dependencies
------------

This role is designed to work with role [martin-v.dehydrated](https://github.com/martin-v/ansible-dehydrated).


Example Playbook
----------------

    - hosts: all
      remote_user: root
      vars_files:
        - nginx_https_only.yml
      roles:
        - martin-v.nginx_https_only
        - martin-v.dehydrated

    - hosts: all
      remote_user: root
      tasks:
      - name: Deploy nginx sslsite config
        template:
          src: templates/sslsite.conf.j2
          dest: /etc/nginx/sites-available/sslsite.conf
      - name: Enable nginx sslsite config
        file:
          src: /etc/nginx/sites-available/sslsite.conf
          dest: /etc/nginx/sites-enabled/sslsite.conf
          state: link


Example variables file
----------------------

    dehydrated_contactemail: certmaster@example.com

    dehydrated_letsencrypt_agreed_terms: https://letsencrypt.org/documents/LE-SA-v1.1.1-August-1-2016.pdf

    nginx_https_server_name: example.com


Example sslsite.conf templates
------------------------------

Simple serve static contents from `/var/www/html`:

    server {
            listen 443 ssl default_server;
            listen [::]:443 ssl default_server;

            server_name {{ nginx_https_server_name }};

            ssl_certificate /etc/nginx/ssl/{{ nginx_https_server_name }}/fullchain.pem;
            ssl_certificate_key /etc/nginx/ssl/{{ nginx_https_server_name }}/privkey.pem;
            ssl_trusted_certificate /etc/nginx/ssl/{{ nginx_https_server_name }}/fullchain.pem;

            root /var/www/html;

            location / {
                    try_files $uri $uri/ =404;
            }
    }

Work as https proxy for an application on `localhost:8080`:

    server {
        listen 443 ssl default_server;
        listen [::]:443 ssl default_server;

        server_name {{ nginx_https_server_name }};

        ssl_certificate /etc/nginx/ssl/{{ nginx_https_server_name }}/fullchain.pem;
        ssl_certificate_key /etc/nginx/ssl/{{ nginx_https_server_name }}/privkey.pem;
        ssl_trusted_certificate /etc/nginx/ssl/{{ nginx_https_server_name }}/fullchain.pem;

        location / {
            proxy_set_header        Host $host;
            proxy_set_header        X-Real-IP $remote_addr;
            proxy_set_header        X-Forwarded-For $proxy_add_x_forwarded_for;
            proxy_set_header        X-Forwarded-Proto $scheme;

            proxy_pass          http://localhost:8080;
            proxy_redirect      http://localhost:8080 {{ nginx_https_server_name }};
            proxy_read_timeout  90;
        }

        # Optionally, require HTTP basic auth.
        # auth_basic "Please authenticate to use {{ nginx_https_server_name }}";
        # auth_basic_user_file /etc/nginx/passwdfile;
    }


Open tasks
----------

* Implement safe usage of ssl_session_ticket_key
** Refresh key every day
* Add more http security header


License
-------

MIT

Author Information
------------------

This role was created in 2016 and improved in 2017 by [Martin V.](https://github.com/martin-v).
