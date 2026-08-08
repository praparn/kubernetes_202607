# Advanced Docker with Kubernetes 136 - Course Outline

Companion repo: kubernetes_202607 (github.com/praparn/kubernetes_202607)
Target: Kubernetes v1.36.2 | New SkillLane course (separate product from the live "Kubernetes134" course)

## Module 1: Kubernetes Foundations

| # | Topic | What's new for 1.36 |
|---|---|---|
| 1.1 | Install Kubernetes (kubeadm) | containerd 2.0+ required; kube-proxy mode nftables (IPVS removed); cgroups v1 disabled by default |
| 1.2 | Pods, Service, Deployment | - |
| 1.3 | Replication Controller | - |
| 1.4 | Deployment | - |
| 1.5 | Volume | - |
| 1.6 | Liveness / Readiness Probe | - |
| 1.7 | Resource Management, Namespaces, QoS, Dashboard | - |
| 1.8 | ConfigMap & Secret | - |

## Module 2: Advanced Orchestration & Operations

| # | Topic | What's new for 1.36 |
|---|---|---|
| 2.1 | Job & CronJob | - |
| 2.2 | Log and Monitoring | - |
| 2.3 | **Ingress Network → Gateway API** | Full rebuild: NGINX Ingress retired (Mar 2026) → Kubernetes Gateway API standard with **Apache APISIX**; HTTP, TLS, and key-auth (PluginConfig/Consumer) labs |
| 2.4 | Security | **NEW:** Mutating Admission Policies (GA, CEL-based); **NEW:** User Namespaces (GA); existing NetworkPolicy/RBAC/PSA/encryption-at-rest labs |
| 2.5 | Kubernetes in the Real World | Multi-node kubeadm cluster on AWS, Cilium CNI, Cloud Controller, Dashboard, **APISIX + AWS NLB** (replacing NGINX Ingress AWS deploy) |
| 2.6 | Orchestrator Assignment (scheduling) | nodeSelector, affinity/anti-affinity, taints & tolerations - unchanged |
| 2.7 | Persistent Storage | **NEW:** OCI VolumeSource (stable); **NEW:** alpha in-place EBS volume resize; existing AWS EBS CSI dynamic provisioning |
| 2.8 | HPA Workshop | **NEW:** in-place Pod resource resize / vertical scaling (ties in directly with HPA) |
| 2.9 | Helm Workshop | - |
| 2.10 | Monitor (Prometheus + Grafana) | - |

## What Changed: 1.34 → 1.36 (Headline Features)

**Removed / breaking:**
- NGINX Ingress Controller retired (no releases/patches since March 2026)
- kube-proxy IPVS mode removed (nftables is the replacement)
- containerd 1.x no longer supported (containerd 2.0+ required)
- cgroups v1 disabled by default
- `gitRepo` volume plugin removed

**New / graduated to stable:**
- User Namespaces (GA)
- Mutating Admission Policies (GA, CEL-based, no webhook required)
- OCI VolumeSource (stable)
- In-place Pod resource resize (GA in 1.35) + Pod-level vertical scaling (1.36)
- Structured Authentication Configuration (GA)
- PreferSameNode traffic distribution (stable)

## Suggested Delivery Order for Video Recording

1. Open with "what changed since 134" (headline features above) as a hook/trailer
2. Module 1 as-is (foundations mostly unchanged, but re-record Part 0 of 1.1 with nftables/containerd 2.0 checks)
3. Module 2 in order, with 2.3 (Gateway API/APISIX) as the flagship new section given the NGINX retirement story
4. Close with 2.7/2.8 (storage + HPA) since both got meaningful new 1.36 features worth demoing live
