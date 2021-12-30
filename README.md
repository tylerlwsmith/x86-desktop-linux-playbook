# x86 Desktop Linux Playbook

This Ansible Playbook sets up a new x86 desktop Linux machine for development. Nuking-and-paving laptops and setting up VMs was getting tedious, so I wanted to automate the process.

The Playbook installs Node.js, Yarn, Docker, Docker Compose, VS Code + extensions, TLDR, Ansible, VIM, jq, and Terminator. It also creates a `~/projects/` and `~/contributions/` directory for storing code.

## Installing the Playbook dependencies

Run the following to install the playbook's dependencies:

```sh
ansible-galaxy install -r requirements.yml
```

## Prepping the target machine

Unfortunately, configuring a new machine with this Playbook isn't _quite_ a 100% automated process: you still need to do some prep on the target machine.

First, check if the SSH service is installed and enabled by default on the machine by running `sudo systemctl status ssh`. If there is no SSH service, installed open SSH server by running the following:

```sh
sudo apt update
sudo apt install openssh-server
```

If the OS is running on hardware instead of inside a VM, you will also need the network IP address of the target device to connect to it. You can get the device's network IP address by running `ifconfig`. Conversely, you could set a hostname in the `/etc/hostname` file which would remove the need for a network IP address.

## Running

This playbook doesn't use an inventory file, so it takes quite a few command line flags. On the `--inventory` flag, notice the comma at the end of `tylerlaptop.local,`. That comma is required: without it, Ansible will assume that `tylerlaptop.local` is a filename.

```sh
ansible-playbook playbook.yml --inventory tylerlaptop.local, --user [your-username] --ask-pass --ask-become-pass
```

If you're running this Playbook on a Linux installation in a local VM, you'll need to pass the forwarding port. You can do that by tacking `--extra-vars "ansible_port=2222"` to the end of the command:

```sh
ansible-playbook playbook.yml --inventory 127.0.0.1, --user [your-username] --ask-pass --ask-become-pass --extra-vars "ansible_port=2222"
```

After running the Playbook, you will need to reboot the target machine.

## Docker installation issues

When running the playbook on Pop!\_OS, the Docker installation task fails because it looks for a `pop!_os` GPG key from Docker. You need to override the Ansible distribution variable by passing in `ansible_distribution=Ubuntu` to the `--extra-vars` argument.

```sh
ansible-playbook playbook.yml --inventory 127.0.0.1, --user [your-username] --ask-pass --ask-become-pass --extra-vars "ansible_port=2222 ansible_distribution=Ubuntu"
```

## Other notes

Paramiko is set as the SSH client because Ansible's default client doesn't seem to support password authentication.
