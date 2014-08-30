# Packer templates for CentOS
[![Build Status](https://box-cutter.ci.cloudbees.com/buildStatus/icon?job=centos-vm)](https://box-cutter.ci.cloudbees.com/job/centos-vm/)

### Overview

This repository contains templates for CentOS that can create Vagrant boxes
using Packer.

## Current Boxes

64-bit boxes:

* [box-cutter/centos70](https://vagrantcloud.com/box-cutter/centos70) - CentOS 7.0 (64-bit), VMware 441MB/VirtualBox 380MB
* [box-cutter/centos70-desktop](https://vagrantcloud.com/box-cutter/centos70-desktop) - CentOS 7.0 Desktop (64-bit), VMware 1.2GB/VirtualBox 1.1GB
* [box-cutter/centos70-docker](https://vagrantcloud.com/box-cutter/centos70-docker) - CentOS 7.0 Core with Docker (64-bit), VMware 430MB/VirtualBox 370MB
* [box-cutter/centos65](https://vagrantcloud.com/box-cutter/centos65) - CentOS 6.5 (64-bit), VMware 452MB/VirtualBox 385MB
* [box-cutter/centos65-desktop](https://vagrantcloud.com/box-cutter/centos65-desktop) - CentOS 6.5 Desktop (64-bit), VMware 1.1GB/VirtualBox 1GB
* [box-cutter/centos65-docker](https://vagrantcloud.com/box-cutter/centos65-docker) - CentOS 6.5 with Docker (64-bit), VMware 456MB/VirtualBox 379MB
* [box-cutter/centos64](https://vagrantcloud.com/box-cutter/centos64) - CentOS 6.4 (64-bit), VMware 423MB/VirtualBox 353MB
* [box-cutter/centos64-desktop](https://vagrantcloud.com/box-cutter/centos64-desktop) - CentOS 6.4 Desktop (64-bit), VMware 1GB/VirtualBox 1016MB
* [box-cutter/centos510](https://vagrantcloud.com/box-cutter/centos510) - CentOS 5.10 (64-bit), VMware 248MB/VirtualBox 180MB
* [box-cutter/centos59](https://vagrantcloud.com/box-cutter/centos59) - CentOS 5.9 (64-bit), VMware 247MB/VirtualBox 179MB

32-bit boxes:

* [box-cutter/centos65-i386](https://vagrantcloud.com/box-cutter/centos65-i386) - CentOS 6.5 (32-bit), VMware 400MB/VirtualBox 352MB
* [box-cutter/centos64-i386](https://vagrantcloud.com/box-cutter/centos64-i386) - CentOS 6.4 (32-bit), VMware 355MB/VirtualBox 318MB
* [box-cutter/centos510-i386](https://vagrantcloud.com/box-cutter/centos510-i386) - CentOS 5.10 (32-bit), VMware 237MB/VirtualBox 170MB
* [box-cutter/centos59-i386](https://vagrantcloud.com/box-cutter/centos59-i386) - CentOS 5.9 (32-bit), VMware 236MB/VirtualBox 169MB

## Building the Vagrant boxes

To build all the boxes, you will need Packer and both VirtualBox and VMware Fusion
installed.

A GNU Make `Makefile` drives the process via the following targets:

    make        # Build all the box types (VirtualBox & VMware)
    make test   # Run tests against all the boxes
    make list   # Print out individual targets
    make clean  # Clean up build detritus

### Proxy Settings

The templates respect the following network proxy environment variables
and forward them on to the virtual machine environment during the box creation
process, should you be using a proxy:

* http_proxy
* https_proxy
* ftp_proxy
* rsync_proxy
* no_proxy
    
### Tests

The tests are written in [Serverspec](http://serverspec.org) and require the
`vagrant-serverspec` plugin to be installed with:

    vagrant plugin install vagrant-serverspec
    
The `Makefile` has individual targets for each box type with the prefix
`test-*` should you wish to run tests individually for each box.

Similarly there are targets with the prefix `ssh-*` for registering a
newly-built box with vagrant and for logging in using just one command to
do exploratory testing.  For example, to do exploratory testing
on the VirtualBox training environmnet, run the following command:

    make ssh-box/virtualbox/centos65-nocm.box
    
Upon logout `make ssh-*` will automatically de-register the box as well.

### Makefile.local override

You can create a `Makefile.local` file alongside the `Makefile` to override
some of the default settings.  The variables can that can be currently
used are:

* CM
* CM_VERSION
* HEADLESS
* <iso_path>
* UPDATE

`Makefile.local` is most commonly used to override the default configuration
management tool, for example with Chef:

    # Makefile.local
    CM := chef

Changing the value of the `CM` variable changes the target suffixes for
the output of `make list` accordingly.

Possible values for the CM variable are:

* `nocm` - No configuration management tool
* `chef` - Install Chef
* `chefdk` - Install Chef Development Kit
* `puppet` - Install Puppet
* `salt`  - Install Salt

You can also specify a variable `CM_VERSION`, if supported by the
configuration management tool, to override the default of `latest`.
The value of `CM_VERSION` should have the form `x.y` or `x.y.z`,
such as `CM_VERSION := 11.12.4`

The variable `HEADLESS` can be set to run Packer in headless mode.
Set `HEADLESS := true`, the default is false.

The variable `UPDATE` can be used to perform OS patch management.  The
default is to not apply OS updates by default.  When `UPDATE := true`,
the latest OS updates will be applied.

Another use for `Makefile.local` is to override the default locations
for the Ubuntu install ISO files.

For CentOS, the ISO path variables are:

* CENTOS59_X86_64
* CENTOS59_I386
* CENTOS510_X86_64
* CENTOS510_I386
* CENTOS64_X86_64
* CENTOS64_I386
* CENTOS65_X86_64
* CENTOS65_I386
* CENTOS70_X86_64

This override is commonly used to speed up Packer builds by
pointing at pre-downloaded ISOs instead of using the default
download Internet URLs:
`CENTOS65_X86_64 := file:///Volumes/CentOS/CentOS-6.5-x86_64-bin-DVD1.iso`

## Contributing


1. Fork and clone the repo.
2. Create a new branch, please don't work in your `master` branch directly.
3. Add new [Serverspec](http://serverspec.org/) or [Bats](https://blog.engineyard.com/2014/bats-test-command-line-tools) tests in the `test/` subtree for the change you want to make.  Run `make test` on a relevant template to see the tests fail (like `make test-virtualbox/centos65`).
4. Fix stuff.  Use `make ssh` to interactively test your box (like `make ssh-virtualbox/centos65`).
5. Run `make test` on a relevant template (like `make test-virtualbox/centos65`) to see if the tests pass.  Repeat steps 3-5 until done.
6. Update `README.md` and `AUTHORS` to reflect any changes.
7. If you have a large change in mind, it is still preferred that you split them into small commits.  Good commit messages are important.  The git documentatproject has some nice guidelines on [writing descriptive commit messages](http://git-scm.com/book/ch5-2.html#Commit-Guidelines).
8. Push to your fork and submit a pull request.
9. Once submitted, a full `make test` run will be performed against your change in the build farm.  You will be notified if the test suite fails.

## Acknowledgments

[CloudBees](http://www.cloudbees.com) is providing a hosted [Jenkins master](http://box-cutter.ci.cloudbees.com/) through their CloudBees FOSS program. Their [On-Premise Executor](https://developer.cloudbees.com/bin/view/DEV/On-Premise+Executors) feature is used to connect physical machines as build slaves running VirtualBox, VMware Fusion, VMware Workstation, VMware ESXi/vSphere and Hyper-V.

![Powered By CloudBees](http://www.cloudbees.com/sites/default/files/Button-Powered-by-CB.png "Powered By CloudBees")![Built On DEV@Cloud](http://www.cloudbees.com/sites/default/files/Button-Built-on-CB-1.png "Built On DEV@Cloud")
