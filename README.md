[![CI](https://github.com/de-it-krachten/ansible-role-snappymail/workflows/CI/badge.svg?event=push)](https://github.com/de-it-krachten/ansible-role-snappymail/actions?query=workflow%3ACI)


# ansible-role-snappymail

Manages snappymail



## Dependencies

#### Roles
None

#### Collections
- community.general
- ansible.posix
- community.general

## Platforms

Supported platforms

- Red Hat Enterprise Linux 7<sup>1</sup>
- Red Hat Enterprise Linux 8<sup>1</sup>
- Red Hat Enterprise Linux 9<sup>1</sup>
- CentOS 7
- RockyLinux 8
- RockyLinux 9
- OracleLinux 8
- AlmaLinux 8
- AlmaLinux 9
- Debian 10 (Buster)
- Debian 11 (Bullseye)
- Ubuntu 18.04 LTS
- Ubuntu 20.04 LTS
- Ubuntu 22.04 LTS

Note:
<sup>1</sup> : no automated testing is performed on these platforms

## Role Variables
### defaults/main.yml
<pre><code>
# Required variables (no default):
# ----------------------------------------------------------
# Snappymail location (where webserver will search)
# snappymail_path: /var/www/webmail.example.com/public_html

# Snappymail logfile location
# snappymail_log: /var/www/webmail.example.com/log



# Optional variables (with default):
# ----------------------------------------------------------

# Location where snappymail will write data
# snappymail_data_path: "{{ snappymail_path }}/data"
snappymail_data_path: "/var/lib/snappymail"

# snappymail version
snappymail_version: latest

# Github API endpoint to get latest version
snappymail_api: https://api.github.com/repos/the-djmaze/snappymail

# Github release archive
snappymail_url: >-
  https://github.com/the-djmaze/snappymail/releases/download/v{{ snappymail_version }}/snappymail-{{ snappymail_version }}.tar.gz

# directory to put temporary download file into
snappymail_tmpdir: /tmp

# local file
snappymail_file: "{{ snappymail_tmpdir }}/{{ snappymail_url | basename }}"

# OS user/group
snappymail_user: "{{ 'apache' if snappymail_web_server == 'apache' else 'www-data' }}"
snappymail_group: "{{ 'apache' if snappymail_web_server == 'apache' else 'www-data' }}"

# Restrict access to local loopback
snappymail_restrict_access: false

# dict of snappymail settings
snappymail_settings: {}

# Default domain template
snappymail_domain_template: domain.j2
</pre></code>

### defaults/family-RedHat.yml
<pre><code>
# list of packages
snappymail_packages:
  - unzip
  - rsync

# default web service
snappymail_web_server: apache

# web service
snappymail_web_service: "{{ 'httpd' if snappymail_web_server == 'apache' else 'nginx' }}"

# php socket
snappymail_php_socket: /var/run/php/php-fpm.sock
</pre></code>

### defaults/family-Debian.yml
<pre><code>
# list of packages
snappymail_packages:
  - unzip
  - rsync

# default web service
snappymail_web_server: nginx

# web service
snappymail_web_service: "{{ 'apache2' if snappymail_web_server == 'apache' else 'nginx' }}"

# php socket
snappymail_php_socket: /var/run/php/php-fpm.sock
</pre></code>




## Example Playbook
### molecule/default/converge.yml
<pre><code>
- name: sample playbook for role 'snappymail'
  hosts: all
  become: "yes"
  vars:
    openssl_fqdn: server.example.com
    apache_fqdn: "{{ openssl_fqdn }}"
    apache_ssl_key: "{{ openssl_server_key }}"
    apache_ssl_crt: "{{ openssl_server_crt }}"
    apache_ssl_chain: "{{ openssl_server_crt }}"
    nginx_server_name: webmail.example.com
    nginx_ssl_key: "{{ openssl_server_key }}"
    nginx_ssl_crt: "{{ openssl_server_crt }}"
    nginx_root: /var/www/webmail.example.com/public_html
    nginx_logdir: /var/www/webmail.example.com/log
    snappymail_vhost: webmail.example.com
    snappymail_domain: example.com
    snappymail_ssl_key: "{{ openssl_server_key }}"
    snappymail_ssl_crt: "{{ openssl_server_crt }}"
    snappymail_ssl_chain: "{{ openssl_server_crt }}"
    snappymail_path: /var/www/webmail.example.com/public_html
    snappymail_log: /var/www/webmail.example.com/log
    snappymail_web_server: "{{ 'apache' if ansible_os_family == 'RedHat' else 'nginx' }}"
  pre_tasks:
    - name: Create 'remote_tmp'
      ansible.builtin.file:
        path: /root/.ansible/tmp
        state: directory
        mode: "0700"
  roles:
    - hosts
    - openssl
    - {'role': 'apache', 'when': "ansible_os_family == 'RedHat'"}
    - {'role': 'nginx', 'when': "ansible_os_family == 'Debian'"}
    - php
  tasks:
    - name: Include role 'snappymail'
      ansible.builtin.include_role:
        name: snappymail
</pre></code>
