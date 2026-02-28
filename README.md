# Paperless-NGX
Paperless-NGX installation from public release tarball.

## [Requirements](https://github.com/r-pufky/ansible_paperless_ngx/blob/main/meta/main.yml)
**galaxy-ng** roles cannot be used independently. Part of
[r_pufky.media](https://github.com/r-pufky/ansible_collection_media)
collection.

Install size: ~2GB

Database backend changes are **not** supported.

## Role Variables
Detailed variable use documented in defaults. See usage for role operation.

* [defaults](https://github.com/r-pufky/ansible_paperless_ngx/tree/main/defaults/main/main.yml) -
  User configurable options.

* [vars](https://github.com/r-pufky/ansible_paperless_ngx/tree/main/vars/main.yml) -
  Role default options (may be referenced in defaults).

* [ports](https://github.com/r-pufky/ansible_paperless_ngx/blob/main/defaults/main/ports.yml) -
  Ports are **not** managed (defined for external use).

## Usage

### Feature Flags
Tasks are gated by feature flags and executed in the following order.

  Step | Flag                          | Notes
 ------|-------------------------------|-------
  1    | paperless_ngx_flg_backup      | Backup existing config. Exits role.
  2    | paperless_ngx_flg_maintenance | Preform role maintenance tasks.
  3    | paperless_ngx_flg_install     | Install required packages, users, etc.
  4    | paperless_ngx_flg_config      | Install user-defined config.
  5    | paperless_ngx_flg_migrate     | Apply database migrations.
  6    | paperless_ngx_flg_start       | Start service and validate install.

## Example Playbooks

### New Deployments
A minimal config will be deployed if no configuration is specified. This will
deploy a working Paperless-NGX install using **/var/opt/paperless** for user
data/SQLite and **/etc/opt/paperless** for configuration, with
**admin**/**admin** as the default administrator.

``` yaml
- name: 'Paperless-NGX'
  ansible.builtin.include_role:
    name: 'r_pufky.media.paperless_ngx'
```

Once configured take a backup of configuration.

``` yaml
- name: 'Backup Paperless-NGX'
  ansible.builtin.include_role:
    name: 'r_pufky.media.paperless_ngx'
  vars:
    paperless_ngx_flg_backup: true
    paperless_ngx_cfg_backup_dir: 'host_vars/pgnx/data'
```

See [Existing Deployments](#existing-deployments) for reproducible
deployments and configuring a deployment with a different database backend.

### Existing Deployments
Manage additional files and directories (data storage locations, certificates,
etc.) outside of role if used **within** the config. Must be accessible to
**paperless_ngx_srv_user** account.

Non-SQLite databases must be configured and accessible before executing.

host_vars/pngx.example.com/paperless.conf
``` ini
# Create a basic customized config.
#
# Have a look at the docs for documentation:
# https://docs.paperless-ngx.com/configuration/
#
# Template:
# https://github.com/paperless-ngx/paperless-ngx/blob/dev/paperless.conf.example

PAPERLESS_CONVERT_TMPDIR=/tmp
PAPERLESS_CONSUMPTION_DIR=/d/consume
PAPERLESS_DATA_DIR=/d/data
PAPERLESS_EMPTY_TRASH_DIR=/d/trash
PAPERLESS_MEDIA_ROOT=/d/media
# Use role built ghostscript which addresses vulnerabilities.
PAPERLESS_GS_BINARY=/usr/local/bin/gs
PAPERLESS_ADMIN_USER=my_custom_admin_user
PAPERLESS_ADMIN_MAIL=admin@example.com
PAPERLESS_ADMIN_PASSWORD={{ vault_pgnx_admin_password }}
```

``` yaml
- name: 'New Paperless Deployment'
  ansible.builtin.include_role:
    name: 'r_pufky.media.paperless_ngx'
  vars:
    paperless_ngx_cfg_file: 'host_vars/pngx.example.com/paperless.conf'
```


## Development
Configure [environment](https://r-pufky.github.io/ansible_collection_docs/ansible/environment)

Run all unit tests:
``` bash
molecule test --all
```

Testing variables:
 Variable          | type | Description
-------------------|------|-------------
 url_inject_enable | bool | Disable **get_url** to inject files locally.

### Releases
[Semantic versioning](https://semver.org/spec/v2.0.0) focused on service
deployment with templated configuration to minimize role churn due to
inconsistent and rapid rolling release cycle.

 Release | Debian | Ansible | Paperless-NGX | Notes
---------|--------|---------|---------------|-------
 13.x.x  | 13     | 2.20    | 2.20.8        | Ansible 2.20, feature flags, and semantic versioning.
 12.x.x  | 13     | 2.18    | 2.20.1        | Last 'fully managed' config.
 11.x.x  | 13     | 2.18    | 2.19.3        | Migrate to r_pufky.media.
 10.x.x  | 13     | 2.18    | 2.19.2        | Redis server required for DB migrations.
 9.x.x   | 13     | 2.18    | 2.18.4        | Data annotations V3.
 8.x.x   | 13     | 2.18    | 2.18.4        | Breaking Paperless changes.
 7.x.x   | 13     | 2.18    | 2.18.2        | Migrate to Debian Trixie.
 6.x.x   | 12     | 2.18    | 2.18.1        | Breaking Paperless changes.
 5.x.x   | 12     | 2.18    | 2.17.3        | Data annotations V2.
 4.x.x   | 12     | 2.18    | 2.16.1        | Data annotations.
 3.x.x   | 12     | 2.18    | 2.16.2        | Use source release for JBIG2enc.
 2.x.x   | 12     | 2.18    | 2.14.7        | Ansible 2.18 support.
 1.x.x   | 12     | 2.11    | 1.15.1        | Migration from private repository.

## Issues
Create a bug and provide as much information as possible.

Associate pull requests with a submitted bug.

## License
[AGPL-3.0 License](https://www.tldrlegal.com/license/gnu-affero-general-public-license-v3-agpl-3-0)
 [(direct link)](https://github.com/r-pufky/ansible_paperless_ngx/blob/main/LICENSE)

## Author Information
PGP Fingerprint: [466EEC2B67516C7117C85CE3A0BC35D16698BAB9](https://keys.openpgp.org/vks/v1/by-fingerprint/466EEC2B67516C7117C85CE3A0BC35D16698BAB9)
| [github gist](https://gist.github.com/r-pufky/a8df36977c55b5bb20829267c4c49d22)
