# Provisioning Compute Resources

The `Vagrantfile` handles the full compute-resources lab automatically.
A `machines.txt` file is also committed for reference and to match the
guide's workflow.

## Machine database (`machines.txt`)

| IP             | FQDN                        | Hostname | Pod Subnet      |
|----------------|-----------------------------|----------|-----------------|
| 192.168.56.11  | server.kubernetes.local     | server   | —               |
| 192.168.56.12  | node-0.kubernetes.local     | node-0   | 10.200.0.0/24   |
| 192.168.56.13  | node-1.kubernetes.local     | node-1   | 10.200.1.0/24   |

## What Vagrant automates

- **Root SSH access** — `PermitRootLogin yes` + public key in `/root/.ssh/authorized_keys` on server, node-0, node-1.
- **SSH key distribution** — jumpbox has the private key; cluster machines have the public key for both `vagrant` and `root`.
- **Hostnames + FQDNs** — `hostnamectl set-hostname` plus the `127.0.1.1` line in `/etc/hosts` set to `FQDN HOST`.
- **Cross-machine `/etc/hosts`** — every machine (including jumpbox) has entries for all three cluster members, so `ssh root@server` resolves without DNS.

## Verification

From the jumpbox as `root`:

```bash
for host in server node-0 node-1; do
  ssh root@${host} hostname --fqdn
done
```

Expected output:

```text
server.kubernetes.local
node-0.kubernetes.local
node-1.kubernetes.local
```
