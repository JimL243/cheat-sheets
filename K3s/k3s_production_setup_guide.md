## K3s Production Setup Guide

This document provides a comprehensive, repeatable step-by-step guide for setting up a production-grade K3s Kubernetes cluster, complete with HA (High Availability), persistent storage, CI/CD pipeline, DNS configuration, monitoring, backup strategy, and secure external/internal access via Cloudflare and Active Directory DNS.

---

### Prerequisites

Before proceeding, ensure the following requirements are met:

* **Cluster Nodes:**

  * Minimum 3 nodes:

    * 1x Master node (control plane)
    * 2x Worker nodes
    * Recommended: promote additional nodes to manager for HA
  * Static IPs assigned
  * Hostnames clearly defined using naming convention (e.g., `DC1-K3-MASTER`, `DC1-K3-WORKER1`)

* **Infrastructure:**

  * VLAN and firewall configured for inter-node communication (ports 6443, 10250, 8472/UDP, etc.)
  * DNS:

    * Cloudflare for external domain management (e.g., `*.jlvcorp.com`)
    * Active Directory for internal DNS resolution (e.g., `*.internal.jlvcorp.com`)

* **Storage:**

  * Synology NAS with NFS export:

    * Mount path: `/mnt/nfs/K3s-Shared`
    * Permissions set for all nodes (read/write, no\_root\_squash)

* **Access & Tools:**

  * SSH access to all nodes
  * Administrator system with kubectl and k3sup/helm installed
  * Backup location available on NAS

---

### Step 1: Install K3s

#### Master Node Installation

On the master node:

```bash
curl -sfL https://get.k3s.io | sh -s - server \
  --disable traefik \
  --tls-san <CLUSTER_VIP> \
  --write-kubeconfig-mode 644
```

* `--disable traefik`: disables bundled Traefik (you’ll install your own)
* `--tls-san`: required for HA and VIP support

#### Retrieve the Node Token:

```bash
sudo cat /var/lib/rancher/k3s/server/node-token
```

#### Copy Kubeconfig:

```bash
sudo cat /etc/rancher/k3s/k3s.yaml
```

Edit server IP to use cluster VIP or external DNS name.

#### Join Worker Nodes:

```bash
curl -sfL https://get.k3s.io | K3S_URL=https://<CLUSTER_VIP>:6443 \
  K3S_TOKEN=<TOKEN> sh -s - agent
```

Repeat on each worker.

---

### Step 2: Enable NFS Storage

#### Mount NFS Share on All Nodes:

```bash
sudo mkdir -p /mnt/nfs/K3s-Shared
sudo apt install -y nfs-common
sudo mount -t nfs <NAS_IP>:/volume1/nfs_shared /mnt/nfs/K3s-Shared
```

Add to `/etc/fstab` for persistence:

```
<NAS_IP>:/volume1/nfs_shared /mnt/nfs/K3s-Shared nfs defaults 0 0
```

#### Configure Dynamic Provisioning

Option 1: **Longhorn**

* Lightweight distributed block storage
* UI and snapshot support

Option 2: **NFS Subdir External Provisioner**

```bash
helm repo add nfs-subdir-external-provisioner https://kubernetes-sigs.github.io/nfs-subdir-external-provisioner/
helm install nfs-client nfs-subdir-external-provisioner/nfs-subdir-external-provisioner \
  --set nfs.server=<NAS_IP> \
  --set nfs.path=/volume1/nfs_shared \
  --set storageClass.defaultClass=true
```

---

### Step 3: Deploy Core Services

#### Kube-VIP (for HA VIP)

```bash
kubectl apply -f https://raw.githubusercontent.com/kube-vip/kube-vip/main/docs/manifests/rbac.yaml
```

Create a DaemonSet or Static Pod depending on cluster mode.

#### Cert-Manager

```bash
kubectl apply -f https://github.com/cert-manager/cert-manager/releases/download/v1.14.2/cert-manager.yaml
```

Install Cloudflare issuer for wildcard DNS challenge.

#### Ingress Controller

Option 1: **Traefik v3** with CRDs Option 2: **Nginx Ingress Controller** with TLS termination

#### Internal DNS (Optional)

* **Pi-hole** in K3s for DNS routing control
* Set AD DHCP to use Pi-hole, fallback to AD DNS

---

### Step 4: Deploy Applications

All apps defined via YAML manifests under `/home/dc1k3admin/k8s/<app>/` and deployed via `kubectl apply -f`.

* **Uptime Kuma** – service monitoring
* **Gitea** – self-hosted Git
* **Drone CI** – continuous integration and deployment
* **Paperless-NGX** – external Docker, managed by Drone

---

### Step 5: Setup CI/CD

#### GitOps Flow

* Code is committed to Gitea
* Drone CI watches tagged Releases
* Upon Release:

  * Clone repo
  * Run `kubectl diff` and apply changes
  * Backup deployment manifests
  * Notify via Slack/email

#### Security

* Use YubiKey+1Password for secrets
* Drone RPC secret stored securely

#### Deployment Control

* Tagging = production deploy
* Manual `git pull` allowed during transition

---

### Step 6: Backup Strategy

#### Persistent Volume Backup

* All PVCs backed up under: `/mnt/nfs/K3s-Shared/<app>/backups/`

#### Schedule:

* Daily backups: keep 7 days
* Monthly backups: keep 12 months
* Use cronjob or systemd timer

#### Retention Policy:

* Latest 2 backups: uncompressed
* 3rd to 10th oldest: gzip
* Older than 10th: delete

---

### Step 7: Monitoring with Prometheus + Grafana

#### Prometheus Deployment

* Use the kube-prometheus-stack Helm chart for production-ready setup:

```bash
helm repo add prometheus-community https://prometheus-community.github.io/helm-charts
helm repo update
helm install monitoring prometheus-community/kube-prometheus-stack \
  -n monitoring --create-namespace \
  -f /home/dc1k3admin/k8s/monitoring/values.yaml
```

* Customize `values.yaml` for:

  * External ingress
  * Retention and storage
  * Alertmanager routing
  * ServiceMonitor and PodMonitor integration

#### Grafana Access

* Default admin login: `admin/prom-operator`
* Ingress should expose Grafana via `grafana.internal.jlvcorp.com`
* Enable NFS persistent volume for dashboards:

```yaml
  persistence:
    enabled: true
    storageClassName: "nfs-client"
    accessModes: ["ReadWriteOnce"]
    size: 5Gi
```

#### Alerts and Dashboards

* Preconfigured K3s, node-exporter, and Kubernetes dashboards
* Define alerts for node status, pod health, PVC usage, and system metrics
* Email/Slack integration via Alertmanager

#### Data Retention and Backup

* Store Prometheus data on NFS if persistence is enabled
* Consider exporting Grafana dashboards regularly for version tracking

---

### Next Steps (To Be Added)

* GitOps with ArgoCD or Flux
* Disaster recovery walkthrough
* Automatic node scaling (if applicable)
* Certificate auto-renewal validation

---

Contact: `jim@lavacca.net` | Last updated: May 26, 2025
