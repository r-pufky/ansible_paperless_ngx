# Paperless-NGX
Paperless-NGX installation from public release tarball.

## [Requirements][i]
Requires [r_pufky.media][g] galaxy-ng collection. See
[reference documentation][h] for troubleshooting and config variables.

Install size: ~2GB

Database backend changes are **not** supported.

## Role Variables
Detailed variable use documented in defaults. See usage for role operation.

* [defaults][j] - User configurable options.

* [ports][k] - Ports are **not** managed (defined for external use).

## Usage

### Feature Flags
Tasks are gated by feature flags and executed in the following order.

  Step | Flag                          | Notes
 ------|-------------------------------|-------
  1    | paperless_ngx_flg_backup      | Backup existing config. Exits role.
  2    | paperless_ngx_flg_restore     | Restore config, SQLite DB. Exits role.
  3    | paperless_ngx_flg_maintenance | Preform role maintenance tasks.
  4    | paperless_ngx_flg_install     | Install required packages, users, etc.
  5    | paperless_ngx_flg_config      | Install user-defined config.
  6    | paperless_ngx_flg_migrate     | Apply database migrations.

### Example Playbooks

#### New Deployments
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
    paperless_ngx_cfg_backup_d: 'host_vars/pgnx/data'
```

See [Existing Deployments](#existing-deployments) for reproducible
deployments and configuring a deployment with a different database backend.

#### Existing Deployments
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
Configure [environment][a].

``` bash
# Run all tests.
molecule test --all
```

Testing variables:

  Variable            | Type | Description
 ---------------------|------|-------------
  molecule_flg_inject | bool | Disable **get_url** to inject files locally.

### [Releases][b]
Focused on service deployment with templated configuration to minimize role
churn due to inconsistent and rapid rolling release cycle.

 Release | Debian | Ansible | Paperless-NGX | Notes
---------|--------|---------|---------------|-------
 14.x.x  | 13     | 2.20    | v2.20.10      | Update to Ansible 2.20 collection dependencies.
 13.x.x  | 13     | 2.20    | v2.20.8       | Ansible 2.20, feature flags, and semantic versioning.
 12.x.x  | 13     | 2.18    | v2.20.1       | Last 'fully managed' config.
 11.x.x  | 13     | 2.18    | v2.19.3       | Migrate to r_pufky.media.
 10.x.x  | 13     | 2.18    | v2.19.2       | Redis server required for DB migrations.
 9.x.x   | 13     | 2.18    | v2.18.4       | Data annotations V3.
 8.x.x   | 13     | 2.18    | v2.18.4       | Breaking Paperless changes.
 7.x.x   | 13     | 2.18    | v2.18.2       | Migrate to Debian Trixie.
 6.x.x   | 12     | 2.18    | v2.18.1       | Breaking Paperless changes.
 5.x.x   | 12     | 2.18    | v2.17.3       | Data annotations V2.
 4.x.x   | 12     | 2.18    | v2.16.1       | Data annotations.
 3.x.x   | 12     | 2.18    | v2.16.2       | Use source release for JBIG2enc.
 2.x.x   | 12     | 2.18    | v2.14.7       | Ansible 2.18 support.
 1.x.x   | 12     | 2.11    | v1.15.1       | Migration from private repository.

## Issues
Create a bug and provide as much information as possible.

Associate pull requests with a submitted bug.

## License
[AGPL-3.0 License][c] | [direct link][f]

## Author Information
PGP: [466EEC2B67516C7117C85CE3A0BC35D16698BAB9][d] | [github gist][e]


[a]: https://r-pufky.github.io/ansible_docs
[b]: https://semver.org/spec/v2.0.0
[c]: https://www.tldrlegal.com/license/gnu-affero-general-public-license-v3-agpl-3-0
[d]: https://keys.openpgp.org/vks/v1/by-fingerprint/466EEC2B67516C7117C85CE3A0BC35D16698BAB9
[e]: https://gist.github.com/r-pufky/a8df36977c55b5bb20829267c4c49d22

[f]: https://github.com/r-pufky/ansible_paperless_ngx/blob/main/LICENSE
[g]: https://github.com/r-pufky/ansible_collection_media
[h]: https://r-pufky.github.io/docs/media/paperless
[i]: https://github.com/r-pufky/ansible_paperless_ngx/blob/main/meta/main.yml
[j]: https://github.com/r-pufky/ansible_paperless_ngx/tree/main/defaults/main/main.yml
[k]: https://github.com/r-pufky/ansible_paperless_ngx/blob/main/defaults/main/ports.yml
