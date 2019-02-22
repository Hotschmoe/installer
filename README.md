Attempt to create a termux wrapper for the althea-mesh linux installer.

Multiproceesing ---> see T0DO #4

pynacl is broken, needs to be built but no make available, read instructions here: https://wiki.termux.com/wiki/Instructions_for_installing_python_packages

# Instructions so Far (2019-02-22 --> check pynacl instructions)

```
pkg update && pkg upgrade
termux-setup-storage
pkg install clang  python-dev libffi-dev libsodium-dev openssl-dev make wireguard-tools git
git clone https://github.com/pyca/pynacl && cd pynacl
find . -type f -not -path '*/\.*' -exec sed -i 's%/bin/sh%/data/data/com.termux/files/usr/bin/sh%g' {} \; 
python setup.py install
cd ..
pip install ansible
git clone https://github.com/Hotschmoe/installer.git
cd installer
git checkout termux-althea
ansible-playbook -e @profiles/example.yml -c local -i ci-hosts install-intermediary.yml
```

Switched to python2 to get ansible<1.9.0

```
pkg install python2 python2-dev
pip2 install 'ansible<1.9.0'
```

Switched to Termux-chroot so that /var/ doesnt through error (update main.yml or always use chroot)

```
pkg install proot
termux-chroot
```

Operation 'become: true' is broken because old ansible doesnt take it. Replaced with 'sudo: yes' dnf replaced with apt (i think this will cause problems) now iptables are broken. Will try to get newer version of ansible running on termux. Possibly though arch proot

# TODO

1. Create TODO

2. Ask Justin what he thinks needs done first - listed below (discord chat)

a. Replace Rita binary for arm - installer currently has x86

b. Double check wireguard

3. Is there a stable ansible?

  https://github.com/termux/termux-packages/issues/1815#
  
  TRY THIS MAY FIX MULTIPROCESSING
  
  https://gist.github.com/hirschnase/9c2a0c6334f55bfdb373cc14dcbdf167
  
4. Ansible install correctly --> python multiprocessing does not work on Android, find the import (i think it's in ansible, create forked ansible or roll back to ansible 1.8.x) and create a fallback --> use example below
  
  https://github.com/asciinema/asciinema/issues/271
  
5. is pkg install libgmp-dev libev-dev needed? pycrypto or gevent? What about ansible 1.8.x? see below:

https://gist.github.com/hirschnase/9c2a0c6334f55bfdb373cc14dcbdf167

# Create Termux Packages

  https://github.com/termux/termux-apt-repo




-------------- From original Repo ----------------------

# Althea Installer

This is the installer for Althea on general purpose Linux. Whereas the [Althea
firmware](https://github.com/althea-mesh/althea-firmware) which is targeted at
OpenWRT compatible routers this installer will work on normal deskop and server
Linux distributions.

Eventually we will publish more traditional packages, right now this automates
the installation process and system setup.

---

## Is this where I get Althea?

If you just want Althea on your computer please download a release from
our website once it becomes available. This page is for developers who want
to help improve Althea. Or technically advanced users who want to try out cutting
edge changes.

## Getting Started

First off you need a Linux machine with Ansible.

On Ubuntu and Debian:

> sudo apt install python-pip libsqlite3-dev libssl-dev build-essential

> sudo pip install ansible

On Fedora:

> sudo dnf install ansible sqllite3-devel openssl-devel gcc

On Centos and RHEL:

> sudo yum install ansible sqllite3-devel openssl-devel gcc

All other required software will be installed by the setup playbook

## Setting up an Exit server

An Althea Exit server is essentially a WireGuard proxy server setup to integrate
with the mesh network.

Create a file named `hosts` and populate it with the ip addreses
of your exit server like so.

```
[exit]
server a
server b
```

You can then put some sort of load balancer in front of multiple servers. Optionally
of course.

Profiles are variables files pulled into Ansible for easy customization of what
the playbook will do. Edit `profiles/exit-example.yml' to match your needs. If you are running against localhost use the`-c local` option and put 'localhost' in your
hosts file.

Once configured run

> ansible-playbook -e @profiles/[your profile name or exit-example.yml][-c local if running against localhost] -i \[your hosts file or ci-hosts] install-exit.yml

To update the users or gateways list simply run again. Users should not be disrupted
unless a new gateway was added. Even then the disruption should be very minor.

## Setting up an Althea node

An Althea intermediary node will pass traffic for other users on the mesh
as well as provide secure internet acces over the mesh from a configured lan
port. This also includes gateway functionality if you include an external_nic
in the profile.

For anything you don't wish to configure (lan, wan, mesh) just provide an empty list.

Create a file named `hosts` and populate it with the ip addreses
of your devices server like so. You can use 'localhost' for your local machine.

```
[intermediary]
localhost
```

Profiles are variables files pulled into Ansible for easy customization of what
the playbook will do. Edit `profiles/example.yml` to match your needs. This should
mostly just involve setting the correct interfaces for your machine. If you are
running against localhost use the `-c local` option and put 'localhost' in your
hosts file.

Once configured run

> ansible-playbook -e @profiles/[your profile name or example.yml][-c local if running against localhost] -i [your hosts file or ci-hosts] install-intermediary.yml

To update the Rita version just run again after building a new binary and placing
it in the same folder as the playbook
