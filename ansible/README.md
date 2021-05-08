# Rockstor Ansible Environment

## Introduction

You can use this ansible environment to provision a VM or physical host with a confirmed working
specification for rockstor.

## Prerequisites

It is assumed that you have the follow software packages install on you host machine:

- ansible

There are a number of ways to install this depending on your Host OS.

### MacOS

1. Install [brew](https://brew.sh)

   ```sh
   /bin/bash -c "$(curl -fsSL https://raw.githubusercontent.com/Homebrew/install/master/install.sh)"
   ```

2. Install ansible from `brew` OR with `pip`

   a. From brew:

      ```sh
      brew install ansible
      ```

   b. If you feel like running ansible from 'pip' under python 3:

      ```sh
      brew install pyenv
      pyenv install 3.9.5
      pyenv local 3.9.5
      pip install ansible
      ```

      Note: you may find you need to install the Xcode command line tools for pyenv to build your python version.

      ```sh
      xcode-select --install
      ```

### Linux

Working under linux will be slightly distribution dependent but similar to MacOS options.

OpenSUSE:
1. Install from your package manager:
    ```
    zypper install ansible
    ```
2. Install through 'pip':
    ```
    pip install ansible
    ```

### Windows

Follow this [guide](https://www.jeffgeerling.com/blog/2017/using-ansible-through-windows-10s-subsystem-linux) by Jeff Geerling.

It is also assumed that you have a local copy of this repository [rockstor-core](https://github.com/rockstor/rockstor-core)
on your machine, see the top level README.md, and are currently within the [vagrant_env](../vagrant_env) directory, where the
`Vagrantfile` is located.

## The Inventory

The file [inventory/hosts.yml](inventory/hosts.yml) contains details of the host you are provisioning with ansible. You will need to add your
hosts IP to this line:

```yaml
all:
  hosts:
    rockstor:
      ansible_ssh_host: <your_host_ip>
```

If your host has a different user from 'root' for provisioning you can also set that in the hosts on this line:

```yaml
  vars:
    ansible_ssh_user: root
```

## Running Ansible

To run the provisioner:

```sh
ansible-playbook Rockstor.yml --ask-pass
```

This will prompt you for the password of the user defined in the `ansible_ssh_user` in the
[inventory/hosts.yml](inventory/hosts.yml) file.

If you would like a little debug for the ansible execution (and this is my normal level of debug) try:

```sh
ansible-playbook Rockstor.yml --ask-pass -vv
```

If you happen to already have an authorised ssh key added to the 'ansible_ssh_user' on the rockstor host
then you can just type the following:

```sh
ansible-playbook Rockstor.yml
```
