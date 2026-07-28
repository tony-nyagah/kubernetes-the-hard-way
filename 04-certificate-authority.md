# Provisioning a CA and Generating TLS Certificates

All certificate operations are run manually from the jumpbox as `root`.

## ca.conf

The `ca.conf` file (from the upstream repo) is stored in the project root
and was copied to the jumpbox with:

```bash
vagrant upload ca.conf /home/vagrant/ca.conf jumpbox
```

It defines sections for every Kubernetes component that needs a certificate:

| Section                   | CN                                | O                                  |
|---------------------------|-----------------------------------|------------------------------------|
| `admin`                   | `admin`                           | `system:masters`                  |
| `node-0`                  | `system:node:node-0`              | `system:nodes`                    |
| `node-1`                  | `system:node:node-1`              | `system:nodes`                    |
| `kube-proxy`              | `system:kube-proxy`               | `system:node-proxier`             |
| `kube-scheduler`          | `system:kube-scheduler`           | `system:system:kube-scheduler`    |
| `kube-controller-manager` | `system:kube-controller-manager`  | `system:kube-controller-manager`  |
| `kube-api-server`         | `kubernetes`                      | —                                  |
| `service-accounts`        | `service-accounts`                | —                                  |

Key SANs on the API server cert:
- `kubernetes`, `kubernetes.default`, `kubernetes.default.svc`, etc.
- `server.kubernetes.local`, `api-server.kubernetes.local` — match Vagrant FQDNs
- `10.32.0.1` — Kubernetes service cluster IP
- `127.0.0.1`

## Commands (run as root on jumpbox)

### 1. Generate the CA

```bash
openssl genrsa -out ca.key 4096
openssl req -x509 -new -sha512 -noenc \
  -key ca.key -days 3653 \
  -config ca.conf \
  -out ca.crt
```

Produces: `ca.key`, `ca.crt`

### 2. Generate component certificates

```bash
certs=(
  "admin" "node-0" "node-1"
  "kube-proxy" "kube-scheduler"
  "kube-controller-manager"
  "kube-api-server"
  "service-accounts"
)

for i in ${certs[*]}; do
  openssl genrsa -out "${i}.key" 4096
  openssl req -new -key "${i}.key" -sha256 \
    -config "ca.conf" -section ${i} \
    -out "${i}.csr"
  openssl x509 -req -days 3653 -in "${i}.csr" \
    -copy_extensions copyall \
    -sha256 -CA "ca.crt" \
    -CAkey "ca.key" \
    -CAcreateserial \
    -out "${i}.crt"
done
```

Produces a `.key`, `.csr`, and `.crt` for each component.

### 3. Distribute to worker nodes

```bash
for host in node-0 node-1; do
  ssh root@${host} mkdir /var/lib/kubelet/
  scp ca.crt root@${host}:/var/lib/kubelet/
  scp ${host}.crt root@${host}:/var/lib/kubelet/kubelet.crt
  scp ${host}.key root@${host}:/var/lib/kubelet/kubelet.key
done
```

### 4. Distribute to the control plane node

```bash
scp \
  ca.key ca.crt \
  kube-api-server.key kube-api-server.crt \
  service-accounts.key service-accounts.crt \
  root@server:~/
```
