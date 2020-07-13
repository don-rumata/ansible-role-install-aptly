# Ansible Role: Install Aptly

[![License][license-image]][license-url] [![Ansible Galaxy][ansible-galaxy-image]][ansible-galaxy-url]

Install [Aptly](https://www.aptly.info/) on Debian or Ubuntu.

## Work on

```yaml
  platforms:
    - name: Ubuntu
      versions:
        - xenial
        - bionic
        - focal
    - name: Debian
      version:
        - oldstable
        - stable
        - testing
```

## Requirements

[min_ansible_version: 2.8]

## Role Variables

```yaml
#--- Main section ---#
# squeeze or nightly: https://www.aptly.info/download/
aptly_repo_version: stable
# aptly_repo_version: unstable

aptly_path_to_local_repo: /var/aptly

aptly_user: aptly

aptly_group: aptly

#--- API section ---#
aptly_run_api_service: true
# aptly_run_api_service: false

# https://www.aptly.info/doc/api/
aptly_api_port: 8080

# /lib/systemd/system/aptly-api.service
aptly_api_service_name: aptly-api

#--- First repo serction ---#
aptly_my_first_repo_create: true
# aptly_create_my_first_repo: false

# https://www.aptly.info/doc/api/repos/ or https://www.aptly.info/doc/aptly/repo/create/
aptly_my_first_repo_create_over: api
# aptly_my_first_repo_create_over: cli

# aptly repo create <name>
aptly_my_first_repo_name: my-first-repo

# -distribution="" from https://www.aptly.info/doc/aptly/repo/create/
aptly_my_first_repo_distribution: rolling

aptly_my_first_repo_comment: Repo generated with https://github.com/don-rumata/ansible-role-install-aptly

# https://wiki.debian.org/ru/SourcesList
aptly_my_first_repo_component: main
# aptly_my_first_repo_component: contrib
# aptly_my_first_repo_component: non-free

#--- Software in created repo section ---#

# Viber. Why? Because permanent direct download link.
# aptly_add_first_software_in_created_repo: true
aptly_add_first_software_in_created_repo: false

#--- GPG section ---#
aptly_gpg_key_generate: true
# aptly_gpg_key_generate: false

aptly_gpg_key_path: '{{ aptly_path_to_local_repo }}/gpg'

aptly_gpg_publickey_filename: repo.key

aptly_gpg_key_maintainer: Jon Doe

aptly_gpg_key_email: aptly@example.com

aptly_gpg_key_expire_date: 365

# TODO. aptly_create_gpg_pass: true\false
aptly_gpg_key_pass: qazwsxedc

aptly_gpg_key_comment: with stupid passphrase

aptly_gpg_key_type: RSA

aptly_gpg_key_length: 4096

#!!! ALL accesses in READ ONLY 4 all !!!#

#--- www section ---#
aptly_www_access: true
# aptly_www_access: false

aptly_www_webdav_access: true
# aptly_www_webdav_access: false

# http://localhost/deb <---
aptly_www_module_name: deb

aptly_www_backend: nginx
# aptly_www_backend: lighttpd
# aptly_www_backend: apache

aptly_www_local_path: /var/www/{{ aptly_www_module_name }}

#--- rsync section ---#
aptly_rsync_access: true
# aptly_rsync_access: false

# rsync://localhost/deb <---
aptly_rsync_module_name: deb

aptly_rsync_local_path: '{{ aptly_path_to_local_repo }}/public'

#--- ftp section ---#
aptly_ftp_access: true
# aptly_ftp_access: false

# ftp://localhost/deb <---
aptly_ftp_module_name: deb

# "anon_root" in vsftpd.conf
aptly_ftp_anon_root_dir: /srv/ftp

aptly_ftp_local_path: '{{ aptly_ftp_anon_root_dir }}/{{ aptly_ftp_module_name }}'

#--- nfs section ---#
aptly_nfs_access: true
# aptly_nfs_access: false
```

## Dependencies

None.

## Example Playbook

Install stable version `aptly` with:

- Create service user for aptly
- Install and config aptly api daemon
- Create "Hello, World!" repository
- Generate gpg keys for sign repository (**WARNING!!!** WITHOUT PASSPHRASE!!!)
- Share repository over (all accesses in **read only**):
  - http (nginx)
  - webdav (nginx)
  - rsync (rsyncd)
  - ftp (vsftpd)
  - nfs (nfs-kernel-server)

`install-aptly.yml`:

```yaml
- name: Install Aptly
  hosts: all
  strategy: free
  serial:
    - "100%"
  roles:
    - ansible-role-install-aptly
  tasks:
```

Install **un**stable version `aptly` without all features:

```yaml
- name: Install Aptly
  hosts: all
  strategy: free
  serial:
    - "100%"
  roles:
    - ansible-role-install-aptly
  vars:
    aptly_repo_version: unstable
    aptly_run_api_service: false
    aptly_create_my_first_repo: false
    aptly_add_first_software_in_created_repo: false
    aptly_gpg_key_generate: false
    aptly_www_access: false
    aptly_www_webdav_access: false
    aptly_rsync_access: false
    aptly_ftp_access: false
    aptly_nfs_access: false
  tasks:
```

Install stable version `aptly`, with create repo over cli and share **empty** repo over apache:

```yaml
- name: Install Aptly
  hosts: all
  strategy: free
  serial:
    - "100%"
  roles:
    - ansible-role-install-aptly
  vars:
    aptly_run_api_service: false
    aptly_create_my_first_repo: true
    aptly_my_first_repo_create_over: cli
    aptly_add_first_software_in_created_repo: false
    aptly_gpg_key_generate: false
    aptly_www_access: true
    aptly_www_backend: apache
    aptly_www_webdav_access: false
    aptly_rsync_access: false
    aptly_ftp_access: false
    aptly_nfs_access: false
  tasks:
```

## Add your repo in your Debian\Ubuntu

`10.10.10.10` - example ip.

### Over http

```bash
echo "deb http://10.10.10.10/deb rolling main" | sudo tee --append /etc/apt/sources.list.d/my-awesome-repo.list
```

```bash
wget -q -O - http://10.10.10.10/deb/repo.key | sudo apt-key add -
```

### Over ftp

```bash
echo "deb ftp://10.10.10.10/deb rolling main" | sudo tee --append /etc/apt/sources.list.d/my-awesome-repo.list
```

```bash
wget -q -O - ftp://10.10.10.10/deb/repo.key | sudo apt-key add -
```

### Over NFS

```bash
sudo mkdir /var/repo
```

```bash
mount.nfs 10.10.10.10:/var/aptly/public /var/repo
```

```bash
echo "deb file:/var/repo rolling main" | sudo tee --append /etc/apt/sources.list.d/my-awesome-repo.list
```

```bash
sudo apt-key add /var/repo/repo.key
```

For permanent mount:

```bash
echo "10.10.10.10:/var/aptly/public /var/repo nfs noatime,nodiratime 0 0" | sudo tee --append /etc/fstab
```

### Over webdav

*Comming soon*.

### After all

```bash
sudo apt update
```

## License

Apache License, Version 2.0

## Author Information

[don Rumata](https://github.com/don-rumata)

## TODO

- Add tests.
- Add `aptly_create_gpg_pass: true\false`.
- Add example for webdav.

[license-image]: https://img.shields.io/github/license/don-rumata/ansible-role-install-aptly.svg
[license-url]: https://opensource.org/licenses/Apache-2.0

[ansible-galaxy-image]: https://img.shields.io/badge/ansible_galaxy-don__rumata.ansible__role__install__aptly-blue.svg
[ansible-galaxy-url]: https://galaxy.ansible.com/don_rumata/ansible_role_install_aptly
