# relkochta/dotfiles-ansible

![An example GNOME environment configured using this playlist](/screenshots/desktop.png?raw=true)

This repository contains the [Ansible](https://www.ansible.com/) playbook that I use to configure my machines from scratch. It runs atop a fresh Fedora 34 installation and installs the GNOME desktop environment, all of my commonly-used applications, and my preferred settings.

I have tried to make it as generic as possible, so it should be relatively easy to adopt some or all of this playbook to suit your needs. However, there are some things specific to my setup that are mentioned in this document.

## Cloning

To clone the repository:

```bash
git clone https://github.com/relkochta/dotfiles-ansible.git
```

## Configuration

The playbook must be provided with both a configuration file and a secrets file.

### Configuration File

The playbook looks for `config/config-(hostname).yml`, and, if not found, uses `config/config-default.yml`. This is so that it can be used on multiple machines.

You can copy `config/config-default.yml` to `config/config-(yourhostname).yml` and make any changes you like. 

### Secrets File

Any secrets, such as user passwords and API keys, are kept in `secrets/secrets.yml`. An example is available as `secrets/secrets.yml.example`, which is well-commented. Be sure to set the variables before proceeding.

This file can optionally be encrypted using `ansible-vault`, as follows:

```bash
ansible-vault encrypt secrets/secrets.yml
```

You can edit the encrypted file as follows:

```bash
ansible-vault edit secrets/secrets.yml
```

If you want to remove the encryption, run:

```bash
ansible-vault decrypt secrets/secrets.yml
```

More information on Ansible Vault is available in the [Ansible Documentation](https://docs.ansible.com/ansible/latest/user_guide/vault.html).

## Installation

The playbook can be run either on bare metal or by creating a new Vagrant virtual machine.

### Installation using Vagrant

[Vagrant](https://www.vagrantup.com/) is a tool that allows virtual machines to be defined in a single configuration file. You will need:

* Vagrant
* Vagrant-Libvirt
* Libvirt
* Ansible

As an example, on Fedora, all four can be installed using:

```bash
sudo dnf install libvirt vagrant vagrant-libvirt ansible
```

Next, you will need to ensure your user is a member of the `libvirt` group, so that Vagrant can create the appropriate network interfaces:

```bash
sudo gpasswd -a $(whoami) libvirt
```

Install the playbook's requirements:

```bash
ansible-galaxy collection install -r requirements.yml
ansible-galaxy install -r requriements.yml
```

You will need to change some settings in the `Vagrantfile`. You can adjust the CPU core count, memory allocation, and network settings. You will need to at the very least replace 'enp6s0' with your primary ethernet interface, as documented in the file.

Finally, create the virtual machine:

```bash
vagrant up
```

The virtual machine will be created using the libvirt backend, and the graphical interface can be accessed with a tool such as [virt-manager](https://virt-manager.org/).

To rerun the playbook after the virtual machine has been created:

```bash
vagrant provision
```

To remove the virtual machine, use:

```bash
vagrant destroy
```

To access a shell on the virtual machine, use:

```bash
vagrant ssh
```

More information regarding Vagrant can be found in the [Vagrant Documentation](https://www.vagrantup.com/docs).

Usability Tip:

Vagrant defaults to a VNC output and a Cirrus virtual graphics card. While this is fine for debugging purposes, it is incredibly slow and Vagrant currently has no way to configure virtio-accelerated graphics.

To do so manually, wait until Vagrant has created and provisioned the virtual machine, then turn it off in `virt-manager`. Under `Display VNC`, set: 

`Type: Spice Server`, 

`Listen Type: None`, and

`OpenGL: Checked`

Under `Video Cirrus`, set:

`Model: Virtio`, and

`3D Acceleration: Checked`

Now start the VM again. There may be some artifacting but the performance will be better.

### Installation on Bare Metal

This playbook should work on any Fedora installation, though it has only been tested on Fedora 34.

First, you will need to remove any dotfiles in your home directory that conflict with the repository configured as `dotfiles_repo` in your configuration file. When the playbook attempts to install these, if it sees a conflict, it will not overwrite local files, but will instead raise an error. This is by design.

Install Ansible:

```bash
sudo dnf install ansible
```

Install the playbook's requirements:

```bash
ansible-galaxy collection install -r requirements.yml
ansible-galaxy install -r requriements.yml
```

Run the playbook locally:

```bash
ansible-playbook -i inventory -K main.yml
```

Or, if you used Ansible Vault to encrypt your secrets file:

```bash
ansible-playbook -i inventory -K --ask-vault-pass main.yml
```

## License

This repository is available under the MIT license, available in the LICENSE file.
