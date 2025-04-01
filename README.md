# 🧪 Vagrant K3s Cluster — A Cloud-Like Kubernetes Lab on Your Local Machine

Welcome to **Vagrant K3s Cluster**, a fully automated, modular lab setup that brings the experience of cloud-native Kubernetes clusters (like EKS, GKE, or AKS) to your local system using **Vagrant**, **K3s**, and **Longhorn**.

Designed for platform engineers, DevOps, and K8s practitioners who want a **realistic, low-friction playground** for:
- Stateful workloads
- Storage class testing (Longhorn)
- Infrastructure automation
- Ingress, ServiceLB, and HAProxy experimentation
- Monitoring stacks (e.g., kube-prometheus-stack)

---

## 🚀 Features

- ✅ Automates a complete K3s HA cluster using **Vagrant + VirtualBox**
- ✅ Modular Ansible roles for:
  - Node bootstrapping
  - Storage provisioning
  - K3s cluster setup
  - Longhorn readiness
  - HAProxy-based control-plane LB
- ✅ Support for control-plane & worker node tagging (e.g., zones)
- ✅ Optional additional disks per node (`/dev/sdb → /var/lib/longhorn`)
- ✅ Seamless PVC provisioning using Longhorn
- ✅ Designed for reproducibility, cleanliness, and modularity

---

## 🖼️ Architecture Overview

```
    ┌──────────────────────────────────────────────┐
    │                Your Host Machine             │
    │    (Vagrant + VirtualBox + Ansible + K3s)    │
    └──────────────────────────────────────────────┘
               │
      ┌────────┘───────────────────────────────┐
      │         Load Balancer                  │ (HAProxy)
      └─────┌──────────┌──────────┌────────────┘
            │          │          │
       ┌────▼─────┌────▼─────┌────▼───────────┐
       │ Controller ││ Controller││ Controller│  (HA-enabled)
       └──────────┘└───────┘└─────────┘└──────┘
            │          │          │
       ┌────▼─────┌────▼─────┌────▼───────────┐
       │  Work      ││   Worker    ││ Worker  │  (Storage-ready)
       └──────────┘└───────┘└─────────┘└──────┘
```

---

## 📁 Repo Layout

```plaintext
.
├── Vagrantfile                # VM definitions
├── cluster.yaml               # Cluster defaults
├── cluster-local.yaml         # Your local overrides (gitignored)
├── ansible/
│   ├── inventory/             # Host inventory and group vars
│   ├── files/                 # SSL certs, optional configs
│   ├── roles/                 # Modular Ansible roles
│   └── vagrant.yaml           # Full Ansible playbook
├── sharedfiles/manifests/    # Your K8s manifests (gitignored)
└── README.md
```

---

## 🧰 Requirements

- 🐧 Linux host with:
  - [VirtualBox](https://www.virtualbox.org/)
  - [Vagrant](https://www.vagrantup.com/)
  - [Ansible](https://www.ansible.com/)
  - `kubectl`, `helm`
- At least **16 GB RAM** and **4+ vCPUs** recommended
- Ansible collections:
  ```bash
  ansible-galaxy collection install kubernetes.core
  ```

---

## ⚙️ Usage

### 1. Clone the repo

```bash
git clone https://github.com/abhinav1107/vagrant-k3s-cluster.git
cd vagrant-k3s-cluster
```

### 2. Customize your cluster (Optional)

Edit `cluster-local.yaml` to override:
- Node count and resources
- Disk size
- Networking options (host-only, bridged)
- Domain names
- TLS SANs

### 3. Launch the VMs

```bash
vagrant up
```

### 4. Run the Ansible playbooks

```bash
cd ansible

# Setup base cluster, HAProxy, Longhorn, etc.
ansible-playbook vagrant.yaml
ansible-playbook haproxy.yaml
ansible-playbook k3s.yaml
```

---

## 🛠️ TODO / Coming Soon

- [ ] Generate inventory from Vagrantfile automatically
- [ ] One-click `make cluster` flow
- [ ] Pre-integrated monitoring stack (Prometheus + Grafana)
- [ ] Ingress and MetalLB baked in
- [ ] Optional Longhorn UI via Ingress

---

## 📦 Credits & Inspiration

- [K3s](https://k3s.io/)
- [Longhorn](https://longhorn.io/)
- [Vagrant Kubeadm Lab](https://github.com/techiescamp/vagrant-kubeadm-kubernetes) (ideas base)
- [ChatGPT](https://chatgpt.com) for this fire 🔥 README content.

---

## 🧑‍💻 License

MIT License. Use it, fork it, improve it.

---

## 👋 Contributing

This repo is a lab-first, opinionated, but flexible automation base. Feedback, issues, and PRs are welcome!
