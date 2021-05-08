# Rockstor Development Environment

## Introduction

You can use this vagrant environment to build a vagrant box VM of a confirmed working
specification with rockstor installed from source for testing.

It will use the ansible playbook and roles in this repository to perform the VM provisioning. However,
Vagrant generates it's own host inventory since it operates using the vagrant user. eg.

```ruby
ANSIBLE_GROUPS = {
  "all_groups" => ["rockstor"],
  "all_groups:vars" => {
        "rs_inst_set_root_pass" => "true"
    }
}
```

## Prerequisites

It is assumped that you have the follow software packages install on you host machine:

- [Vagrant](https://www.vagrantup.com/downloads)
- Ansible - see this [README](ansible/README.md) for download instructions.

All above packages are available on Linux, MacOS, and Windows.

You will also need one of the following virtualisation platforms:

- **Linux users:** may prefer to use libvirt, qemu + kvm (see [vagrant-libvirt](https://github.com/vagrant-libvirt/vagrant-libvirt))
- Virtualbox is available on all platforms: [Download](https://www.virtualbox.org/wiki/Downloads)

It is also assumed that you have a local copy of the core repository [rockstor-core](https://github.com/rockstor/rockstor-core)
on your machine at the same level as this repo:

```sh
./rockstor-core
./rockstor-dev-env
```

You should also be currently within the [vagrant_env](/vagrant_env) directory in this repo, where the `Vagrantfile` is located.

## Vagrant Boxes

This Vagrantfile uses a number of vagrant boxen:

- [opensuse/leap-15.2.x86_64](https://app.vagrantup.com/opensuse/boxes/Leap-15.2.x86_64)
- [opensuse/Tumbleweed.x86_64](https://app.vagrantup.com/opensuse/boxes/Tumbleweed.x86_64)
- [opensuse/Tumbleweed.aarch64](https://app.vagrantup.com/opensuse/boxes/Tumbleweed.aarch64)

## Building an OpenSUSE Leap VM and installing Rockstor from source

On MacOS, Linux and Windows:

```sh
./build.sh
```

This will, by default, build and provision an OpenSUSE Leap 15.2 vagrant box in libvirt - including rsyncing the source to the
VM in /root. It will then build and install Rockstor from source.

Should you prefer to use VirtualBox you can perform the following:

```sh
./build.sh leap virtualbox
```

The VM window should have popped up and show the usual Rockstor messages regarding how to reach the Web UI. e.g.

```sh
Rockstor is successfully installed.

web-ui is accessible with the following links:
https://127.0.0.1
https://10.0.2.15
https://172.28.128.101
https://172.28.128.100
rockstor login:
```

Note: your IPs may vary.

CAUTION: This command will ALWAYS do a clean build due to the nature of the vagrant rsync which uses the rsync
option `--delete`.

To do subsequent builds see the [Aliases](#aliases) section that follows on how to login as root and run further operations.

## Accessing Rockstor on the VMs

Port forwarding is used to provide local access to the Rockstor Web UI. To allow you to have Rockstor built and
running in each platform side by side port the ports are assigned as follows:

|VM Name|Host Port|Guest Port|Example URL|
|-------|---------|----------|-----------|
|rockstor-leap|2443|443|[https://172.28.128.101:2443](https://172.28.128.101:2443)|
|rockstor-tw|2444|443|[https://172.28.128.101:2444](https://172.28.128.101:2444)|
|rockstor-pi|2445|443|[https://172.28.128.101:2445](https://172.28.128.101:2445)|

## Aliases

Some aliases are defined in the file [alias_rc](vagrant_env/alias_rc) to assist with some commands detailed in
[the developer contribution docs](http://rockstor.com/docs/contribute.html#developers).

These are appended onto the file `/root/.bashrc` so that they are available
when logged in as the root user.

Typically in the world of vagrant you login as the user `vagrant` and use sudo for privilege escalation.
However, for ease in this case and in line with the instructions you can also log in as `root` in one of
the following ways:

```sh
host> vagrant ssh
vm> su root
password: vagrant
vm> cd
```

or

```sh
host> ssh root@<vm IP>
password: vagrant
```

If you have set the ansible option `rs_inst_set_root_pass: yes` then the root password will be defined in
[ansible/roles/rockstor_installer/defaults/main.yml](ansible/roles/rockstor_installer/defaults/main.yml)
and defaults to `password`.

From here the aliases can be used. Here is a summary of them:

```sh
build_init
build_all
build_ui
make_mig_stadm
apply_migrations_stadm
make_mig_smtmgr
apply_migrations_smtmgr
```

See [alias_rc](vagrant_env/alias_rc) for details

## Deploy from Testing Channel

If you wish to deploy a VM from the Test Channel RPMs (rather than building from source) you need to update
the ansible playbook [ansible/Rockstor.yml](ansible/Rockstor.yml) to set the variable
`rs_inst_install_from_repo: yes` as follows:

```yaml
  roles:
    - role: rockstor_installer
      vars:
        rs_inst_install_from_repo: yes
```

Then simply bring up the VM using the command:

```sh
vagrant up
```

Do NOT run `build.sh` in this scenario.

## Building for Tumbleweed and Aarch64

To build for Tumbleweed x86_64 in libvirt:

```sh
./build.sh tw
```

or for VirtualBox:

```sh
./build.sh tw virtualbox
```

To build for Tumbleweed Aarch64 in libvirt:

NOTE: Work in progress... only works on libvirt [see vagrant libvirt plugin](https://github.com/vagrant-libvirt/vagrant-libvirt):

```sh
./build.sh pi
```

## Managing the Virtual Machine

To manage the Vagrant box VM simply type the following from this directory...

- Enable disk management to allow creation of data disks requires the enablement
  of the experimental `disk` feature. You can set these for the current shell, or
  enable them permanently by setting them in `~/.profile`, `~/.bashrc`, or likewise:

  ```sh
  export VAGRANT_EXPERIMENTAL="disks"
  export VAGRANT_DEFAULT_PROVIDER=kvm
  ```

  Note: This is enabled in `build.sh` but needs external enablement if you wish to use discrete vagrant calls below.

- Bring up all vagrant box VMs in libvirt

  ```sh
  vagrant up --provider=libvirt
  ```

  or for VirtualBox

  ```sh
  vagrant up --provider=virtualbox
  ```

  Note: You can NOT manage boxes in both libvirt and VirtualBox at the same time.

- Bring up a particular vagrant box VM

  ```sh
  vagrant up <box-name eg. rockstor-core> --provider=libvirt
  ```

- Reconfigure the vagrant box VM following a change to the Vagrantfile:

  ```sh
  vagrant reload
  ```

  or

  ```sh
  vagrant reload <box-name eg. rockstor-core>
  ```

- If you change the provisioner section of the Vagrantfile or update the ansible, you can rerun just that part as follows:

  ```sh
  vagrant provision
  ```

  or

  ```sh
  vagrant provision <box-name eg. rockstor-core>
  ```

- Destroy the vagrant box VM:

  ```sh
  vagrant destroy
  ```

  or

  ```sh
  vagrant destroy <box-name eg. rockstor-core>
  ```

- If you wish to ssh into the vagrant box VM to poke around, try this:

  ```sh
  vagrant ssh
  ```

  or

  ```sh
  vagrant ssh <box-name eg. rockstor-core>
  ```

- If you wish to re-rsync your source files (warning this will have the effect of cleaning the build too due
to using the --delete options):

  ```sh
  vagrant rsync
  ```

  or

  ```sh
  vagrant rsync <box-name eg. rockstor-core>
  ```

## Debugging Ansible

If you need additional debug for the ansible configuration add additional `v`'s to the
`ansible.verbose` variable in the `Vagrantfile`. eg.

```ruby
    config.vm.provision "rockstor", type: "ansible" do |ansible|
        ansible.config_file = "../ansible/ansible.cfg"
        ansible.playbook = "../ansible/Rockstor.yml"
        ansible.verbose = "vvvv"
        ansible.groups = ANSIBLE_GROUPS
```
