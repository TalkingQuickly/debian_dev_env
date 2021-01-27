# Debian based development environment with Ansible

This repo contains an Ansible setup which given SSH access to a fresh Debian install, will setup everything required for modern web development. This is currently primarily used as a remote development environment for VSCode in the following setups:

1. Running a Virtualbox VM with a Debian guest on MacOS
1. Running a cloud VM (e.g. GCP, AWS, Hetzer etc) with Debian installed
1. Running high powered bare metal servers locally with Debian installed

The driver behind this style of development has been:

1. Greatly improved Docker performance, especially when dealing with large numbers of small files
1. The ability to easily switch between different machines for development, e.g. to upgrade to a high powered remote server when needed

For more details on how to use this repo, see here: <https://www.talkingquickly.co.uk/2021/01/debian-dev-environment-for-remote-vscode/>

## Prerequisites

Ansible must be [installed](https://docs.ansible.com/ansible/latest/installation_guide/intro_installation.html). This is often as simple as `pip3 install ansible`.

And then dependent roles and collections must be fetched with:

```
ansible-galaxy install -r requirements.yml
```

## Usage

Ensure that you have SSH access to the target VM and that your SSH public key is in authorized keys, e.g:

```
ssh-copy-id SSH_USER@VPS_IP
```

If using a local VM (e.g. Virtualbox) with port forwarding (in this example with SSH forwarded to host port 22) then use:

```
ssh-copy-id -p 2222 SSH_USER@localhost
```

Then create an inventory file. This is a standard Ansible inventory file, e.g:

```yml
all:
  hosts:
    HOSTNAME_OR_IP_ADDRESS:
      ansible_user: SSH_USER_NAME
      main_user: DESIRED_DEV_USER_NAME
      ssh_port: SSH_PORT
      initial_become_method: su # this should be `su` if the SSH user does not, by default have sudo access, otherwise this should be `sudo`
```

and save it as something like `inventory.dev.yml`.

Then run:

```
ansible-playbook -i inventory.dev.yml main.yml --ask-become-pass
```

And enter the root or sudo password for remote machine when prompted for "ASK Password". This playbook currently assumes that the root and sudo passwords are the same.

Then just wait and enjoy!

## Remote development with VSCode

Make sure you have the [Remote Extension Pack](https://marketplace.visualstudio.com/items?itemName=ms-vscode-remote.vscode-remote-extensionpack) installed. For more about how VSCode development via SSH works [go here](https://code.visualstudio.com/docs/remote/ssh).

## SSH Forwarding

Setting up SSH forwarding means that when you're connected to the remote machine and the remote machine needs to access something using your ssh key (e.g. cloning a git repo), this can be done using the SSH keys on your local machine, without ever needing to copy these onto the remote machine.

To set this up, edit your `~/.ssh/config` and add something like the following (for a local Virtualbox host):

```
Host localdev
  HostName localhost
  User SSH_USERNAME
  Port SSH_PORT
  ForwardAgent yes
```

The host will then show up with the name `localdev` in the VSCode Remote Explorer (when "SSH Targets" is selected).

You can also SSH into it directly simply with `ssh localdev` rather than `ssh -p SSH_PORT SSH_USERNAME@localhost`. 