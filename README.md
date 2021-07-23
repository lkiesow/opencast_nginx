Ansible: Opencast Nginx Role
============================

This Ansible role installs and prepares Nginx as reverse proxy for Opencast.
If no certificate is present, this role deploys a dummy certificate which allows Nginx to start up.

Role Variables
--------------

- `opencast_storage_downloads_path`
    - Path to Opencast's downloads directory (default: `/srv/opencast/downloads/`)
- `opencast_cors_urls`
    - List of URLs to add CORS exceptions for (default: `[]`)


Additional Configuration
------------------------

While this deploys dummy TSL certificates which allow Nginx to start up,
make sure to deploy proper certificates for production.
To do that, copy your certificates to:

- `/etc/nginx/ssl/{{ inventory_hostname }}.key`
- `/etc/nginx/ssl/{{ inventory_hostname }}.crt`

If you want to use Let's Encrypt to generate certificates, you can also include the role
[`elan.opencast_certbot`](https://galaxy.ansible.com/elan/opencast_certbot)
which will automatically generate TLS certificates for you.


You can also add some custom configuration in the file `/etc/nginx/conf.d/extra.conf`.
The file is included after Opencast's main `location` block.
The role will not modify this file if it exists.


Example Playbook
----------------

Example of how to configure and use the role:

```yaml
- hosts: servers
  become: true
  roles:
    - role: elan.opencast_nginx
```

This will leave you with an invalid dummy certificate.
You will need to replace it with a valid one before booting up Opencast.
The role will _not_ replace an existing certificate so you can safely use a `file` task to deploy it afterwards:

```yaml
- hosts: servers
  become: true
  tasks:
    - include_role:
        name: elan.opencast_nginx

    - name: install tls certificate
      copy:
        src: tls-{{ item }}.pem
        dest: /etc/nginx/ssl/{{ inventory_hostname }}.{{ item }}
        owner: root
        group: root
        mode: '0400'
      notify: reload nginx
      loop:
        - key
        - crt
```
