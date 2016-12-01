ansible-openwisp2-imagegenerator
================================

[![Galaxy](http://img.shields.io/badge/galaxy-openwisp.openwisp2--imagegenerator-blue.svg?style=flat-square)](https://galaxy.ansible.com/openwisp/ansible-openwisp2-imagegenerator/)

This ansible role allows to build several openwisp2 firmware images for different organizations while keeping track of their configurations.

**NOTE**: this role has not been tested in the wild yet.
If you intend to use it, try it out and if something goes wrong please proceed to report your issue, but bear in mind you need to be willing
to understand the [build process](#build-process) and it's inner working in order to make it work for you.

Required role variables
-----------------------

The following variables are required:

* `openwisp2fw_source_dir`: indicates the directory of the OpenWRT/LEDE source that is used during [compilation](#2-compilation)
* `openwisp2fw_generator_dir`: indicates the directory used for the [preparation of generators](#3-preparation-of-generators)
* `openwisp2fw_bin_dir`: indicates the directory used when [building the final images](#4-building-of-final-images)
* `openwisp2fw_organizations`: a list of organizations; see the example below to understand its structure

```yaml
# playbook.yml
- hosts: your_host_here
  roles:
    - openwisp.openwisp2-imagegenerator
  vars:
    openwisp2fw_source_dir: /home/user/openwisp2-firmware-source
    openwisp2fw_generator_dir: /home/user/openwisp2-firmware-generator
    openwisp2fw_bin_dir: /home/user/openwisp2-firmware-builds
    openwisp2fw_organizations:
        - name: snakeoil # name of the org
          flavours: # supported flavours
            - standard
          luci_openwisp: # /etc/config/luci_openwisp
            username: "operator"
            # "password" string encrypted
            password: "$1$openwisp$iQpdG2IrO4lya98cODuUB/"
            salt: "openwisp"
            # other config keys can be added freely
          openwisp: # /etc/config/openwisp
            url: "https://my-openwisp2-instance.com"
            secret: "my-openwisp2-secret"
            unmanaged: "{{ openwisp2fw_default_unmanaged }}"
            # other config keys can be added freely
```

Other role variables
--------------------

There are many variables that can be customized if needed,
take a look at [the defaults](https://github.com/openwisp/ansible-openwisp2-imagegenerator/blob/master/defaults/main.yml) for a comprehensive list.

Some of those variables are also explained in [Organizations](#organizations) and [Flavours](#flavours).

Build process
-------------

The build process is composed of the following steps.

### 1. Installation of dependencies

**tag**: `install`

In this phase the operating system dependencies needed for the subsequent steps are installed or upgraded.

### 2. Compilation

**tag**: `compile`

The OpenWRT/LEDE source is compiled in order to produce something
called "[Image Generator](https://wiki.openwrt.org/doc/howto/obtain.firmware.generate)".
The *image generator* is an archive that contains the precompiled packages and a special `Makefile` that will be used to generate the customized images for each organization.

The source is downloaded and compiled in the directory specified in
`openwisp2fw_source_dir`.

### 3. Preparation of generators

**tag**: `generator`

During this step the *image generators* are extracted and prepared
for building different images for different [organizations](#organizations), each organization can build images for different [flavours](#flavours) (eg: full-featured, minimal, mesh, ecc);

The images are extracted and prepared in the directory specified in
`openwisp2fw_generator_dir`.

### 4. Building of final images

**tag**: `build`

In this phase a series of images is produced.

Several images will be built for each architecture, organization and flavour. This can generate quite a lof of files, therefore use this
power with caution: it's probably better to start with fewer options
and add more cases as you go ahead.

For example, if you choose to use 2 architectures (ar71xx and x86), 2 organizations (eg: A and B) and 2 flavours (eg: standard and mini), you will get 8 groups of images:

* **organization**: A / **flavour**: standard / **arch**: ar71xx
* **organization**: A / **flavour**: standard / **arch**: x86
* **organization**: A / **flavour**: mini / **arch**: ar71xx
* **organization**: A / **flavour**: mini / **arch**: x86
* **organization**: B / **flavour**: standard / **arch**: ar71xx
* **organization**: B / **flavour**: standard / **arch**: x86
* **organization**: B / **flavour**: mini / **arch**: ar71xx
* **organization**: B / **flavour**: mini / **arch**: x86

The images will be created in the directory specified in
`openwisp2fw_bin_dir`.

Organizations
-------------

If you are working with OpenWISP, there are chances you may be compiling images for different groups of people: for-profit clients, no-profit organizations or any group of people that can
be defined as an "*organization*".

Organizations can be defined freely in `openwisp2fw_organizations`.

For an example of how to do this, refer to the "[Required role variables](#required-role-variables)" section.

If you need to add specific files in the filesystem tree of the images of each organization, see "[Adding files for specific organizations](#adding-files-for-specific-organizations)".

Flavours
--------

A flavour is a combination of packages that are included in an image.

You may want to create different flavours for your images, for example:

* `standard`: the most common use case
* `minimal`: an image for device which have little storage space available
* `mesh`: an image with packages that are needed for implementing a mesh network

By default only a `standard` flavour is available.

You define your own flavours by tweaking `openwisp2fw_image_flavours` - take a look at [the default variables](https://github.com/openwisp/ansible-openwisp2-imagegenerator/blob/master/defaults/main.yml) to find out the default value of `openwisp2fw_image_flavours`.

Adding files to images
----------------------

You can add arbitrary files in every generated image by placing these files in a directory named `files/` in your playbook directory.

Example:

```
.
├── hosts
├── playbook.yml
└── files/etc/profile
```

`files/etc/profile` will be added in every generated image.

Adding files for specific organizations
---------------------------------------

You can add files to images of specific organizations too.

Let's say you have an organization called `snakeoil` and you want to add a custom banner, you can accomplish this by creating the following directory structure:

```
.
├── hosts
├── playbook.yml
└── organizations/snakeoil/etc/banner
```

Since this step is one of the last steps performed before building
the final images, you can use this feature to overwrite any file
built automatically during the previous steps.

Extra parameters
----------------

You can pass the following extra parameters to `ansible-playbook`:

* `recompile`: wether to repeat the compilation step
* `cores`: number of cores to use during the compilation step

Example:

```
ansible-playbook -i hosts playbook.yml -e "recompile=1 cores=16"
```

Run specific steps
------------------

Since each step in the process is tagged, you can run specific steps by using ansible tags.

Example 1, run only the preparation of generators:

```
ansible-playbook -i hosts playbook.yml -t generator
```

Example 2, run only the preparation of generators and build steps:

```
ansible-playbook -i hosts playbook.yml -t generator,build
```
