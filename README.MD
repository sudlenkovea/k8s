# Ansible Role **RKE2 Setup**

This role **prepares the system**, **installs**, and **configures** a Kubernetes cluster based on **RKE2** (Rancher Kubernetes Engine 2).
A three-phase model is implemented to reliably and sequentially deploy or expand a cluster.

---

## Execution Phases

### Phase 1. System preparation (runs on **all** hosts)

1. **system_preparation.yml**
   - Update repositories and packages
   - Install required utilities
   - Remove unnecessary software
   - Configure system limits (`sysctl`, `limits.d`)
   - Reboot the node and wait for it to come back online if required

### Phase 2. RKE2 installation & configuration (runs on **all** hosts)

2. **download_rke2_locally.yml** — cache RKE2 binaries locally
3. **install_rke2.yml** — install RKE2 server/agent and systemd units
4. **setup_kubectl_kubeconfig.yml** — install `kubectl` and distribute kubeconfig
5. **config_rke_plugins.yml** — configure Ingress-NGINX, CoreDNS, and other addons

### Phase 3. Cluster bootstrap & verification (sequential operations)

6. **start_rke2_leader.yml** — start the first master (leader) node
7. **configure_rke2_vars.yml** — retrieve and propagate the bootstrap token
8. **start_rke2_masters.yml** — join remaining master nodes
9. **start_rke2_workers.yml** — join worker nodes
10. **verify_cluster.yml** — check node readiness and cluster health

---

## Role Variables

### Default values (`defaults/main.yml`)

```yaml
# RKE2 version
rke2_version: "v1.32.5+rke2r1"

# Paths
rke2_local_download_path: "/tmp/rke2_downloads"
rke2_remote_install_path: "/tmp/rke2_install"

# System settings
configure_sysctl_limits: true
configure_custom_inotify: true
reboot_after_system_prep: true

# Packages
system_packages:
  - net-tools
  - screen
  - software-properties-common
  - ufw
  - telnet
  - traceroute
  - qemu-guest-agent

common_packages:
  - mc
  - unzip
  - iftop
  - iotop

packages_to_remove:
  - fail2ban
  - metricbeat
  - snapd
```

Additional variables (e.g. `cluster_name`, `kubelet_args`, `node_taint`, HelmChartConfig contents) are defined in inventory or via `-e`.

---

## Supported Tags

| Tag                    | Purpose                                                                     |
|------------------------|------------------------------------------------------------------------------|
| `cluster_setup`        | Complete cluster deployment (all three phases).                              |
| `cluster_expand`       | Add new nodes to an existing cluster.                                        |
| `cluster_addons_config`| Install / update only addons (Ingress-NGINX, CoreDNS, etc.).                 |
| `system_preparation`   | Run only Phase 1: OS preparation, limits, reboot.                            |

Tags allow flexible role execution for specific tasks without affecting others.

---

## Usage Example

```yaml
- name: Deploy RKE2 cluster
  hosts: all
  become: true
  roles:
    - rke2_setup
  vars:
    cluster_name: "production"
    rke2_version: "v1.32.5+rke2r1"
```

Run full deployment:

```bash
ansible-playbook -i rke2-inventory-example.yaml rke2_setup.yaml \
  --become --tags cluster_setup -u <sudo_or_root_user>
```

Run only system preparation:

```bash
ansible-playbook -i rke2-inventory-example.yaml rke2_setup.yaml \
  --become --tags system_preparation -u <sudo_or_root_user>
```

---

## Requirements

* Ansible **2.9+**  
* Target Linux hosts with **sudo** privileges  
* Network access to package repositories & GitHub for RKE2 binaries  

---

## License

MIT  

Author — Evgenii Sudlenkov
