# Setting Up The Jumpbox

The jumpbox is the administration host — all commands to configure the
Kubernetes cluster will be run from here. The `Vagrantfile` provisions it
automatically.

## Machine specs

| Attribute | Value              |
|-----------|--------------------|
| OS        | Debian 12 (bookworm) |
| CPU       | 1                  |
| RAM       | 512MB              |
| Disk      | 10GB               |
| IP        | 192.168.56.10      |
| Network   | `kthw` private (libvirt) |

## SSH access

A dedicated ed25519 keypair is generated once into `.kthw-keys/`:

```bash
ls .kthw-keys/
# id_ed25519  id_ed25519.pub
```

The public key is pushed to all four VMs (`vagrant` and `root` users). The
private key is placed on the jumpbox only, along with an SSH config that
disables strict host key checking for the cluster machines:

```
Host server node-0 node-1
  StrictHostKeyChecking no
  UserKnownHostsFile /dev/null
```

## Accessing the jumpbox

```bash
vagrant ssh jumpbox         # as vagrant user
sudo -i                     # switch to root
```

## Connecting to cluster machines from the jumpbox

```bash
ssh root@server             # → server.kubernetes.local
ssh root@node-0             # → node-0.kubernetes.local
ssh root@node-1             # → node-1.kubernetes.local
```

No password required — key-based auth is pre-configured.
