ansible-openwisp2-imagegenerator
================================

[![Galaxy](http://img.shields.io/badge/galaxy-openwisp.openwisp2--imagegenerator-blue.svg?style=flat-square)](https://galaxy.ansible.com/openwisp/ansible-openwisp2-imagegenerator/)

TODO.

Required variables:

```yaml
- hosts: compiler
  roles:
    - openwisp.openwisp2-imagegenerator
  vars:
    openwisp2fw_source_dir: /home/user/openwisp2-firmware-source
    openwisp2fw_generator_dir: /home/user/openwisp2-firmware-generator
    openwisp2fw_bin_dir: /home/user/openwisp2-firmware-builds
    openwisp2fw_organizations:
        - name: snakeoil
          flavours:
            - standard
          luci_openwisp:
            username: "operator"
            # "password" string encrypted
            password: "$1$openwisp$iQpdG2IrO4lya98cODuUB/"
            salt: "openwisp"
          openwisp:
            url: "https://my-openwisp2-instance.com"
            secret: "my-openwisp2-secret"
            unmanaged: "{{ openwisp2fw_default_unmanaged }}"
```

Compilation process
-------------------

1. compile OpenWRT/LEDE in order to produce Image Generators
2. extract ImageGenerator archives for each architecture
3. build custom images for each organization and flavour

Organizations
-------------

What is an organization

How to configure an organization

How to add files to specific organizations > link to additional files

Flavours
--------

What is a flavour: Just a variation of packages, no custom configurations (would have been to complex).

How to create new flavours

Additional files
----------------

How to add files that will be added to all organizations: put them in a dir called "files/" in your playbook directory.

How to add files for specific organizations - this means that during this step it is possible to overwrite any file generated previously.

Params
------

recompile

cores

Available playbook variables
----------------------------

TODO.
