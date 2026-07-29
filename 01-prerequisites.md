# Prerequisites

In this lab you will review the machine requirements necessary to follow this tutorial.

## Virtual or Physical Machines

This tutorial requires four (4) virtual or physical ARM64 or AMD64 machines running Debian 12 (bookworm). The following table lists the four machines and their CPU, memory, and storage requirements.

| Name    | Description            | CPU | RAM   | Storage |
|---------|------------------------|-----|-------|---------|
| jumpbox | Administration host    | 1   | 512MB | 10GB    |
| server  | Kubernetes server      | 1   | 2GB   | 20GB    |
| node-0  | Kubernetes worker node | 1   | 2GB   | 20GB    |
| node-1  | Kubernetes worker node | 1   | 2GB   | 20GB    |

How you provision the machines is up to you, the only requirement is that each machine meet the above system requirements including the machine specs and OS version. Once you have all four machines provisioned, verify the OS requirements by viewing the `/etc/os-release` file:

```bash
cat /etc/os-release
```

You should see something similar to the following output:

```text
PRETTY_NAME="Debian GNU/Linux 12 (bookworm)"
NAME="Debian GNU/Linux"
VERSION_ID="12"
VERSION="12 (bookworm)"
VERSION_CODENAME=bookworm
ID=debian
```

## My solution

Use Vagrant to provision the machines. Install `vagrant`, add the `vagrant-libvirt` and `vagrant-disksize` plugin, and use the `Vagrantfile` to define the machines.

Install the vagrant-libvirt and vagrant-disksize plugins:
```bash
vagrant plugin install vagrant-libvirt
vagrant plugin install vagrant-disksize
```

Start the Vagrant machines:
```bash
vagrant up --provider=libvirt
```

Next: [setting-up-the-jumpbox](02-jumpbox.md)
