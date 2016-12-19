ansible-openwisp2-imagegenerator
================================

[![Galaxy](http://img.shields.io/badge/galaxy-openwisp.openwisp2--imagegenerator-blue.svg?style=flat-square)](https://galaxy.ansible.com/openwisp/openwisp2-imagegenerator/)

This ansible role allows to build several openwisp2 firmware images for different organizations while
keeping track of their configurations.

**NOTE**: this role has not been tested in the wild yet.
If you intend to use it, try it out and if something goes wrong please proceed to report your issue,
but bear in mind you need to be willing to understand the [build process](#build-process)
and it's inner working in order to make it work for you.

Required role variables
=======================

The following variables are required:

* `openwisp2fw_source_dir`: indicates the directory of the OpenWRT/LEDE source that is used during [compilation](#2-compilation)
* `openwisp2fw_generator_dir`: indicates the directory used for the [preparation of generators](#3-preparation-of-generators)
* `openwisp2fw_bin_dir`: indicates the directory used when [building the final images](#4-building-of-final-images)
* `openwisp2fw_organizations`: a list of organizations; see the example `playbook.yml` file in the
  [create playbook section](#5-create-playbook-file) to understand its structure

Usage (tutorial)
================

If you don't know how to use ansible, don't panic, this procedure will guide you step by step.

If you already know how to use ansible, you can skip this section and jump straight to the
"Install this role" section.

First of all you need to understand two key concepts:

* for **"compilation server"** we mean a server which is used to compile the images
* for **"local machine"** we mean the host from which you launch ansible, eg: your own laptop or CI server

Ansible is a configuration management/automation tool that works by entering the compilation server
via SSH and executing a series of commands.

### 1. Install ansible

Install ansible **on your local machine** if you haven't done already, there are various ways
in which you can do this, but we prefer to use the official python package manager, eg:

    sudo pip install ansible

If you don't have pip installed see [Installing pip](https://pip.pypa.io/en/stable/installing/)
on the pip documentation website.

[Installing ansible in other ways](http://docs.ansible.com/ansible/intro_installation.html#latest-release-via-yum)
is fine too, just make sure to install a version of the `2.0.x` series (which is the version with
which we have tested this playbook).

### 2. Install this role

For the sake of simplicity, the easiest thing is to install this role **on your local machine**
via `ansible-galaxy` (which was installed when installing ansible), therefore run:

    sudo ansible-galaxy install openwisp.openwisp2-imagegenerator

### 3. Choose a working directory

Choose a working directory **on your local machine** where to put the configuration of
your firmware images.

Eg:

    mkdir ~/my-openwisp2-firmware-conf
    cd ~/my-openwisp2-firmware-conf

Putting this working directory under version control is also a very good idea, it will help you to
track change, rollback, set up a CI server to automatically build the images for you and so on.

### 4. Create inventory file

The inventory file is where group of servers are defined. In our simple case we can get away with
defining a group in which we will put just one server.

Create a new file `hosts` **on your local machine** with the following contents:

    [myserver]
    mycompiler.mydomain.com ansible_user=<youruser> ansible_become_pass=<sudo-password>

Substitute `mycompiler.mydomain.com` with your hostname (ip addresses are allowed as well).

Also put your SSH user and password respectively in place of `<youruser>` and `<sudo-password>`.
These credentials are used during the [Installation of dependencies step](#1-installation-of-dependencies).

### 5. Create playbook file

Create a new playbook file `playbook.yml` **on your local machine** with the following contents:

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
            # other config keys can be added freely
            username: "operator"
            # "password" string encrypted
            password: "$1$openwisp$iQpdG2IrO4lya98cODuUB/"
            salt: "openwisp"
          openwisp: # /etc/config/openwisp
            # other config keys can be added freely
            url: "https://my-openwisp2-instance.com"
            secret: "my-openwisp2-secret"
            unmanaged: "{{ openwisp2fw_default_unmanaged }}"
          # encrypted password that will be put in /etc/shadow
          root_password: "$1$8YoE5Fl2$N49pL.Pa.6/fZ/E/f/afw/"  # encrypted version of "myPassword"
```

This playbook will let you compile firmware images for an organization named `snakeoil` using only
the `standard` flavour (which includes a default OpenWRT image with the standard OpenWISP2 modules)
for two architectures, ar71xx and x86.

See the section [Role Variables](#role-variables) to know how to customize the available configurations.

At this stage your directory layout should look like the following:

```
.
├── hosts
└── playbook.yml
```

### 6. Run the playbook

Now is time to **start the compilation of OpenWISP2 Firmware**.

Run the playbook **on your local machine** with:

    ansible-playbook -i hosts playbook.yml -e "recompile=1 cores=4"

You can substitute `cores=4` with the number of cores at your disposal.

When the playbook is done running you will find the images on **the compilation server** located in
the directory specified in `openwisp2fw_bin_dir`, which in our example is
`/home/user/openwisp2-firmware-builds` with a directory layout like the following:

```
/home/user/openwisp2-firmware-builds
├── snakeoil/                                      # (each org has its own dir)
├── snakeoil/2016-12-02-094316/ar71xx/standard/    # (contains ar71xx images for standard flavour)
├── snakeoil/2016-12-02-094316/x86/standard/       # (contains x86 images for standard flavour)
└── snakeoil/latest/                               # (latest is a symbolic link to the last compilation)
```

Now, if you followed this tutorial and everything worked out, you are ready to customize your
configuration to suit your needs! Read on to find out how to accomplish this.

Role variables
==============

There are many variables that can be customized if needed, take a look at
[the defaults](https://github.com/openwisp/ansible-openwisp2-imagegenerator/blob/master/defaults/main.yml)
for a comprehensive list.

Some of those variables are also explained in [Organizations](#organizations) and [Flavours](#flavours).

Organizations
=============

If you are working with OpenWISP, there are chances you may be compiling images for different groups
of people: for-profit clients, no-profit organizations or any group of people that can
be defined as an "*organization*".

Organizations can be defined freely in `openwisp2fw_organizations`.

For an example of how to do this, refer to the "[example playbook.yml file](#5-create-playbook-file)".

If you need to add specific files in the filesystem tree of the images of each organization, see
"[Adding files for specific organizations](#adding-files-for-specific-organizations)".

Flavours
========

A flavour is a combination of packages that are included in an image.

You may want to create different flavours for your images, for example:

* `standard`: the most common use case
* `minimal`: an image for device which have little storage space available
* `mesh`: an image with packages that are needed for implementing a mesh network

By default only a `standard` flavour is available.

You can define your own flavours by setting `openwisp2fw_image_flavours` - take a look at
[the default variables](https://github.com/openwisp/ansible-openwisp2-imagegenerator/blob/master/defaults/main.yml)
to undetstand its structure.

Build process
=============

The build process is composed of the following steps.

### 1. Installation of dependencies

**tag**: `install`

In this phase the operating system dependencies needed for the subsequent steps are installed or upgraded.

### 2. Compilation

**tag**: `compile`

The OpenWRT/LEDE source is compiled in order to produce something
called "[Image Generator](https://wiki.openwrt.org/doc/howto/obtain.firmware.generate)".
The *image generator* is an archive that contains the precompiled packages and a special
`Makefile` that will be used to generate the customized images for each organization.

The source is downloaded and compiled in the directory specified in
`openwisp2fw_source_dir`.

### 3. Preparation of generators

**tag**: `generator`

During this step the *image generators* are extracted and prepared
for building different images for different [organizations](#organizations), each organization
can build images for different [flavours](#flavours) (eg: full-featured, minimal, mesh, ecc);

The images are extracted and prepared in the directory specified in
`openwisp2fw_generator_dir`.

### 4. Building of final images

**tag**: `build`

In this phase a series of images is produced.

Several images will be built for each architecture, organization and flavour.
This can generate quite a lof of files, therefore use this power with caution: it's probably
better to start with fewer options and add more cases as you go ahead.

For example, if you choose to use 2 architectures (ar71xx and x86), 2 organizations (eg: A and B)
and 2 flavours (eg: standard and mini), you will get 8 groups of images:

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

Adding files to images
======================

You can add arbitrary files in every generated image by placing these files in a directory named
`files/` in your playbook directory.

Example:

```
.
├── hosts
├── playbook.yml
└── files/etc/profile
```

`files/etc/profile` will be added in every generated image.

Adding files for specific organizations
=======================================

You can add files to images of specific organizations too.

Let's say you have an organization called `snakeoil` and you want to add a custom banner,
you can accomplish this by creating the following directory structure:

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
================

You can pass the following extra parameters to `ansible-playbook`:

* `recompile`: wether to repeat the compilation step
* `cores`: number of cores to use during the compilation step

Example:

```
ansible-playbook -i hosts playbook.yml -e "recompile=1 cores=16"
```

Run specific steps
==================

Since each step in the process is tagged, you can run specific steps by using ansible tags.

Example 1, run only the preparation of generators:

```
ansible-playbook -i hosts playbook.yml -t generator
```

Example 2, run only the preparation of generators and build steps:

```
ansible-playbook -i hosts playbook.yml -t generator,build
```
