# The Four Machines

To setup the needed virtual machines, I will use Vagrant with the libvirt provider (works well on Fedora with KVM).

The `Vagrantfile` in this repo sets up the four needed VMs.

| Name    | Description            | CPU | RAM   | Storage |
|---------|------------------------|-----|-------|---------|
| jumpbox | Administration host    | 1   | 512MB | 10GB    |
| server  | Kubernetes server      | 1   | 2GB   | 20GB    |
| node-0  | Kubernetes worker node | 1   | 2GB   | 20GB    |
| node-1  | Kubernetes worker node | 1   | 2GB   | 20GB    |

## Install Vagrant and the libvirt provider

```bash
sudo dnf install -y vagrant vagrant-libvirt libvirt libvirt-devel \
    gcc make ruby-devel qemu-kvm
sudo systemctl enable --now libvirtd
sudo usermod -aG libvirt $(whoami)
newgrp libvirt
```

If the plugin isn't bundled, install it explicitly:

```bash
vagrant plugin install vagrant-libvirt
```

Verify it's installed:

```bash
vagrant plugin list
```

## Network

Each VM gets a static IP on a dedicated `kthw` libvirt network (defined in the `Vagrantfile`), so machines can reach each other by address without depending on DHCP:

| Name    | IP             |
|---------|----------------|
| jumpbox | 192.168.56.10  |
| server  | 192.168.56.11  |
| node-0  | 192.168.56.12  |
| node-1  | 192.168.56.13  |

> Note: an explicit subnet is required for `private_network` — omitting it causes a `to_range` crash in vagrant-libvirt when it tries to auto-derive a DHCP range.

## SSH access

The `Vagrantfile` provisions a dedicated SSH keypair (generated once into `.kthw-keys/`) so `jumpbox` can SSH into `server`, `node-0`, and `node-1` without a password, matching the access pattern this tutorial expects.

## Start the VMs

```bash
vagrant up --provider=libvirt
```

Check status:

```bash
vagrant status
```

## Verify

SSH into any machine and confirm the OS version matches the tutorial's requirement:

```bash
vagrant ssh jumpbox
cat /etc/os-release
```

Expected output:

```text
PRETTY_NAME="Debian GNU/Linux 12 (bookworm)"
NAME="Debian GNU/Linux"
VERSION_ID="12"
VERSION="12 (bookworm)"
VERSION_CODENAME=bookworm
ID=debian
```

## Useful commands

```bash
vagrant halt          # stop all VMs
vagrant destroy -f    # tear everything down
vagrant reload        # apply Vagrantfile changes
```
