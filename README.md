# Paperless-NGX
Paperless-NGX installation from public release tarball.

## Requirements
[supported platforms](https://github.com/r-pufky/ansible_paperless_ngx/blob/main/meta/main.yml)

## Role Variables
[defaults](https://github.com/r-pufky/ansible_paperless_ngx/tree/main/defaults/main)

### Ports
All ports and protocols have been defined for the role.

[defaults/ports.yml](https://github.com/r-pufky/ansible_paperless_ngx/blob/main/defaults/main/ports.yml)

## Dependencies
**galaxy-ng** roles cannot be used independently. Part of
[r_pufky.srv](https://github.com/r-pufky/ansible_collection_srv) collection.

## Example Playbook
Read defaults documentation.

The following example will get an instance quickly up and running. Paperless
is highly customizable and you should fully read through defaults before using.
``` yaml
- name: 'Paperless-NGX server'
  hosts: 'ngx.example.com'
  become: true
  roles:
     - 'r_pufky.srv.paperless_ngx'
  vars:
    paperless_ngx_cfg_dbengine: 'sqlite'
    paperless_ngx_cfg_admin_user: 'example_user'
    paperless_ngx_cfg_admin_mail: 'user@example.com'
    paperless_ngx_cfg_admin_password: '{{ vault_paperless_ngx_admin_password }}'
    paperless_ngx_cfg_filename_format: !unsafe '{{ document_type }}/{{ correspondent }}/{{ created }}-{{ title }}-[{{ tag_list }}]'
```

Changes updating the configuration only can be done to speed role application:
``` bash
ansible-playbook site.yml --tags Paperless-NGX -e 'paperless_ngx_srv_force_config_only_enable=true'
```

### Reverse Proxy
Reverse proxy configuration has drastically **changed**. Be sure to set the
following configuration variables:

host_vars/paperless.example.com/vars/paperless.yml
``` yaml
paperless_ngx_use_x_forward_host: true
paperless_ngx_use_x_forward_port: true
paperless_ngx_url: 'https://paperless.example.com'
paperless_ngx_csrf_trusted_origins: 'https://paperless.example.com'
paperless_ngx_allowed_hosts: 'paperless.example.com'
paperless_ngx_cors_allowed_hosts: 'https://paperless.example.com'
```
See linked bugs detailing reasons for change:
* https://github.com/paperless-ngx/paperless-ngx/pull/674
* https://github.com/paperless-ngx/paperless-ngx/issues/817
* https://github.com/paperless-ngx/paperless-ngx/issues/712

Receiving a 403 after logging in explicitly after upgrading with:
```
Forbidden (403) CSRF verification failed. Request aborted.
```
See the [Proxy Rule Changes](https://github.com/paperless-ngx/paperless-ngx/wiki/Using-a-Reverse-Proxy-with-Paperless-ngx#nginx)
and be sure to add referrer-policy to allow requests through:

```
add_header Referrer-Policy 'strict-origin-when-cross-origin';
```

Restart both NGINX and Paperless and try again.

## Suggested Archival Use
Suggested Use (based on archivst recommendations):

1. **Document Types** refer to the broad type of document in question. Is it
  a letter? Receipt? Bill? Every instance will be different, but this should
  be your broadest field. You just want to more of less get it in the
  ballpark. For example, my Receipts doctype holds receipts that I scan in,
  but it will also hold confirmations from my debtors that I paid a bill, or
  an email from Cash app that I sold Bitcoin.
2. **Correspondent** refers to the person/organization you are communicating
  with in the document. A bill from your credit card would have Capital One
  as correspondent for example, while a copy of your W2 might go under IRS.
  Again, you can be broad here, as trying to narrow it down is going to
  drive you crazy.
3. **Tags** are used to answer the below basic concepts:
  * **Who** is it referring to? In my case, I have tags for myself, my wife,
    the kids, and the dogs. They are all the same color to easily denote
    that. Note that this is NOT the same as correspondent.
  * **What** is it referring to? Is it related to your car loan? Is it
    related to your homes maintenance? Mark these tags in a different color
    to easily notice them.
  * **When** is the information in this document relevant? Was it a bill from
    2 years ago? Does it relate to your taxes for 2022? Personally, I make
    tags for the year it was received, as it makes it easier to sort. You can
    further break this down by month if needed.
4. I also make tags for special categories that I need to track. For example,
  I have a tag for any documents that we'll need for our taxes in the coming
  year, or critical documents (birth certs, etc). This helps to further
  break it down.
[Reference](https://old.reddit.com/r/selfhosted/comments/sdv0rr/paperless_ng_which_tags_document_types/hugenfp/)

## Using Management Utilities
Login and switch to `paperless_ngx_srv_user` to run management utilities.

```bash
su - -s /bin/bash {{ paperless_ngx_srv_user }}
. /var/venv/paperless/bin/activate
cd /opt/paperless/paperless/src
python3 manage.py document_renamer
```

[Reference](https://docs.paperless-ngx.com/administration/)

### img2pdf
`img2pdf` has been included to provide increased functionality for import
scripts. This will enable lossless conversion of images to pdf's, enabling
import into paperless. The following example will strip alpha channel data from
png's and convert to a lossless pdf for import.

```bash
convert test.png -background white -alpha remove -alpha off out.png
img2pdf out.png -o import.pdf
```
[Reference](https://github.com/josch/img2pdf)

### ghostscript
`ghostscript` is included with paperless.

#### Reduce PDF size
This will enable you to reduce pdf size if needed. Use the following settings
for specific resolutions:
* `/screen` 72dpi
* `/ebook` 150dpi
* `/prepress` 300dpi
* `/printer` 300dpi
* `/default` no change

```bash
gs -sDEVICE=pdfwrite -dCompatibilityLevel=1.4 -dPDFSETTINGS={SETTING} -dNOPAUSE
-dQUIET -dBATCH -sOutputFile={OUTPUT}.pdf {INPUT}.pdf
```
[Reference](https://askubuntu.com/questions/113544/how-can-i-reduce-the-file-size-of-a-scanned-pdf-file)

#### Merge PDF's
This is now supported directly in the UI: select both documents and
 `actions > merge`.

```bash
gs -dNOPAUSE -sDEVICE=pdfwrite -dBATCH -sOutputFile={OUTPUT}.pdf {INPUT}1.pdf
{INPUT}2.pdf
```

[Reference](https://www.fosslinux.com/49661/merge-pdf-files-on-linux.htm)

#### Split PDF's
This is now supported directly in the UI: select both documents and
 `actions > split`.

For documents that failed consumption, manually split before re-adding to the
consumption directory.

```bash
gs -dBATCH -dPDFINFO {INPUT}.pdf
gs -dNOPAUSE -sDEVICE=pdfwrite -dBATCH -sOutputFile={OUTPUT}.pdf -dFirstPage=1
-dLastPage=3 {INPUT}.pdf
```
* Repeat for each chunk of the PDF document.

[Reference](https://stackoverflow.com/questions/10228592/splitting-a-pdf-with-ghostscript)

## Migration
In place migrations from NG to NGX can be done using this role if the
following conditions are met:
1. Ensure the last release of paperless-ng is already deployed (1.5.0) and
   successfully launched. Shutdown.
2. Backup your data, and your databases. Recommend cloning your paperless
   instance as well.
3. Role variables are migrated and updated from `paperlessng_` or `pngx_` to
   `paperless_ngx_`. Many variables have been added, removed, or changed. The
   easiest way is to start fresh using values from `defaults` and add your
   existing values into it for your host. Specifically watch out for:
4. See [proxy instructions below](#reverse-proxy-migration-changes) for reverse
   proxy changes.
4. Database backend changes are **not** supported. Database upgrades to NGX are
   done automatically during role application.

[Reference](https://docs.paperless-ngx.com/setup/#migrating-from-paperless-ng)

## Troubleshooting
https://docs.paperless-ngx.com/troubleshooting/

### PDF's fail to ingest with older ICC profiles.
Some PDF's with older ICC profiles may fail to be injested. Though rare, these
can be manually pre-processed to fix the ICC profiles:

``` bash
gs -o output.pdf -sDEVICE=pdfwrite -dPDFSETTINGS=/prepress input.pdf
```

[Reference](https://kcore.org/2021/05/08/paperless-ng/)

### PDF failed import from consumption directory.
Paperless does not clean cache aggressively and TMPFS is typically cleared only
on boot. In cases where there are mass processing of documents it is better to
use disk. Some paperless subprocesses will always use `/tmp`.

Check logs for specific errors. Increase `paperless_ngx_cfg_convert_tmpdir`
backing space if necessary and restart the machine or the consumption service:

``` bash
systemctl restart paperless-consumer.service
```
* Tasks may also be restarted in the django admin interface
  `settings > Open Django Admin > Paperless tasks`.

The service may also be stopped and `paperless_ngx_cfg_convert_tmpdir` and
`/tmp` cleared:
``` bash
systemctl stop paperless-*
rm -rf /tmp/paperless*
rm -rf {{ paperless_ngx_cfg_convert_tmpdir }}
systemctl start paperless-*
```

## Development
Configure [environment](https://github.com/r-pufky/ansible_collection_srv/blob/main/docs/dev/environment/README.md)

Run all unit tests:
``` bash
molecule test --all
```

## Issues
Create a bug and provide as much information as possible.

Associate pull requests with a submitted bug.

## License
[AGPL-3.0 License](https://www.tldrlegal.com/license/gnu-affero-general-public-license-v3-agpl-3-0)
 [(direct link)](https://github.com/r-pufky/ansible_paperless_ngx/blob/main/LICENSE)

## Author Information
PGP Fingerprint: [466EEC2B67516C7117C85CE3A0BC35D16698BAB9](https://keys.openpgp.org/vks/v1/by-fingerprint/466EEC2B67516C7117C85CE3A0BC35D16698BAB9)
| [github gist](https://gist.github.com/r-pufky/a8df36977c55b5bb20829267c4c49d22)
