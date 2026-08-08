---
marp: true
theme: default
paginate: true
footer: "Kubernetes: Production Workload Orchestration"
---

<!-- PDF p.1 -->
# Advanced Docker with Kubernetes
## Kubernetes v1.36 (Haru) Edition

**Kubernetes: Production Workload Orchestration**

By: Praparn L (eva10409@gmail.com)

---

<!-- PDF p.2 -->
## Outline Part 1

- Container concept (Recap)
- Introduction to Kubernetes
- System Architecture
- Fundamental of Kubernetes
  - Pods, Container and Services
  - Daemon Sets and Replication Controller (RC)
  - Deployment/Replica-Set (RS) and Rolling update
  - Volume
  - Resource Management and Horizontal Pods Autoscaling (HPA)
  - Liveness, Readiness and Startup Probe
  - ConfigMap Secret

---

<!-- PDF p.3 -->
## Outline Part 2

- Fundamental of Kubernetes: Job and CronJob / Log and Monitoring
- **Gateway API Networking (Ingress successor)**
- Security on Kubernetes
  - Network Policy / Volume Policy
  - Resource Usage Policy / Resource Consumption Policy
  - Access Control Policy / Security Policy
  - **Mutating Admission Policies (NEW: GA in 1.36)**
  - **User Namespaces (NEW: GA in 1.36)**
- Kubernetes in real world
  - Cluster Setup for Bare Metal
  - Orchestrator Assignment (nodeSelector / Interlude / Affinity / Taints & Tolerations)
- Stateful application deployment (Consideration / Persistent Volumes / StatefulSets)
- Helm basic operation
- Monitoring with prometheus operator

---

<!-- PDF p.4 -->
## Pre-require

- Windows 11 (64 bit) / Mac OSX (64 bit) with memory 16 GB
- Intermediate understanding of docker/compose and container concept
- Cloud / Bare Metal / Play with K8S
- Account on hub.docker.com
- Basic understanding of network / load balance concept
- Tool for editor (VS Code etc.)
- Tool for shell (putty / terminal etc.)
- Tool for transfer file (winscp / scp)
- Internet for download / upload image

---

<!-- PDF p.5-8 -->
## Lab Resource

- Repository for lab *(screenshot: GitHub repo page)*
- Software in lab *(screenshot: lab software list)*
- Account on hub.docker.com *(screenshot: Docker Hub signup/login)*

---

<!-- PDF p.9 — UPDATED for 1.36 -->
## Lab Resource

- Download on Google Drive
  - https://tinyurl.com/k8slabth
- Download on GitHub
  - `git clone https://github.com/praparn/kubernetes_202607`

Setup script:
`https://github.com/praparn/sourcesetup/blob/master/standard_kubernetes360_containerd.sh`

---

<!-- PDF p.10-11 -->
## Workshop: Install Kubernetes

- LAB Sheet
  - https://tinyurl.com/y8yazrxy
- Target version: **Kubernetes v1.36.2** on **containerd 2.x**
- Pre-flight checks new in this course:
  - `sudo lsmod | grep -e nf_tables -e nf_conntrack` (kube-proxy **nftables** mode)
  - `sudo ctr --version` (containerd 2.0+)
  - cgroups **v2** required (cgroups v1 is disabled by default in 1.36)

---

<!-- PDF p.12 — REBUILT: was "Kubernetes 1.34 Wind & Will" -->
## Kubernetes 1.36 Wind & Will (O' WaW)

**v1.36 "Haru" — released 22 April 2026**

- 70 enhancements: 18 Stable, 25 Beta, 25 Alpha
- Release theme: security hardening, AI/ML workloads, API scalability

Ref: https://kubernetes.io/blog/2026/04/22/kubernetes-v1-36-release/

---

<!-- PDF p.13 -->
## Kubernetes 1.36 Wind & Will (O' WaW)

**Removed / Breaking in 1.36 — check before upgrade!**

- **kube-proxy IPVS mode removed** (deprecated in 1.35) → migrate to **nftables**
- **`gitRepo` volume plugin removed** (security risk)
- **containerd 1.x no longer supported** → containerd 2.0+ required
- **cgroups v1 disabled by default** → hosts must run cgroups v2

Ref: https://kubernetes.io/blog/2026/04/22/kubernetes-v1-36-release/

---

<!-- PDF p.14 -->
## Kubernetes 1.36 Wind & Will (O' WaW)

**Ingress NGINX retired (24 March 2026)**

- Retired by SIG Network and the Security Response Committee
- No more releases or security patches
- Replacement path: **Kubernetes Gateway API** (this course uses **Apache APISIX**)
- We rebuild the whole Ingress chapter on Gateway API (Part 2)

Ref: https://kubernetes.io/blog/2026/03/

---

<!-- PDF p.15 -->
## Kubernetes 1.36 Wind & Will (O' WaW)

**GA (Stable) in 1.36: User Namespaces**

- Container's root user maps to a non-privileged user on the host
- Container escape ≠ node admin access
- Enable per-pod: `hostUsers: false`

Ref: https://kubernetes.io/blog/2026/04/22/kubernetes-v1-36-release/

---

<!-- PDF p.16 -->
## Kubernetes 1.36 Wind & Will (O' WaW)

**GA (Stable) in 1.36: Mutating Admission Policies**

- Mutation logic as a native Kubernetes object using **CEL** (Common Expression Language)
- No webhook server to build/operate → lower latency, less complexity
- Complements Validating Admission Policies (GA since 1.30)

Ref: https://kubernetes.io/blog/2026/04/22/kubernetes-v1-36-release/

---

<!-- PDF p.17 -->
## Kubernetes 1.36 Wind & Will (O' WaW)

**GA (Stable) in 1.36: OCI VolumeSource (image volume)**

- Mount an **OCI image** directly as a read-only volume in a Pod
- Share files/models/tools among containers without baking them into the main image
- Great for AI/ML model weights and CLI tool bundles

**Also stable:** fine-grained kubelet API authorization, PreferSameNode traffic distribution, Structured Authentication Configuration

---

<!-- PDF p.18 -->
## Kubernetes 1.36 Wind & Will (O' WaW)

**In-place Pod resource resize / vertical scaling**

- Resize CPU/memory of a running Pod **without restart** (GA since 1.35)
- 1.36 adds pod-level vertical scaling improvements
- Ties directly into the HPA/Resource-Management chapter (demo in Workshop 2.8)

Ref: https://kubernetes.io/docs/tasks/configure-pod-container/resize-container-resources/

---

<!-- PDF p.19 -->
## Kubernetes 1.36 Wind & Will (O' WaW)

**Dynamic Resource Allocation (DRA) keeps maturing**

- Standard way to request GPUs / accelerators / special hardware
- v1.36: more drivers, feature graduations, extends toward native resources (CPU/memory)

Ref: https://kubernetes.io/blog/2026/05/07/kubernetes-v1-36-dra-136-updates/

---

<!-- PDF p.20 -->
## Kubernetes 1.36 Wind & Will (O' WaW)

**KYAML — Kubernetes YAML dialect**

- Safer, less ambiguous YAML subset (introduced alpha in 1.34, `kubectl` output format)
- Solves the "Norway bug" / whitespace pitfalls
- 100% compatible with existing YAML tooling

Ref: https://medium.com/@simardeep.oberoi/kyaml-kubernetes-answer-to-yaml-s-configuration-chaos-0c0c09f51587

---

<!-- PDF p.21 -->
## Kubernetes 1.36 Wind & Will (O' WaW)

**Ecosystem watch**

- AI/ML workload support matures (DRA, image volumes for model weights)
- nftables is now the assumed dataplane — distro iptables deprecations no longer matter
- *(diagram: community issues / ecosystem screenshots)*

Ref: https://www.infoq.com/news/2026/05/kubernetes-1-36-released/

---

<!-- PDF p.22-27 -->
## Kubernetes in the wild 2025/2026

*(6 slides of survey charts from the "Kubernetes in the wild 2025" report — see `kubernetes-in-the-wide-2025.pdf` in the repo)*

- Adoption by industry / cluster size trends
- Managed vs self-hosted distribution share
- CNI / ingress / observability tooling market share

---

<!-- PDF p.28 -->
## Kubernetes Environment

*(diagram: release/support timeline)*

- v1.36 supported ~14 months (April 2026 → ~June 2027)
- Patch cadence monthly; always check release notes before upgrade

Ref: https://kubernetes.io/docs/setup/release/notes/

---

<!-- PDF p.29 -->
# Introduction to Kubernetes

---

<!-- PDF p.30-31 -->
## Workshop: Install Kubernetes

- Follow LAB Sheet (AWS machines + `standard_kubernetes360_containerd.sh`)
- Alternate Solution: https://labs.play-with-k8s.com/

---

<!-- PDF p.32 -->
## Introduction to Kubernetes

- Check kubernetes version

```bash
kubectl get nodes -o yaml
kubectl version
```

Expect: `Server Version: v1.36.x`

---

<!-- PDF p.33 -->
## Introduction to Kubernetes

*(diagram: Kubernetes logo / ecosystem overview)*

---

<!-- PDF p.34 -->
## Introduction to Kubernetes

- What is Orchestration (Computing)?
  - Align business request with Application/Data/Infrastructure
  - Centralized management for:
    - Resource Pool
    - Automated Workflow
    - Provisioning
    - Scale Up/Down
    - Monitoring
    - Billing
    - etc.

Ref: https://en.wikipedia.org/wiki/Orchestration_(computing)

---

<!-- PDF p.35 -->
## Introduction to Kubernetes

- But why do we need Container Orchestration?
  - Production environment is a cluster system
  - Microservices require maintained connectivity
  - Stateful applications must run on stateless architecture
  - Application scale up/down is required
  - Too many native commands/shells to maintain containers in production manually
  - etc.

---

<!-- PDF p.36-37 -->
## Introduction to Kubernetes

*(diagram: Kubernetes features & SIGs landscape)*

---

<!-- PDF p.38-39 -->
## Introduction to Kubernetes

**Physical View**

*(diagram: control plane + worker nodes on physical machines)*

Ref: https://www.cncf.io/blog/2019/08/19/how-kubernetes-works/
Ref: https://phoenixnap.com/kb/understanding-kubernetes-architecture-diagrams

---

<!-- PDF p.40 -->
## Introduction to Kubernetes

**Logical View**

*(diagram: namespaces, deployments, services logical layering)*

---

<!-- PDF p.41 -->
## Introduction to Kubernetes

- Key Features
  - Automatic binpacking
  - Horizontal Pod Autoscaling (HPA)
  - Automated rollouts and rollbacks
  - Storage orchestration
  - Self-healing
  - Service discovery and load balancing
  - Secret and configuration management
  - Batch execution

---

<!-- PDF p.42 -->
## Introduction to Kubernetes

- Automatic binpacking
  - CPU/Memory utilization can be defined on Pods (smallest unit of Kubernetes)
  - Scheduler selects the node ensuring all resources are enough to run the Pod
  - If memory limit reached:
    - Current Pod will be terminated (killed)
    - If restart flag set, Kubernetes will restart the Pod (possibly on another node)
  - If CPU limit reached:
    - Scheduler will not kill the Pod — it is throttled until back to normal
  - Check available node resource: `kubectl describe node`

---

<!-- PDF p.43 -->
## Introduction to Kubernetes

- Horizontal Pod Autoscaling (HPA)
  - Works with the workload controller (Deployment/ReplicaSet) to complete the task
  - Automatically scales Pod count based on CPU / memory utilization
  - Supports multiple criteria (metrics) and custom/external metrics
  - Control loop checks resource utilization every 15 seconds (default)
  - **New for 1.36 era:** combine with in-place Pod resize for vertical scaling

---

<!-- PDF p.44 -->
## Introduction to Kubernetes

- Automated Rollout and Rollbacks
  - Update is based on the Pod template
  - Rollout keeps the desired replica count during the change
  - If rollout succeeds, the new version is provisioned to completion
  - Rollback is possible with a single command
  - Rollout process can pause/resume as needed

---

<!-- PDF p.45 -->
## Introduction to Kubernetes

- Storage Orchestrator
  - Supports several storage types via **CSI drivers**:
    - Local Storage
    - Network Storage (NFS, iSCSI, Ceph)
    - Cloud Storage (AWS EBS, GCE PD, Azure Disk — all via CSI)
    - **OCI images as read-only volumes (stable in 1.36)**

---

<!-- PDF p.46 -->
## Introduction to Kubernetes

- Self-healing
  - The replica controller maintains the designed number of Pods
    (not too many → kill; not too few → create)
  - Fulfills Pods automatically on every failure case

---

<!-- PDF p.47 -->
## Introduction to Kubernetes

- Service Discovery and Load Balancing
  - Service acts like a "connector" for clients that need to reach Pods
  - Discovery via environment variables or DNS service
  - Supports load balancing between multiple Pods (replicas)

---

<!-- PDF p.48-49 -->
## Introduction to Kubernetes

- Secret and Configuration Management
  - Kubernetes can keep confidential data (username/password) for applications in encoded format
  - Reference it in Pods instead of plain-text configuration
  - ConfigMap: central configuration file defining all app configuration, referenced by Pods

---

<!-- PDF p.50 -->
## Introduction to Kubernetes

- Batch Execution
  - A Job creates special Pods that terminate when the batch completes
  - Kubernetes uses jobs for background processes in the cluster
  - Types of Job:
    - Non-parallel jobs
    - Parallel jobs with fixed completion count
    - Parallel jobs with work queue

---

<!-- PDF p.51 -->
# System Architecture

---

<!-- PDF p.52-53 -->
## System Architecture

*(diagram: control plane / worker node components)*

Ref: https://platform9.com/blog/kubernetes-enterprise-chapter-2-kubernetes-architecture-concepts/

---

<!-- PDF p.54 -->
## System Architecture

- Kubernetes Control Plane (Master)
  - **API Server**: interface for all UIs/commands interacting with the cluster
  - **Scheduler** (job dispatcher): schedules node resources, dispatches Pods to nodes matching criteria (CPU? Memory? Affinity/Constraint?)
  - **Controller Manager**: controls/coordinates all nodes to maintain the configured state
  - **etcd** (open-source): key-value database keeping state of nodes/Pods/containers
  - Secret: encoded confidential data
  - HPA: scales pods on CPU/memory criteria
  - Event: keeps log and events of the cluster
  - Namespace: limits resource quota (cpu, memory, pods etc.)

---

<!-- PDF p.55 -->
## System Architecture

- Kubernetes Node
  - **Pods**: smallest deployable unit of computing in Kubernetes (Pods != Container)
  - **Container runtime**: engine running containers, via standard CRI — **containerd 2.0+** (or CRI-O); containerd 1.x is not supported by 1.36
  - **Kubelet**: agent on node talking to the control plane, reports node status/health
  - **Kube-Proxy**: manages service networking on the node — default dataplane is now **nftables**

---

<!-- PDF p.56-57 -->
## System Architecture

*(diagram: full cluster component cheat-sheet)*

Ref: http://k8s.info/cs.html

---

<!-- PDF p.58 -->
## System Architecture

| Topic | K8S | Docker/Swarm |
|---|---|---|
| Architecture | Open-system (based on Google's "Borg") | Swarm: proprietary, "easy to use" |
| Operation | Mostly declarative (YAML) | Mostly imperative (commands) |
| Unit of Work | Pods (Pods >= Container) | Container |
| Identify Work | Label operation | Container / service / stack name |
| Workload mgmt | Service / Replication / Deployment levels | Service level only |
| Auto scaling | HPA (CPU, memory, custom) | No |
| Health check | Liveness & Readiness & Startup | Service health only |

---

<!-- PDF p.59-60 -->
## System Architecture

*(diagram: request flow through API server → etcd → scheduler → kubelet)*

---

<!-- PDF p.61 -->
# Fundamental of Kubernetes

---

<!-- PDF p.62 -->
# Pods, Container and Services

---

<!-- PDF p.63 -->
## Pods, Container and Services

- Pods vs Container
  - Docker's viewpoint:
    - 1 Container : 1 Application, 1 component of a microservice
    - So for microservice we need multiple containers (cache / web / database / etc.)
  - Kubernetes viewpoint:
    - 1 Pod = 1 Container
    - 1 Pod = N Containers (containers in the same context, working closely)
    - So one Pod can hold more than one container

---

<!-- PDF p.64 -->
## Pods, Container and Services

- All containers in the same Pod share:
  - Process ID (PID)
  - Network access (communicate with each other via `localhost`)
  - Inter-Process Communication (IPC)
  - Unix Time-Sharing (UTS), hostname
  - IP Address/Ports
- Use cases for multi-container Pods:
  - Apache + Tomcat / Apache + PHP / Nginx cache + Apache/PHP
  - Web server + data-volume container
  - Service Mesh sidecars (Istio / Kong etc.)
- Pods can be replicated to 1000+ per cluster

---

<!-- PDF p.65 -->
## Pods, Container and Services

- Pod Life-Cycle
  - **Pending**: accepted by the cluster, but containers not yet set up (includes scheduling wait and image download)
  - **Running**: bound to a node, all containers created, at least one running or starting/restarting
  - **Succeeded**: all containers terminated successfully and will not restart
  - **Failed**: all containers terminated, at least one failed (non-zero exit or killed by the system)
  - **Unknown**: state of the Pod could not be obtained (typically node communication error)

---

<!-- PDF p.66-67 -->
## Pods, Container and Services

- Container / Pods / Service / Deployment / RS

*(diagram: object layering from container up to deployment)*

---

<!-- PDF p.68 -->
## Pods, Container and Services

- Ways to deploy objects in kubernetes by kubectl
  - Imperative commands:
    - `kubectl <action> <type/name> <option>`
    - Ex: `kubectl run webtest --image labdocker/nginx:latest`
  - Imperative object configuration:
    - `kubectl <action> -f <YAML file>`
    - Ex: `kubectl create -f nginx.yml` / `kubectl replace -f nginx_update.yml`
  - Declarative object configuration:
    - `kubectl apply -f <directory>`

---

<!-- PDF p.69 -->
## Pods, Container and Services

- Imperative Command:
  - Create object via interactive shell
    - `run`: create new pods
    - `expose`: create new service to access pods
    - `autoscale`: create autoscale for deployment etc.
    - `create`: create specific object — `kubectl create service nodeport myservice`
  - Update object via interactive shell
    - `scale`: add/remove replicas · `annotate` · `label`
    - `set <field>`: define specific value — `kubectl set env deployment/registry ac=/local`
    - `edit`: direct edit live object · `patch`: update fields in a single command

---

<!-- PDF p.70 -->
## Pods, Container and Services

- Imperative Command:
  - Delete object via interactive shell
    - `delete`: delete element by command
    - Ex: `kubectl delete pods/abandon`
    - Ex: `kubectl delete service/abdc`

---

<!-- PDF p.71 -->
## Pods, Container and Services

- Imperative commands:
  - `kubectl run --image=<image name> <option>`
  - Options: `--env="key=value"`, `--port`, `--labels`, `--overrides=<json>`
  - Note: generator flags creating Deployment+RC via `kubectl run` are **removed** — `kubectl run` now creates a bare Pod only; use `kubectl create deployment` for workloads

---

<!-- PDF p.72 -->
## Pods, Container and Services

- Imperative commands:

```bash
kubectl expose deployment <name> <option>
```

  - `--name`, `--port` (service port), `--target-port` (container port)
  - `--type=<NodePort/ClusterIP/LoadBalancer>`
  - `--protocol=<TCP/UDP>`

---

<!-- PDF p.73 -->
## Pods, Container and Services

- Imperative commands: *(terminal screenshots: run + expose + get)*

---

<!-- PDF p.74 -->
## Pods, Container and Services

- Imperative object configuration
  - Create Pods (YAML) / Create Service (YAML)

```bash
kubectl create -f <Filename>
```

*(YAML listings: pod spec and service spec side by side)*

---

<!-- PDF p.75 -->
## Pods, Container and Services

- Imperative object configuration
  - Create Pods (KYAML) / Create Service (KYAML)

```bash
kubectl create -f <Filename>
```

*(KYAML listings: same objects in KYAML syntax)*

---

<!-- PDF p.76 -->
## Pods, Container and Services

- KYAML (Kubernetes YAML)
  - YAML has been the fundamental config format of Kubernetes since the beginning
  - Advantages: human readable, comments, datatypes, multi-document, superset of JSON
  - After 10+ years, users hit YAML limitations (whitespace-sensitive indentation errors,
    implicit type coercion "Norway bug") — often worked around with Helm/Kustomize

---

<!-- PDF p.77 -->
## Pods, Container and Services

- KYAML: Kubernetes YAML (Safer YAML Subset)
  - Introduced as **alpha in Kubernetes 1.34** (KEP-5295) to reduce YAML ambiguity
  - KYAML is 100% compatible with existing tools (no new parser): kubectl, yamllint, devops workflows
  - Tools exist to convert yaml → kyaml (ex: yamlfmt)
  - Expect adoption to grow as KYAML matures through beta

---

<!-- PDF p.78 -->
## Pods, Container and Services

- Rules of KYAML
  - **Double-quoted strings**: all string values in `" "`
    - YAML: `name: app` → KYAML: `name: "app"`
  - **No indentation dependency**: removes reliance on whitespace for structure
  - **JSON-like structure**:
    - Objects always use `{ }`
    - Arrays always use `[ ]`
    - Comma between objects

---

<!-- PDF p.79-81 -->
## Pods, Container and Services

*(screenshots: KYAML example, yamllint, kube-score)*

Ref: https://www.yamllint.com/
Ref: https://github.com/zegl/kube-score#installation

---

<!-- PDF p.82-83 -->
## Pods, Container and Services

- Imperative object configuration

```bash
kubectl create -f <Filename>
```

*(terminal screenshots: creating pod + service from files)*

---

<!-- PDF p.84 -->
## Pods, Container and Services

- Check log on container:

```bash
kubectl logs <Pods name> -c <container name>
```

- Shell inside container:

```bash
kubectl exec -it <Pods name> -c <container name> -- sh
```

---

<!-- PDF p.85-86 -->
## Pods, Container and Services

- Check detail property of Pods/Service

```bash
kubectl describe <Pods/SVC/etc> <Name>
```

---

<!-- PDF p.87 -->
## Pods, Container and Services

- Check overall of Pods/Service

```bash
kubectl get <Pods/SVC/etc>
```

- Remove Pod/Service

```bash
kubectl delete -f <Filename>
```

---

<!-- PDF p.88 -->
## Pods, Container and Services

Kubectl reference guide:

Ref: https://kubernetes.io/docs/reference/kubectl/generated/

---

<!-- PDF p.89 -->
## Workshop: Pods & Service

- Part 1: Deploy simple web pods with single container

*(diagram: Client → WebServer Frontend, Port 5000)*

---

<!-- PDF p.90 -->
## Pods, Container and Service

- Example Case: Basic Restful API

*(diagram: Frontend Client → Restful WebService [Port 5000] → Database [Port 3306];
Restful JSON: Add / Delete / Show)*

---

<!-- PDF p.91 -->
## Pods, Container and Service

- Example Case: High I/O Restful API

*(diagram: Frontend Client → Web Cache [Port 80] → Restful WebService [Port 5000]
→ DB Cache [Port 6379] → Database [Port 3306])*

---

<!-- PDF p.92-94 -->
## Pods, Container and Service

- Restful WebService endpoints:
  - `/init`
  - `/insertuser`
  - `/removeuser/<uid>`
  - `/users/<uid>`

*(terminal screenshots: curl calls for each endpoint)*

---

<!-- PDF p.95-96 -->
## Pods, Container and Service

- Web Cache *(nginx cache container)*
- Database and Database Cache
  - MySQL Database
  - Redis Key-Value Database

---

<!-- PDF p.97 -->
## Pods, Container and Service

*(diagram: Pods "web" [Web Cache:80 + WebService:5000] and Pods "maindb" [Redis:6379 + MySQL:3306], communicating via localhost inside pods;
Service maindb: ClusterIP, port 3306; Service web: NodePort 30500→80, 32500→5000)*

Request: `http://<node-ip>:30500` / `http://<node-ip>:32500`

---

<!-- PDF p.98 -->
## Pods, Container and Service

- Images used in lab:
  - `labdocker/cluster:webcache_kubenetes`
  - `labdocker/cluster:webservice`
  - `labdocker/mariadb:latest`
  - `labdocker/redis:latest`

---

<!-- PDF p.99-100 -->
## Pods, Container and Service

- Pods: maindb (YAML) / Service: maindb (YAML)
- Pods: web (YAML) / Service: web (YAML)

*(YAML listings from `WorkShop_1.2_Pods_Service_Deployment`)*

---

<!-- PDF p.101-105 -->
## Pods, Container and Service

- Create "maindb" Pods and service
- Create "web" Pods and service
- Initial database and insert data
- Get data (Direct/Cache)
- Delete data (both cache and database), then remove all

*(terminal screenshots per step)*

---

<!-- PDF p.106 -->
## Workshop: Pods & Service

- Part 2: Deploy web pods with multi container

*(diagram: full stack — web pod [cache + webservice] on NodePort 30500/32500, maindb pod [redis + mysql])*

---

<!-- PDF p.107 -->
## Pods, Container and Services

- Check how to operate components on Kubernetes:

```bash
kubectl explain <Pods/SVC/etc>
kubectl api-versions
```

---

<!-- PDF p.108 -->
## Pods, Container and Services

- Check all components of this K8S version:

```bash
kubectl api-resources -o wide
```

---

<!-- PDF p.109 -->
## Pods, Container and Services

- Init Container
  - Runs "predefined" tasks before the main container starts
    - Wait for related components: `for i in {1..100}; do sleep 1; if nslookup myservice; then exit 0; fi; done; exit 1`
    - Prepare configuration: `git clone ... && sh init.sh`
  - If an init container fails, Kubernetes restarts it until it works
  - Multiple init containers run **sequentially**
  - Main containers do not start until init containers are done
  - Init containers do not support `lifecycle`, `livenessProbe`, `readinessProbe`, `startupProbe`

---

<!-- PDF p.110 -->
## Pods, Container and Services

*(YAML listing: pod with initContainers example)*

---

<!-- PDF p.111 -->
## Pods, Container and Services

- SideCar Containers (native since 1.29, **stable since 1.33**)
  - Use cases: Service Mesh (Kuma, Istio), Monitoring agents, Security enhancement
  - Defined under `initContainers` with special `restartPolicy: Always`
  - Supports `readinessProbe` to define pod-ready state
  - Also supports batch workloads
  - Difference from a plain extra container:
    - Runs alongside the main container but with independent lifecycle (update, patch)
    - Still shares resources with the main container (network, storage)

---

<!-- PDF p.112 -->
## Pods, Container and Services

*(YAML listing: sidecar container example)*

---

<!-- PDF p.113 -->
## Pods, Container and Services

- Services
  - Service is independent from Pods
    - Pods are created/destroyed/restarted all the time → their node/IP always changes
    - Service doesn't care how Pods change; it still maps to them
  - Service is an abstraction over Pods (1–N), selected by **Label**
  - Exposes/load-balances Pods (dataplane: kube-proxy **nftables**)
    - `port`: what the service opens · `targetPort`: container port · protocol TCP/UDP
  - Discovery options:
    - ENVIRONMENT: `{SVC_NAME}_SERVICE_HOST` / `{SVC_NAME}_SERVICE_PORT`
    - DNS (cluster addon): name = service name

---

<!-- PDF p.114 -->
## Pods, Container and Services

- Endpoint (Legacy) and EndpointSlices
  - Creating a Service with a selector auto-creates endpoint objects tracking matching Pods
  - **Endpoints (Legacy)**: single object as source of truth (1000 pods = 1 giant object)
  - **EndpointSlices**: endpoints grouped into small sets (default max 100 per slice) —
    modern and scalable; only the affected slice updates when pods change
  - Since 1.33 the Endpoints API is officially **deprecated** — use EndpointSlices

---

<!-- PDF p.115-116 -->
## Pods, Container and Services

*(diagrams: Endpoints vs EndpointSlices)*

Ref: https://kubernetes.io/blog/2025/04/24/endpoints-deprecation/

---

<!-- PDF p.117-119 -->
## Pods, Container and Services

- Services and label selectors
  - Pods "web" / Pods "web2" / Service "web"
  - Existing Pods "web" → replace with new Pods "web2": service keeps routing by label

*(YAML + terminal screenshots)*

---

<!-- PDF p.120 -->
## Pods, Container and Services

- Publish Service Types:
  - **ClusterIP** (default): binds port to cluster-internal IP (accessible inside cluster)
  - **NodePort**: binds port on every node IP. Default random range 30000–32767, or set `nodePort: XXXXX`

---

<!-- PDF p.121 -->
## Pods, Container and Services

- Services: ClusterIP

*(diagram: 3 nodes; SVC App1 → Pods App1-1/App1-2, SVC App2 → Pods App2-1/App2-2;
internal call by service name)*

---

<!-- PDF p.122 -->
## Pods, Container and Services

- Services: NodePort

*(diagram: every node listens 32311 → App1, 32312 → App2; client can hit any node IP)*

---

<!-- PDF p.123-124 -->
## Pods, Container and Services

- **LoadBalancer** type:
  - Uses external cloud provider load balancer to intercept traffic
  - Managed by Kubernetes itself; flexible for cloud facilities

*(diagram: AWS ELB DNS names → SVC App1/App2 → pods)*

---

<!-- PDF p.125-126 -->
## Pods, Container and Services

- **ExternalIP** type:
  - Similar to LoadBalancer but references an external LB by IP address
  - **Not** managed by Kubernetes itself

*(diagram: external LB 10.18.2.3/10.18.2.4 → services)*

---

<!-- PDF p.127-128 -->
## Pods, Container and Services

- **Headless Service** (for StatefulSet):
  - No cluster IP — DNS returns round-robin of all pod IPs

*(diagram: DNS record lists pod IPs directly)*

---

<!-- PDF p.129 -->
## Pods, Container and Services

- **ExternalName** type:
  - No selector / no pods / no endpoints
  - Returns a CNAME record for DNS load-balance
  - `nslookup my-service.prod.svc.cluster.local` → CNAME `my.database.example.com`

---

<!-- PDF p.130-131 -->
## Pods, Container and Services

- Services without Selector (no automatic EndpointSlice)
  - Use cases:
    - Call an external redis via a service name (proxy style)
    - Call a service across namespaces with a simple name
    - Call across clusters
  - You create the **EndpointSlice** yourself → fully customizable service

*(YAML listing: manual EndpointSlice)*

---

<!-- PDF p.132 -->
## Pods, Container and Services

- kube-proxy (proxy-mode)
  - Every node runs kube-proxy to realize Services (virtual IPs, except ExternalName)
  - **History — userspace mode** (default v1.0–1.7):
    - Opened random local port, proxied to backend pods in a userspace binary
    - Round robin / retry on pod failure — slow (kernel↔user switching)
  - *(removed long ago)*

---

<!-- PDF p.133-134 -->
## Pods, Container and Services

- kube-proxy (proxy-mode)
  - **iptables mode** (default v1.8–1.29 era):
    - iptables rules capture Service IP (VIP) traffic and redirect to pods
    - Kernel space — faster than userspace
    - Linux distributions have deprecated iptables (RHEL 9+ etc.)

Ref: https://access.redhat.com/documentation/.../deprecated-functionality

---

<!-- PDF p.135 — UPDATED for 1.36 -->
## Pods, Container and Services

- kube-proxy (proxy-mode)
  - **ipvs mode — REMOVED in Kubernetes 1.36** (deprecated in 1.35)
    - Was: L4 LB via IPVS kernel module with rr/wrr/sh algorithms
    - If your clusters still run IPVS you **must migrate to nftables before upgrading**
    - Migration: set `mode: nftables` in kube-proxy configuration

---

<!-- PDF p.136-137 -->
## Pods, Container and Services

- Configure kube-proxy (proxy-mode)

```yaml
# kube-proxy ConfigMap
mode: "nftables"
```

*(comparison chart: iptables vs nftables performance)*

Ref: https://www.tigera.io/blog/comparing-kube-proxy-modes-iptables-or-ipvs/

---

<!-- PDF p.138 — UPDATED for 1.36 -->
## Pods, Container and Services

- kube-proxy (proxy-mode)
  - **nftables mode** (GA since 1.33 — the standard mode in 1.36)
    - Replacement for legacy iptables/ipvs
    - Next-generation packet filtering: new codebase, faster rule evaluation, dual-stack IPv4/IPv6
    - Requires Linux kernel 3.13+
    - Resolves the iptables performance issues at scale (O(1) vs O(n) rule matching)

---

<!-- PDF p.139 -->
## Pods, Container and Services

- kube-proxy (proxy-mode)
  - **kernelspace mode** (since v1.14)
    - Only available on Windows nodes
    - Runs on Windows Virtual Filtering Platform (VFP) (VSwitch extension)

Ref: https://www.usenix.org/system/files/login/articles/login_fall17_02_firestone.pdf

---

<!-- PDF p.140 -->
## Pods, Container and Services

- Pods Priority and Preemption
  - Assign creation priority to pods: when resources are insufficient, Kubernetes runs
    higher-priority pods first and evicts lower-priority pods
  - Priority takes effect only when the pod is **created**
  - Disabling Preemption is not recommended

---

<!-- PDF p.141 -->
## Pods, Container and Services

- Pods Priority and Preemption
  - Kubernetes uses the **PriorityClass** object value for resource and quota consideration

*(YAML listing: PriorityClass + pod using priorityClassName)*

---

<!-- PDF p.142-143 -->
# Daemon Set and RC

*(section divider + concept diagram)*

---

<!-- PDF p.144 -->
## Daemon Set and RC

- What is a DaemonSet?
  - Basic pods and service can make a microservice up and running, but…
    - No maintenance of pod availability, no response to scaling
  - DaemonSet responsibilities:
    - Ensure Pods run on **all (or some) nodes** in the cluster
    - When a node joins, the Pod deploys automatically
    - When a node is removed, GC removes the Pod
    - Deleting the DaemonSet removes its Pods
  - General use cases:
    - Storage daemons (Ceph etc.), Node monitoring daemons (Prometheus agent etc.)

---

<!-- PDF p.145 -->
## Replication Controller (RC)

- What is RC?
  - DaemonSet and Replication Controller work similarly
  - RC responsibilities:
    - Create/Maintain Pods as a "pod farm"
    - Keep the designed number of replicas: too many → kill; too few → create
    - Auto-healing when Pods crash for any reason
    - Maintains at **cluster level**, not node level

---

<!-- PDF p.146 -->
## Replication Controller (RC)

- RC is the **first generation** module to keep Pods available in the cluster
- Use cases: rescheduling, scaling, rolling updates (complicated), map with service to manage releases
- RC selects pods with label type: "Equality-based requirement"
- Note: in modern clusters use **Deployment/ReplicaSet** — RC is legacy but still teachable

*(YAML listing: RC example)*

---

<!-- PDF p.147-148 -->
## Replication Controller (RC)

- Example: Pods "webtest" / RC "webtest" / SVC "webtest"
- Create rc "webtest", create svc "webtest"

*(YAML + terminal screenshots)*

---

<!-- PDF p.149-150 -->
## Replication Controller (RC)

- Test deleting some Pods from the command line
- Recheck Pod count and availability — RC recreates them
- Check detail of the create process on RC (`kubectl describe rc`)

---

<!-- PDF p.151 -->
## Replication Controller (RC)

- Scale up replicas on RC

```bash
kubectl scale --replicas=<N> <Type/Name>
```

---

<!-- PDF p.152-154 -->
## Replication Controller

- Check detail of create process on RC
- Test delete some Pods / recheck availability
- Cleanup Lab

---

<!-- PDF p.155 -->
## Workshop: RC

- Deploy replication controller (`WorkShop_1.3_Replication_Controller`)

---

<!-- PDF p.156-157 -->
# Deployment/RS and Update

---

<!-- PDF p.158 -->
## Deployment/RS and Update

- What is Deployment/RS?
  - Deployment + ReplicaSet = "next generation of RC" with full versioning of Pods in production (no downtime, on-the-fly):
    - Update new version (Rollout)
    - Revert old version (Rollback)
    - Pause/Resume process
    - Check status
  - Deployment orders the ReplicaSet (RS) to create Pods as designed
  - On rollout of a new version (automatic):
    - Create new RS and scale it up as desired
    - Scale existing RS down to 0, then delete it

---

<!-- PDF p.159 -->
## Deployment/RS and Update

- ReplicaSet (RS) vs Replication Controller (RC)
  - RS evolved from RC with more dynamic capability
  - RC supports labels with "Equality-based requirement"
  - RS supports both "Equality-based" **and** "Set-based requirement"

---

<!-- PDF p.160 -->
## Deployment/RS and Update

- Example images for rollout lab:
  - `labdocker/cluster:webservicelite_v1`
  - `labdocker/cluster:webservicelite_v1.51rc`
  - `labdocker/cluster:webservicelite_v1.8ga`

---

<!-- PDF p.161-162 -->
## Deployment/RS and Update

- Example: Deployment "webtest" / SVC "webtest"
- Create deployment "webtest" (and its ReplicaSet), create svc "webtest"

*(YAML + terminal screenshots)*

---

<!-- PDF p.163 -->
## Deployment/RS and Update

- Procedure of a deployment operation:
  - Check existing RS (0 when creating new)
  - Create new RS
  - Scale RS to designed replicas

---

<!-- PDF p.164 -->
## Deployment/RS and Update

- Deployment update strategy (Rolling Update)
  - Update triggers when anything under `spec.template` changes
  - Steps per change (default):
    - Create new RS for the changed template
    - Scale down existing RS (bounded by maxUnavailable)
    - Scale new RS up (no more than desired + maxSurge)
    - Delete existing RS
  - Defaults: 25% maxUnavailable, 25% maxSurge

---

<!-- PDF p.165-167 -->
## Deployment and change-cause tracking

- The old `--record` flag is **deprecated/removed** — use annotations instead:

```bash
kubectl annotate deployment/webtest kubernetes.io/change-cause="update to v1.51rc"
```

- `kubectl rollout history` shows the change-cause per revision

Ref: https://github.com/kubernetes/kubernetes/issues/40422#issuecomment-1426936343

---

<!-- PDF p.168 -->
## Deployment/RS and Update

- Let's talk about deployment strategies!
  - **Recreate** (for development): destroy all pods and recreate
  - **RollingUpdate** (default): create new pods, wait ready, then terminate old
  - **Blue/Green**: create new "green" deployment, re-point the service
  - **Canary**: new deployment with same label, fewer replicas at first, increase as needed
  - **A/B**: new deployment + service mesh routes selected traffic (header/IP/weight)

Ref: https://auth0.com/blog/deployment-strategies-in-kubernetes/

---

<!-- PDF p.169-173 -->
## Deployment/RS and Update

- Strategy diagrams:
  - Recreate *(all v1 down → all v2 up)*
  - RollingUpdate *(gradual v1→v2)*
  - Blue/Green *(two full stacks, service switch)*
  - Canary *(v1 majority + v2 minority behind same label)*
  - A/B *(service mesh filter: header / client IP / weight)*

---

<!-- PDF p.174 -->
## Deployment/RS and Update

- RollingUpdate under the hood:

*(diagram: `create -f` v1 → RS webtest-1x9 with 3 pods; `apply -f` v2 →
new RS webtest-1YU scales up while old RS scales down to 0)*

---

<!-- PDF p.175 -->
## Deployment/RS and Update

- How to do graceful rollout/rollback
  - Set the RollingUpdate strategy
  - Set `terminationGracePeriodSeconds` (after SIGTERM)
  - Set `readiness` probe and code the app to fail readiness on SIGTERM
  - Set `readinessGates` for external hook-back before pods open

```yaml
replicas: 3
strategy:
  type: RollingUpdate
  rollingUpdate:
    maxUnavailable: 50%
    maxSurge: 100%
spec:
  terminationGracePeriodSeconds: 30
```

---

<!-- PDF p.176 -->
## Deployment/RS and Update

- Define `revisionHistoryLimit` as designed (default: 10)

```yaml
replicas: 3
revisionHistoryLimit: 3
strategy:
  type: RollingUpdate
  rollingUpdate:
    maxUnavailable: 50%
    maxSurge: 100%
```

---

<!-- PDF p.177 -->
## Deployment/RS and Update

- Trigger change on Deployment (Rollout)
  - Method 1 — set online: `kubectl set image deployment/webtest webtest=<image>:betaversion`
  - Method 2 — edit online: `kubectl edit deployment/webtest`
  - Method 3 — edit YAML file and `kubectl apply -f <FileName>`

---

<!-- PDF p.178 -->
## Deployment/RS and Update

- Check status of deployment update (rollout):

```bash
kubectl rollout status  deployment/webtest
kubectl rollout history deployment/webtest
kubectl rollout undo    deployment/webtest --to-revision=2
kubectl rollout pause   deployment/webtest
kubectl rollout resume  deployment/webtest
```

---

<!-- PDF p.179-182 -->
## Deployment/RS and Update

- Set new image on deployment *(terminal screenshots)*
- Rollout History / Rollback *(terminal screenshots)*

---

<!-- PDF p.183 -->
## Workshop: Deployment

- `WorkShop_1.4_Deployment` — rollout v1 → v1.51rc → v1.8ga, rollback, pause/resume

---

<!-- PDF p.184 -->
# Volume

---

<!-- PDF p.185 -->
## Volume

- By default all containers in the same Pod can share storage (**emptyDir**)
  - Keeps all read/write for all containers in the Pod
  - Persistent for the containers as long as the Pod exists
  - Container crash does **not** affect this storage
  - When the Pod is deleted from the node, emptyDir is deleted forever
- Data in container/Pods is "ephemeral" — may be lost when a Pod restarts/moves
- Kubernetes supports many volume types

---

<!-- PDF p.186 — UPDATED for 1.36 -->
## Volume

- List of supported storage (in-tree):
  - Active: `emptyDir`, `hostPath`, `local`, `nfs`, `iscsi`, `fc`, `downwardAPI`,
    `configMap`, `secret`, `persistentVolumeClaim` (PVC — Day 2), `projected`
  - **NEW — `image` (OCI VolumeSource): stable in 1.36** — mount an OCI image as read-only volume
  - Removed/deprecated: `gitRepo` (**removed in 1.36**), `gcePersistentDisk` (removed 1.29),
    `awsElasticBlockStore`, `azureDisk`, `azureFile`, `cinder`, `cephfs`, `rbd`,
    `portworxVolume`, `vsphereVolume`, `flexVolume` → all replaced by **CSI drivers**

---

<!-- PDF p.187 -->
## Volume

- hostPath:
  - Mount file/directory from the host into the Pod
  - Recommended for Pods that must read host data (cAdvisor etc.)
  - Considerations:
    - Paths/files may differ between hosts
    - Scheduler cannot check readiness of hostPath
    - Some host paths need privilege to access — **security risk**, restricted under PSA baseline

---

<!-- PDF p.188-189 -->
## Volume

- hostPath example:
  - Container creates a file → host sees the file

*(YAML + terminal screenshots)*

---

<!-- PDF p.190 -->
## Volume

- **NEW in 1.36 — OCI image volume (stable)**

```yaml
volumes:
  - name: model-weights
    image:
      reference: registry.example/models/llm:v1
      pullPolicy: IfNotPresent
```

- Share files/tools/model weights among containers without rebuilding the app image

---

<!-- PDF p.191-192 -->
## Workshop: Volume

- `WorkShop_1.5_Volume` — emptyDir + hostPath labs

---

<!-- PDF p.193 -->
# Liveness, Readiness and Startup Probe

---

<!-- PDF p.194 -->
## Liveness, Readiness and Startup Probe

- Pods, are you still alright?
  - Kubernetes checks Pod status and applies the `restartPolicy`:
    - `Always` (default) / `OnFailure` / `Never`
  - Pod status depends on the `ContainerState` of its containers (1–M):
    - Waiting (ContainerStateWaiting)
    - Running (ContainerStateRunning)
    - Terminated (ContainerStateTerminated)

---

<!-- PDF p.195 -->
## Liveness, Readiness and Startup Probe

- Pod Phase values: Pending / Running / Succeeded / Failed / Unknown
- Do we need more specific health-checks?
  - Depends on the application's failure condition
  - If failure crashes the process → container goes down → no extra check needed
  - If failure happens **inside** the app while the process stays online → needs a real health check

---

<!-- PDF p.196 -->
## Liveness, Readiness and Startup Probe

- Container probe mechanisms:
  - **ExecAction**: execute a command inside the container (expect exit 0)
  - **gRPC**: remote procedure call; target implements gRPC health check (expect `SERVING`)
  - **TCPSocketAction**: open TCP connection on a container port (expect port open)
  - **HTTPGetAction**: send HTTP(S) request to port+path (expect 2xx)

Ref: https://kubernetes.io/docs/concepts/workloads/pods/pod-lifecycle/

---

<!-- PDF p.197-198 -->
## Liveness, Readiness and Startup Probe

*(diagrams: gRPC health checking, probe decision flow)*

Ref: https://kubernetes.io/blog/2018/10/01/health-checking-grpc-servers-on-kubernetes/

---

<!-- PDF p.199 -->
## Liveness and Readiness Probe

- Probe results: Success / Failed / Unknown
- Differences:
  - **livenessProbe**: is the container running properly?
  - **readinessProbe**: is it ready to serve traffic?
  - **startupProbe**: has the app started? (all other probes are disabled until it succeeds;
    if it fails, kubelet kills the container per restart policy; default state "Success" if absent)

---

<!-- PDF p.200 -->
## Liveness and Readiness Probe

- Probe configuration:
  - `initialDelaySeconds` / `periodSeconds` / `timeoutSeconds`
  - `successThreshold` / `failureThreshold`
  - `terminationGracePeriodSeconds`

---

<!-- PDF p.201 -->
## Liveness and Readiness Probe

- Pod Conditions:
  - **PodScheduled**: the Pod has been scheduled to a node
  - **PodReadyToStartContainers**: sandbox created and networking configured (stable)
  - **Initialized**: all init containers completed successfully
  - **ContainersReady**: all containers in the Pod are ready
  - **Ready**: Pod can serve requests; added to matching Services' load-balancing pools

---

<!-- PDF p.202 -->
## Liveness and Readiness Probe

- Container Lifecycle Hooks (Exec, HTTP, Sleep)
  - **PostStart**: runs immediately when container is created
    (no guarantee it executes before ENTRYPOINT)
  - **PreStop**: runs immediately before container terminates (delete, liveness fail) —
    blocking before termination

---

<!-- PDF p.203-204 -->
## Liveness and Readiness Probe

- Lifecycle hook: **Sleep action** (stable — zero-value sleep since 1.34)

```yaml
lifecycle:
  preStop:
    sleep:
      seconds: 5
```

- Cleaner graceful shutdown without a shell `sleep` binary in the image

Ref: https://github.com/kubernetes/enhancements/tree/master/keps/sig-node/3960-pod-lifecycle-sleep-action

---

<!-- PDF p.205-208 -->
## Liveness and Readiness Probe

- **ExecAction** examples

*(YAML: livenessProbe exec `cat /tmp/healthy` + terminal screenshots of failure/restart)*

---

<!-- PDF p.209-212 -->
## Liveness and Readiness Probe

- **TCPSocketAction** examples

*(YAML: readiness/liveness tcpSocket port 8080 + terminal screenshots)*

---

<!-- PDF p.213-218 -->
## Liveness and Readiness Probe

- **HTTPGetAction** examples

*(YAML: httpGet path /healthz port 8080, headers; screenshots of 200 vs 500 behavior)*

---

<!-- PDF p.219 -->
## Liveness and Readiness Probe

- **ReadinessGate**: external condition injected into Pod readiness

*(YAML: readinessGates conditionType example)*

---

<!-- PDF p.220 -->
## Liveness and Readiness Probe

- **gRPCAction**:

```yaml
livenessProbe:
  grpc:
    port: 2379
```

---

<!-- PDF p.221 -->
## Workshop: Healthcheck Probe

- `WorkShop_1.6_Liveness_Readiness_Probe`

---

<!-- PDF p.222 -->
# Resource Management and HPA

---

<!-- PDF p.223 -->
## Resource Management and HPA

*(diagram: resource cheat-sheet)*

Ref: http://k8s.info/cs.html

---

<!-- PDF p.224 -->
## Resource Management and HPA

- Kubernetes manages resources in 2 ways:
  - **Container level**: scope per container, independent
  - **Namespace level**: global limits assigned to a namespace, applied to any Pods
- What happens when a container exceeds its resources?
  - **CPU**: throttled — free CPU slots distributed to requests by ratio
  - **Memory**: container is killed and restarted (OOMKill)
  - **Ephemeral-storage** (emptyDir, configmap, CSI ephemeral): eviction → new pod
  - **Storage**: cannot use more space (behavior depends on application)

---

<!-- PDF p.225-228 -->
## Resource Management and HPA

- CPU Limit and Throttling
  - `cpu.cfs_period_us` (default): 100 ms (100%)
  - `limit.cpu = 500m` → 50 ms per period (50%) → `cpu.cfs_quota_us = 50,000`

*(diagrams: throttling timeline, latency impact)*

Ref: https://www.ibm.com/blog/kubernetes-cpu-throttling-the-silent-killer-of-response-time/

---

<!-- PDF p.229 -->
## Resource Management and HPA

- Memory Limit and OOM Kill

*(diagram: memory usage crossing limit → OOMKilled)*

Ref: https://sysdig.com/blog/troubleshoot-kubernetes-oom/

---

<!-- PDF p.230 -->
## Resource Management and HPA

- Container level configuration
  - **CPU** — Request: `XXXm` millicores (0.1 = 100m; 1 CPU = 1 vCPU AWS / 1 GCP core / 1 Azure vCore / 1 hyper-thread) ≈ `--cpu-share`
  - **CPU** — Limit: `XXXm` millicores ≈ `--cpu-quota`
  - **Memory** — Request/Limit: `Ki, Mi, Gi, Pi, Ei`

Ref: https://github.com/kubernetes/community/blob/master/contributors/design-proposals/resource-qos.md

---

<!-- PDF p.231-232 -->
## Resource Management and HPA

- Swap support (GA since 1.34, `LimitedSwap` mode)
  - kubelet: `swapBehavior: LimitedSwap` — Burstable pods may swap within limits
  - Requires cgroups v2 (which is mandatory in 1.36 anyway)

Ref: https://kubernetes.io/docs/concepts/cluster-administration/swap-memory-management/

---

<!-- PDF p.233-236 -->
## Resource Management and HPA

- Container level configuration in action:
  - Pods "webtest" + system status
  - Allocate status on node (`kubectl describe node`)
  - Create load on CPU (T1, T2) and watch throttling

*(terminal screenshots)*

---

<!-- PDF p.237 -->
## Workshop: Resource MNG/HPA

- Part 1: Container level configuration (`WorkShop_1.7_Resource_Management`)

---

<!-- PDF p.238 -->
## Resource Management and HPA

- Namespace level configuration
  - Namespace = collection of cluster resources, easy to define and apply to objects
  - Applies to Pods, Services, RC, Deployments/RS, Quota, LimitRange, etc.
  - For namespace-level resource management we use:
    - **ResourceQuota**: summary/total resource control per namespace
    - **LimitRange**: admission control per object / default resource values

---

<!-- PDF p.239 -->
## Resource Management and HPA

- Quota — compute resources:
  - CPU (limits / requests)
  - Memory (limits / requests)
  - Ephemeral-storage (limits / requests)
  - Storage (+ per StorageClass):
    - `<storage-class>.storageclass.storage.k8s.io/requests.storage`
    - `<storage-class>.storageclass.storage.k8s.io/persistentvolumeclaims`

Ref: https://kubernetes.io/docs/concepts/policy/resource-quotas

---

<!-- PDF p.240 -->
## Resource Management and HPA

- Quota — object counting:
  - pods, services, services.loadbalancers, services.nodeports
  - replicationcontrollers, resourcequotas, configmaps, secrets

---

<!-- PDF p.241-242 -->
## Resource Management and HPA

- Quota per PriorityClass

*(YAML: scopeSelector with priorityClass example)*

Ref: https://kubernetes.io/docs/concepts/policy/resource-quotas

---

<!-- PDF p.243-244 -->
## Resource Management and HPA

- LimitRange (type: Pod/Container)
  - **Max / Min**: CPU, Memory, Ephemeral-storage, PersistentVolumeClaim
  - **Default** (limit default for container): CPU, Memory, Ephemeral-storage
  - **DefaultRequest**: CPU, Memory, Ephemeral-storage

Ref: https://kubernetes.io/docs/tasks/administer-cluster/apply-resource-quota-limit/

---

<!-- PDF p.245 -->
## Resource Management and HPA

- Create Namespace, assign quota:

```bash
kubectl create namespace <name>
kubectl create -f <Quota File> --namespace <name>
```

---

<!-- PDF p.246-253 -->
## Resource Management and HPA

- Lab flow (namespace level):
  - Create Deployment in namespace → **fails**: `kubectl describe rs --namespace <name>` (no default requests)
  - Create LimitRange → recreate Deployment → succeeds
  - `kubectl describe pods --namespace <name>` shows defaulted requests/limits
  - Burn CPU and monitor; edit deployment to update cpu/memory limits & requests

*(terminal screenshots per step)*

---

<!-- PDF p.254 -->
## Workshop: Resource MNG/HPA

- Part 2: Namespace level configuration

---

<!-- PDF p.255 -->
## Resource Management and HPA

- QoS for resource management — 3 classes:
  - **Guaranteed**: all containers have memory & CPU limit = request (fully reserved)
  - **Burstable**: not Guaranteed, but at least one container has a memory or CPU request — extendable if resources available
  - **BestEffort**: no memory or CPU limits/requests at all

---

<!-- PDF p.256 -->
## Workshop: Resource MNG/HPA

- Part 3: QoS POC

---

<!-- PDF p.257 -->
## Resource Management and HPA

- Horizontal Pod Autoscaling (HPA)
  - Manual scale is easy: `kubectl scale --replicas=XXX deployment/<name>`
  - But… how do you scale to match actual demand?
  - HPA monitors workload (CPU/Memory) and triggers the deployment to scale
  - Since 1.30: HPA can target **per-container** resource metrics

```bash
kubectl autoscale deployment/webtest --min=1 --max=10 --cpu-percent=80
```

---

<!-- PDF p.258 -->
## Resource Management and HPA

- HPA controller timing flags:
  - `--horizontal-pod-autoscaler-initial-readiness-delay` (default 30s)
  - `--horizontal-pod-autoscaler-cpu-initialization-period` (default 300s)
  - `--horizontal-pod-autoscaler-downscale-stabilization` (default 300s)

---

<!-- PDF p.259 -->
## Resource Management and HPA

- HPA scaling algorithm: `desired = ceil(current × currentMetric / targetMetric)`
  - Case 1: 1 pod, current 100m, target 200m → (100/200)=0.5 → ceil → 1 pod
  - Case 2: 3 pods, current 300m, target 200m → (300/200)=1.5 → ceil(1.5)=2 → 6 pods

Ref: https://kubernetes.io/docs/tasks/run-application/horizontal-pod-autoscale/

---

<!-- PDF p.260-262 -->
## Resource Management and HPA

- HPA configurable tolerance (alpha since 1.33)
  - Default tolerance 10% around target before scaling
  - `spec.behavior`: shape scale-down (policies/period), limit scale-down rate, or disable

*(YAML: behavior.scaleDown policies example)*

---

<!-- PDF p.263 — NEW for 1.36 -->
## Resource Management and HPA

- **NEW: In-place Pod resource resize (GA since 1.35)**
  - Change a running Pod's CPU/memory without restart:

```bash
kubectl patch pod webtest --subresource=resize \
  --patch '{"spec":{"containers":[{"name":"webtest","resources":{"limits":{"cpu":"800m"}}}]}}'
```

  - 1.36 extends toward pod-level vertical scaling
  - Vertical + horizontal scaling can now work together

---

<!-- PDF p.264-270 -->
## Resource Management and HPA

- HPA lab flow:
  - Apply HPA (target CPU 10%)
  - Generate load with busybox (wget every 10 ms)
  - Load increases → HPA scales out (check interval loop)
  - Scale-out continues until target is met
  - Stop load → HPA scales down (after stabilization window)

*(terminal screenshots per step)*

---

<!-- PDF p.271 -->
# ConfigMap and Secret

---

<!-- PDF p.272 -->
## ConfigMap and Secret

- Make secret data and configuration great again!
- Many containers need configuration/sensitive data — should it live in the image or object spec?
  - Root password of database, environment variables, custom variables, volume paths…
- **ConfigMap**: central configuration for Pods
- **Secret**: encoded sensitive data
- Both support `immutable: true` to protect against accidental updates (since 1.21)

---

<!-- PDF p.273 -->
## ConfigMap and Secret

- ConfigMap
  - Namespace-scoped
  - Ways to create:
    - literal values: `kubectl create configmap webmodule_configmap --from-literal=REDIS_HOST=localhost`
    - from file or folder
    - YAML file: `kubectl create -f webmodule_configmap.yml`

---

<!-- PDF p.274 -->
## ConfigMap and Secret

- Secret
  - Values are **base64-encoded** (encoding, not encryption!)
  - Ways to create:
    - from files: `kubectl create secret generic databasemodule_secret --from-file=./username.txt --from-file=./password.txt`
    - YAML file: `kubectl create -f databasemodule_secret.yml`

---

<!-- PDF p.275 -->
## ConfigMap and Secret

*(YAML listings: configmap + secret consumed as env and as volume)*

---

<!-- PDF p.276-277 -->
## ConfigMap and Secret

- External Secrets Operator (CNCF project)
  - Sync secrets from 3rd-party secret managers (HashiCorp Vault, AWS Secrets Manager, Azure Key Vault…) and inject as Kubernetes Secrets
  - Benefits:
    - Separates secret management from Kubernetes (segregation of duty)
    - Standard Secret object format for workloads
    - Automatic sync on interval / on update

Ref: https://external-secrets.io/latest/

---

<!-- PDF p.278-281 -->
## ConfigMap and Secret

- Lab architecture (refactor of Pods & Service lab):
  - `webmodule_deploy_config.yml` + `webmodule_configmap.yml`
  - `databasemodule_deploy_config.yml` + `databasemodule_secret.yml`

*(diagrams: web pod [cache:80 + webservice:5000] & maindb pod [redis:6379 + mysql:3306], now consuming ConfigMap + Secret)*

---

<!-- PDF p.282 -->
## Workshop: ConfigMap and Secret

- `WorkShop_1.8_ConfigMap_Secret`

---

<!-- PDF p.283 -->
## Recap Part 1

- Container concept (Recap)
- Introduction to Kubernetes
- System Architecture
- Fundamental of Kubernetes
  - Pods, Container and Services
  - Replication Controller (RC)
  - Deployment/Replica-Set (RS) and Rolling update
  - Volume
  - Liveness and Readiness Probe
  - Resource Management and HPA (+ in-place resize)
  - ConfigMap Secret

---

<!-- PDF p.284-287 -->
# Part 2

## Outline Part 2

- Fundamental of Kubernetes: Job and CronJob / Log and Monitoring
- **Gateway API Networking** (NGINX Ingress retired → Apache APISIX)
- Security on Kubernetes (Network/Volume/Resource/Access/Security policies, Encryption Provider, **Mutating Admission Policies**, **User Namespaces**)
- Kubernetes in real world (bare metal cluster setup, orchestrator assignment)
- Stateful application deployment (PV / PVC / StatefulSet, **OCI VolumeSource**)

Question & Answer Section — By: Praparn L (eva10409@gmail.com)

---

<!-- PDF p.288 -->
# Job and Cron Jobs

---

<!-- PDF p.289 -->
## Job and Cron Job

- Some tasks on kubernetes are batch / non-interactive:
  - Update EOD process, monitor system health, calculate balance, reindex file/database…
- "Job" is designed for this special purpose:
  - Tracks status of completed pods
  - Auto-starts new Pods when they fail or are deleted
  - Deletes Pods when the Job is deleted

```bash
kubectl create -f <YAML File>
```

---

<!-- PDF p.290 -->
## Job and Cron Job

- Parallel execution for Jobs
  - **Non-parallel Jobs** (default): one Pod; Job completes when it terminates successfully
  - **Parallel Jobs with fixed completion count**:
    - `completions: xx` — total successful pods required
    - `completionMode: "Indexed"` — gives each pod an index identity
  - **Parallel Jobs with a work queue**:
    - `parallelism: xx` — pods coordinate among themselves or via an external service

---

<!-- PDF p.291 -->
## Job and Cron Job

- Parallel Jobs with a work queue (continued):
  - When any Pod terminates with success, no new Pods are created
  - Job completes when at least one succeeded and all pods are terminated
  - Once one Pod exits successfully, the others should be exiting too
- `podReplacementPolicy` (stable since 1.34):
  - `podReplacementPolicy: Failed` — new pods are created only when existing pods are **fully terminated** (not while terminating)

---

<!-- PDF p.292-293 -->
## Job and Cron Job

*(diagrams: job pod lifecycle, completions vs parallelism)*

Ref: https://wangwei1237.github.io/Kubernetes-in-Action-Second-Edition/docs/Running_tasks_with_the_Job_resource.html

---

<!-- PDF p.294 -->
## Job and Cron Job

- "CronJob" is a time-based "Job":
  - Runs at a specific point in time
  - Repeats on schedule (standard cron syntax)

*(YAML: CronJob example)*

---

<!-- PDF p.295-296 -->
## Job and Cron Job

- WorkFlow Job — beyond plain Jobs: **Argo Workflows**

*(diagrams: DAG workflow of jobs)*

Ref: https://github.com/argoproj/argo-workflows

---

<!-- PDF p.297 -->
## Workshop: Job and CronJob

- Task: Parallel job with Redis (`WorkShop_2.1_Job_CronJob`)

---

<!-- PDF p.298 -->
# Debug Log and Monitoring

---

<!-- PDF p.299 -->
## Debug Log and Monitoring

- Kubernetes provides log tooling at Pod/container level:

```bash
kubectl logs pods/<pods name> -c <container name> <option>
```

  - `-p, --previous` — print last failed container in the Pod
  - `-f, --follow` — stream log
  - `-c, --container` — specific container
  - `-l, --selector` — filter by label
  - Ex: `kubectl logs -f pods/maindb -c maindb`

---

<!-- PDF p.300 -->
## Debug Log and Monitoring

- Log Rotation (since 1.21): tune kubelet:
  - `containerLogMaxFiles`: max number of log files
  - `containerLogMaxSize`: max size of each log

Ref: https://kubernetes.io/docs/reference/config-api/kubelet-config.v1beta1/

---

<!-- PDF p.301-302 -->
## Debug Log and Monitoring

- Monitoring kubernetes via Dashboard

*(screenshots: kubernetes-dashboard workload views)*

---

<!-- PDF p.303 -->
## Debug Log and Monitoring

- Ephemeral Container (GA since 1.25):
  - Debug an application **inside a running Pod**
  - Run a debugger container in the pod's namespaces:

```bash
kubectl debug -it <pod> --image=busybox --target=<container>
```

---

<!-- PDF p.304 -->
## Debug Log and Monitoring

*(terminal screenshots: kubectl debug session)*

---

<!-- PDF p.305 -->
## Workshop: Log and Monitoring

- `WorkShop_2.2_Log_and_Monitoring`

---

<!-- PDF p.306 — CHAPTER REBUILT FOR 1.36: Ingress (legacy concept) → Gateway API + APISIX -->
# Ingress → Gateway API Networking

**NGINX Ingress Controller retired 24 March 2026 — this chapter now teaches the Gateway API standard with Apache APISIX**

---

<!-- PDF p.307 -->
## Ingress Network (concept — still relevant)

- Remember Service?
  - Service exposes Pods internally/externally, but it is limited and inflexible:
    - How to handle multiple services on the same port?
    - How to limit protocol (HTTP/HTTPS)? How to bind hostnames (www.xxx.yyy)?
    - How to do TLS termination?
- **Ingress** was the L7 answer: host/path routing rules implemented by a controller (selected via ingress class)
- The Ingress API still works in 1.36, but it is **frozen** — new work happens in **Gateway API**

---

<!-- PDF p.308-310 -->
## Ingress Network

*(diagrams:
- Service-only access: client → NodePort per service
- Ingress access: client → Ingress → services webtest1/webtest2)*

---

<!-- PDF p.311-314 -->
## Ingress Network

- Ingress resource: host/path rules → backend services

*(YAML: ingress `ingresswebtest` routing webtest1/webtest2 by host)*

---

<!-- PDF p.315 -->
## Ingress Network

- Path Types:
  - **Prefix**: match based on URL path prefix (case sensitive)
  - **Exact**: match path exactly, case sensitive
  - **ImplementationSpecific**: defer to the controller/ingress class
- Multiple match precedence: longest path first; Exact > Prefix

---

<!-- PDF p.316-317 -->
## Ingress Network

- Multiple controllers per cluster: each controller owns an **IngressClass** (category)

*(diagram/YAML: IngressClass objects)*

---

<!-- PDF p.318-320 -->
## Ingress Network (TLS)

- TLS termination at the edge:
  - `https://webtest1.kuberneteslabthailand.com` → webtest1 (service)
  - `https://webtest2.kuberneteslabthailand.com` → webtest2 (service)
- Certificates stored as Secrets (`ingress_webtest_tls_secret_webtest1/2.yml`)

*(diagram + YAML)*

---

<!-- PDF p.321-324 -->
## Edge exposure for On-prem strategy

- **MetalLB** (L2 load balance) — gives `LoadBalancer` services a real IP on-prem
- **NodePort** + external load balancer
- **Host network** (security concern, not recommended)

*(diagrams per option — these strategies apply the same to the APISIX gateway service)*

---

<!-- PDF p.325-327 -->
## Edge exposure for Cloud strategy

- Example AWS:
  - Tag resources `kubernetes.io/cluster/<Cluster Name>`: EC2, VPC, Subnet, Routing Table
  - **AWS Load Balancer Controller** provisions NLB/ALB for `LoadBalancer` services
  - In this course: **APISIX gateway service + AWS NLB** (replaces the old NGINX Ingress AWS deploy)

Ref: https://github.com/kubernetes-sigs/aws-load-balancer-controller

---

<!-- PDF p.329 -->
## Gateway API

- **Gateway API = Ingress + Load Balance + Service Mesh**
- Gateway API is the next generation of Ingress
- Focus on L4 & L7 network routing on Kubernetes
- Role-oriented personas: Infrastructure Provider (Ian) / Cluster Operator (Chihiro) / Application Developer (Ana)

---

<!-- PDF p.330 -->
## Gateway API

- What improves over Ingress?
  - **Role-oriented**: persona-based resource split
  - **Portable**: many implementations of the same API
  - **Expressive**: header matching, traffic weighting — no more controller-specific annotations
  - **Extensible**: typed extension points
- Special improvements:
  - GatewayClasses
  - Shared Gateways and cross-namespace routing (**ReferenceGrant**)
  - Typed routes (HTTPRoute, GRPCRoute, TLSRoute…) and typed backends
  - Service mesh support (GAMMA)

---

<!-- PDF p.331 -->
## Gateway API

- Difference between Gateway API and API Gateway?
  - **API Gateway** (general concept): anything exposing backend capabilities with traffic routing/manipulation — LB, transformation, authn/authz, rate limiting, circuit breaking
  - **Gateway API** (Kubernetes interface): a set of resources modeling service networking; the main resource is a **Gateway**, declaring the gateway class to instantiate and its configuration

Ref: https://gateway-api.sigs.k8s.io/

---

<!-- PDF p.332 -->
## Gateway API — resource model

*(diagram: GatewayClass → Gateway (listeners HTTP/HTTPS) → HTTPRoute → Service → Pods,
with ReferenceGrant allowing the Gateway in ns "apisix" to read TLS secrets in ns "default")*

---

<!-- NEW slides for 1.36 rebuild: APISIX implementation -->
## Gateway API with Apache APISIX

- **Apache APISIX**: high-performance, open-source API gateway (etcd-backed, plugin-rich)
- APISIX Ingress Controller implements the **Gateway API standard**
- Install (Workshop 2.3):

```bash
kubectl apply -f https://github.com/kubernetes-sigs/gateway-api/releases/download/v1.3.0/standard-install.yaml
helm repo add apisix https://apache.github.io/apisix-helm-chart
helm install apisix apisix/apisix -n apisix --create-namespace \
  -f ~/kubernetes_202607/WorkShop_2.3_Ingress_Network/apisix/apisix-values.yml
```

- Record the NodePort of `apisix-gateway` service (80:`<NodePort>` / 443:`<NodePort>`)

---

## Gateway API with Apache APISIX

- Core objects in the lab (`WorkShop_2.3_Ingress_Network/apisix/`):
  - `gatewayclass-gateway.yml` — GatewayClass + Gateway (HTTP + per-host HTTPS listeners)
  - `referencegrant-tls-secrets.yml` — lets the Gateway (ns `apisix`) use TLS secrets in `default`
  - `apisix-httproute-webtest1/2.yml` — HTTPRoutes for host-based routing
- Verify:

```bash
kubectl get gatewayclass
kubectl describe gateway/apisix -n apisix   # listeners "Programmed: True"
kubectl get httproute
```

---

## Gateway API with Apache APISIX — HTTP lab

- Route by Host header:

```bash
curl http://<Public IP>:<NodePort> -H 'Host:webtest1.kuberneteslabthailand.com'
curl http://<Public IP>:<NodePort> -H 'Host:webtest2.kuberneteslabthailand.com'
```

- Or add `/etc/hosts` entries and browse:
  `http://webtest1.kuberneteslabthailand.com:<NodePort>`

---

## Gateway API with Apache APISIX — TLS lab

- TLS certs loaded onto the Gateway's HTTPS listeners (per-host)
- HTTPS listeners reference Secrets across namespaces via **ReferenceGrant**
- Test:

```bash
curl -k https://webtest1.kuberneteslabthailand.com:<TLS NodePort>
```

---

## Gateway API with Apache APISIX — key-auth lab

- APISIX plugin ecosystem via CRDs:
  - `apisix-jarvis-consumer.yml` — **Consumer** with key-auth credential (from `apisix-jarvis-secret.yml`)
  - `apisix-jarvis-pluginconfig.yml` — **PluginConfig** enabling `key-auth` on a route
  - `apisix-httproute-webtest1-auth.yml` / `webtest2-auth.yml` — HTTPRoutes with auth filter
- Test:

```bash
curl http://<IP>:<NodePort> -H 'Host:webtest1.kuberneteslabthailand.com'            # 401
curl http://<IP>:<NodePort> -H 'Host:webtest1.kuberneteslabthailand.com' \
     -H 'apikey: <key>'                                                             # 200
```

---

<!-- PDF p.328, 333 -->
## Workshop: Gateway API Network

- `WorkShop_2.3_Ingress_Network` — HTTP routing, TLS, key-auth (PluginConfig/Consumer)

*(diagram: client → APISIX Gateway (NodePort) → HTTPRoute → webtest1/webtest2 services)*

---

<!-- PDF p.334 -->
# Security

---

<!-- PDF p.335 — UPDATED for 1.36 -->
## Security

- Kubernetes has several security enhancements for the farm:
  - **Network Policy**: allow/deny connection between endpoints
  - **Volume Policy**: control/limit total volumes attached per node
  - **Resource Usage Policy**: acceptable range per namespace (LimitRange)
  - **Resource Consumption Policy**: max consumption per namespace (Quota)
  - **Access Control Policy**: fine-grained permission via RBAC
  - **Pod Security Admission (PSA)**: security context enforcement
  - **Encryption Provider**: make Secret great again
  - **NEW 1.36 — Mutating Admission Policies (GA)**: CEL-based native mutation
  - **NEW 1.36 — User Namespaces (GA)**: container root ≠ host root

---

<!-- PDF p.336 -->
## Security

- Network Policy:
  - Within the same namespace: call by `<service>` or `<service>.<namespace>`
  - Across namespaces: call by `<service>.<namespace>`

---

<!-- PDF p.337 -->
## Security

- Network Policy:
  - By default, pods are **non-isolated**: they accept traffic from any source
  - Policies apply to targets by `podSelector`, configured by label
  - **Ingress**: control incoming traffic — `ingress: - from: <podSelector>/<namespaceSelector>`
  - **Egress**: control outgoing traffic — `egress: - to: <podSelector>/<namespaceSelector>`
  - Selector types:
    - `podSelector`: pods in/out same namespace (by label)
    - `namespaceSelector`: pods in/out between namespaces (by label)
    - `ipBlock`: allow/deny by IP range

---

<!-- PDF p.338-339 -->
## Security

*(YAML: NetworkPolicy examples — default-deny-all, allow-frontend)*

---

<!-- PDF p.340-346 -->
## Security — Network Policy lab ("Stars" demo)

- Architecture:
  - ns `management-ui` (role=management-ui) — NodePort 32500 → port 9001
  - ns `client` (role=client) — ClusterIP 9000
  - ns `stars`: frontend (port 80) + backend (port 6379)
- Lab progression *(diagrams per step)*:
  1. No policy: everything can talk to everything
  2. `policy-default-deny-all` → all blocked (X)
  3. Allow UI → frontend/backend/client visibility restored selectively (Y)
  4. `policy-allow-frontend` → client → frontend only; frontend → backend only

---

<!-- PDF p.347 -->
## Security

- Network Policy: final state — least-privilege traffic graph

*(diagram: allowed edges only)*

---

<!-- PDF p.348 -->
## Workshop: Security

- Part: Network Policy (`WorkShop_2.4_Security` — policy-*.yml)

---

<!-- PDF p.349 -->
## Security

- Volume Policy:
  - Kubernetes has a default limit of volumes attached per host
  - Customize via environment variable: `KUBE_MAX_PD_VOLS`

Ref: https://kubernetes.io/docs/concepts/storage/storage-limits/

---

<!-- PDF p.350 -->
## Security

- Resource Usage Policy: controlled by **ResourceQuota**
- Resource Consume Policy: controlled by **LimitRange**

---

<!-- PDF p.351 -->
## Security

- Access Control Policy
  - Kubernetes manages privileges via **RBAC** (role-based access control)
  - Accounts: **User Account / Service Account**
  - RBAC has 4 categories:
    - Role and ClusterRole
    - RoleBinding and ClusterRoleBinding
    - Referring to Resources
    - Aggregated ClusterRoles

---

<!-- PDF p.352-353 -->
## Security

- Account Management
  - User accounts are for humans; service accounts are for processes running in pods
  - User accounts are global scope — names unique across all namespaces
  - Auditing considerations differ for humans vs service accounts
  - Supports OpenID Connect — and since 1.34+, **Structured Authentication Configuration** (GA) allows multiple OIDC providers / CEL claim mapping

Ref: https://kubernetes.io/docs/reference/access-authn-authz/authentication/

---

<!-- PDF p.354 -->
## Security

- Account Management
  - Service Accounts: YAML-managed, token projected into pods
  - User Accounts:
    - No YAML object — created via openssl/kubectl:
      - Private key + CSR (openssl)
      - Sign certificate with Kubernetes CA
      - Create kubectl context for the user

---

<!-- PDF p.355 -->
## Security

- RBAC Operate — Role and ClusterRole
  - **Role**: allow actions at namespace level
  - **ClusterRole**: allow actions at cluster level
  - Syntax structure — `rules`:
    - `apiGroups: [""]` → core API group
    - `resources: [""]` → kind of resource to manage
    - `verbs: [""]` → actions to allow
  - Default ClusterRoles: see bootstrap policy in kubernetes repo

---

<!-- PDF p.356-358 -->
## Security

*(YAML: role, clusterrole, serviceaccount examples from `WorkShop_2.4_Security/security-*.yml`)*

---

<!-- PDF p.359 -->
## Security

- RBAC Operate — RoleBinding and ClusterRoleBinding:
  - Bind user accounts / service accounts to a Role or ClusterRole

---

<!-- PDF p.360 -->
## Security

- RBAC Operate — Referring to Resources:
  - Identify a **subresource** under a main resource (ex: `pods/log`)
  - Identify a **specific resource** by name (`resourceNames`)

---

<!-- PDF p.361 -->
## Security

- RBAC Operate — Aggregated ClusterRoles:
  - Combine multiple cluster roles
  - Match by label; rules are automatically filled in

---

<!-- PDF p.362 -->
## Workshop: Security

- Part: Access Control Policy (RBAC labs in `WorkShop_2.4_Security`)

---

<!-- PDF p.363 -->
## Security

- Pod Security Admission (PSA):
  - Next generation of Pod Security Policy (PSP removed in 1.25)
  - Enforcement at cluster/namespace/pod level, 3 **modes**:
    - **enforce**: violations reject the pod
    - **audit**: violations add audit annotations, but allowed
    - **warn**: violations trigger user-facing warning, but allowed
  - 3 **profiles** per mode:
    - **privileged**: unrestricted
    - **baseline**: minimally restrictive, prevents known privilege escalations
    - **restricted**: heavily restricted, current Pod hardening best practices

---

<!-- PDF p.364 -->
## Security

- Privileged Profile:
  - Unrestricted at all!
  - Aimed at infrastructure/system namespaces
  - Managed by trusted users

---

<!-- PDF p.365 -->
## Security

- Baseline Profile:
  - For common containerized workloads; prevents known privilege escalation
  - Target: general application deployment, non-critical systems
  - Allows Windows privileges for Windows pods (NT AUTHORITY\SYSTEM / LocalService / NetworkService)

Ref: https://kubernetes.io/docs/concepts/security/pod-security-standards/

---

<!-- PDF p.366-370 -->
## Security

- Baseline Profile — control table:

*(tables: HostProcess, host namespaces, privileged containers, capabilities, hostPath volumes, hostPorts, AppArmor, SELinux, /proc mount, seccomp, sysctls)*

---

<!-- PDF p.371 -->
## Security

- Restricted Profile:
  - Hardest security profile — low-trust user environments, "critical" systems
  - Based on baseline profile with more enhancements

---

<!-- PDF p.372-374 -->
## Security

- Restricted Profile — control table:

*(tables: volume types whitelist, runAsNonRoot, runAsUser, seccompProfile, capabilities drop ALL)*

---

<!-- PDF p.375 -->
## Security

- Pod security standard for cluster/namespace:
  - Customize per mode: `enforce` / `audit` / `warn` = baseline|restricted|privileged
  - Enforce PSA for a new namespace via labels:

```yaml
metadata:
  labels:
    pod-security.kubernetes.io/enforce: restricted
```

---

<!-- PDF p.376-377 -->
## Security

- Namespace Level: enforce PSA on existing namespace (`kubectl label ns ...`)
- Cluster Level: enforce PSA on all namespaces (AdmissionConfiguration for the API server)

---

<!-- PDF p.378 -->
## Security

- Pod security standard for pods/containers (securityContext):
  - Permission to access objects (user/group id)
  - SELinux / AppArmor / Seccomp
  - Privileged / non-privileged running
  - Limit root capabilities (Linux capabilities)
  - `allowPrivilegeEscalation`
  - `readOnlyRootFilesystem`

---

<!-- PDF p.379 -->
## Security

- Example: Pod security context

*(YAML: securityContext with runAsUser/fsGroup/capabilities)*

---

<!-- NEW for 1.36 -->
## Security — User Namespaces (GA in 1.36)

- `hostUsers: false` in the Pod spec maps container UIDs to unprivileged host UIDs
- Root inside the container (UID 0) is a high, unprivileged UID on the host
- Container escape no longer grants node administrative access
- Requirements: containerd 2.0+, idmap-capable filesystem
- Combine with PSA restricted profile for defense-in-depth

Ref: https://kubernetes.io/blog/2026/04/22/kubernetes-v1-36-release/

---

<!-- NEW for 1.36 -->
## Security — Mutating Admission Policies (GA in 1.36)

- Define **mutation** logic as a native Kubernetes object using **CEL** — no webhook server!
- Complements Validating Admission Policies (GA 1.30)
- Use cases: inject sidecars/labels/tolerations, default securityContext, normalize resources

```yaml
apiVersion: admissionregistration.k8s.io/v1
kind: MutatingAdmissionPolicy
spec:
  matchConstraints:
    resourceRules: [{apiGroups: [""], resources: ["pods"], operations: ["CREATE"]}]
  mutations:
    - patchType: ApplyConfiguration
      applyConfiguration:
        expression: >
          Object{ metadata: Object.metadata{ labels: {"env": "training"} } }
```

---

<!-- PDF p.380 -->
## Workshop: Security

- Part: Pod Security Admission (PSA) — `psa-*.yml` in `WorkShop_2.4_Security`

---

<!-- PDF p.381 -->
## Security

- Encryption Provider: make Secret great again
  - Default Secret is only base64 — reversible; data sits as plaintext in etcd
  - Many users pick alternatives (Vault etc.)
  - Kubernetes SIG provides **encryption at rest** for etcd data
  - Providers: AESCBC, AESGCM, Secretbox, **KMS**
  - KMS v1 (since 1.27) deprecated in 1.28; **KMS v2 stable since 1.29** — the recommended option

---

<!-- PDF p.382-385 -->
## Security

- Key Management System (KMS): envelope encryption for secret keys

*(diagrams: DEK/KEK envelope encryption, EncryptionConfiguration flow — `enc.yaml` in lab)*

Ref: https://kubernetes.io/docs/tasks/administer-cluster/kms-provider/
Ref: https://kubernetes.io/docs/tasks/administer-cluster/encrypt-data/

---

<!-- PDF p.386 -->
## Security

- Why not AESCBC everywhere? — Padding oracle attack risk; prefer AESGCM/KMS v2

Ref: https://en.wikipedia.org/wiki/Padding_oracle_attack

---

<!-- PDF p.387 -->
## Workshop: Security

- Part: Encryption Provider (`enc.yaml` — encrypt secrets at rest, verify in etcd)

---

<!-- PDF p.388 -->
# Kubernetes in real world

---

<!-- PDF p.389 — UPDATED for 1.36 -->
## Kubernetes in real world

- Kubernetes has a lot of components to install/configure in the real world
- Normally you need some 3rd-party tooling for deployment (MAAS, Ansible, etc.)
- **kubeadm** provides the standard solution to create a cluster:
  - `--apiserver-advertise-address=x.x.x.x` (default: first interface)
  - `--apiserver-bind-port=xxx` (default: 6443)
  - `--kubernetes-version=xxx` (default: latest)
  - `--pod-network-cidr=x.x.x.x` (needed by most CNIs)
  - `--token=xxxx` (default: auto-gen)

---

<!-- PDF p.390 — UPDATED for 1.36 -->
## Kubernetes in real world

- Create cluster from bare metal — steps:
  - **Phase 1**: Install prerequisites — **containerd 2.0+** (not Docker Engine!), kubelet, kubeadm; cgroups v2 host; nftables kernel modules
  - **Phase 2**: Initialize control-plane node — `kubeadm init <option>`
  - **Phase 3**: Install Pod network (CNI): **Cilium** (this course), Calico, Flannel, kube-router…
  - **Phase 4**: Join nodes — `kubeadm join <option>`

---

<!-- PDF p.391 -->
## Kubernetes in real world

*(screenshot: kubeadm docs)*

Ref: https://kubernetes.io/docs/setup/production-environment/tools/kubeadm/create-cluster-kubeadm/

---

<!-- PDF p.392-394 -->
## Kubernetes in real world

- Network options — CNI benchmarks

*(charts: CNI benchmark results — Cilium eBPF vs Calico vs Flannel, 10/40 Gbit/s)*

Ref: https://cilium.io/blog/2021/05/11/cni-benchmark
Ref: https://itnext.io/benchmark-results-of-kubernetes-network-plugins-cni-over-40gbit-s-network-2024-156f085a5e4e

---

<!-- PDF p.395-407 -->
## Kubernetes in real world

- CNI deep-dive: **Cilium** (eBPF dataplane — used in this course's cluster)
  - Hubble observability (installed in Workshop 1.1 prerequisites)
  - kube-proxy replacement option, NetworkPolicy support, encryption

*(diagrams/screenshots: cilium architecture, hubble UI, CNI comparisons)*

---

<!-- PDF p.408 -->
## Kubernetes in real world

- Initial control-plane role: `kubeadm init <options>`
- Join node in cluster: `kubeadm join <options>`

---

<!-- PDF p.409-410 -->
## Kubernetes in real world

- Initial control-plane by config file:

```bash
kubeadm init --config <configuration file>
kubeadm join --config <configuration file>
```

Ref: https://kubernetes.io/docs/reference/config-api/kubeadm-config.v1beta4/

---

<!-- PDF p.411-412 -->
## Kubernetes in real world

*(YAML: kubeadm ClusterConfiguration / InitConfiguration / JoinConfiguration v1beta4 —
note `proxy.config.mode: nftables` and containerd 2.x socket `unix:///run/containerd/containerd.sock`)*

---

<!-- PDF p.413 -->
## Kubernetes in real world

- Cloud Controller Manager supports multiple cloud providers:
  - AWS, Azure, GCE, OpenStack, vSphere, IBM Cloud, …
- Get full benefit from cloud capabilities: storage dynamic provisioning, network load balancers, node lifecycle

---

<!-- PDF p.414-418 -->
## Kubernetes in real world

- Phase 1: Install prerequisite components *(containerd 2.x + kubeadm/kubelet/kubectl v1.36)*
- Phase 2: Initialize control-plane node
- Phase 3: Install Pod Network (**Cilium** via cilium-cli)
- Phase 4: Join nodes to the cluster

*(terminal screenshots per phase)*

---

<!-- PDF p.419 -->
## Workshop: Kubernetes Real World

- Multi-node kubeadm cluster on AWS:
  - Machine `Kubernetes_MS` (control plane) + `Kubernetes_1`, `Kubernetes_2` (nodes)
  - Each: 4 vCPU / 2+ GB, IP 10.0.x.x/16
  - Client machine: kubectl + AWS CLI
- Includes: Cilium CNI, Cloud Controller, Dashboard, **APISIX gateway + AWS NLB**

---

<!-- PDF p.420 -->
## Kubernetes in real world

- Multiple control-plane nodes (HA)

Ref: https://kubernetes.io/docs/setup/production-environment/tools/kubeadm/high-availability/

---

<!-- PDF p.421 -->
# Orchestrator Assignment

---

<!-- PDF p.422 -->
## Orchestrator Assignment

- Kubernetes has several ways to control/restrict which nodes run pods:
  - **nodeSelector**: label nodes, select nodes by label criteria
  - **Interlude** — built-in node labels:
    - `kubernetes.io/hostname`
    - `topology.kubernetes.io/zone` / `topology.kubernetes.io/region`
    - `node.kubernetes.io/instance-type`
    - `kubernetes.io/arch` / `kubernetes.io/os`
  - **Affinity**: Node Affinity, Inter-Pod Affinity and Anti-Affinity
  - **Taints and Tolerations**

---

<!-- PDF p.423-424 -->
## nodeSelector

- Nodes: kubernetes-ms, kubernetes-1, kubernetes-2

*(YAML: pod with `nodeSelector: {storage: ssd}` + terminal screenshots)*

---

<!-- PDF p.425-426 -->
## Interlude

- Schedule using built-in labels (no custom labeling needed)

*(YAML: nodeSelector with `kubernetes.io/hostname` + screenshots)*

---

<!-- PDF p.427 -->
## Affinity

- **Node Affinity**: like nodeSelector but more flexible
  - `requiredDuringSchedulingIgnoredDuringExecution` (Hard)
  - `preferredDuringSchedulingIgnoredDuringExecution` (Soft)
  - If both nodeSelector and affinity are defined, **both** must be satisfied
- **Inter-pod Affinity / Anti-affinity**: based on labels of pods (X) already running on nodes, with criteria (Y) = `topologyKey`
  - `podAffinity`: run with existing pods
  - `podAntiAffinity`: don't run with existing pods
  - Ex: run pod on any node with same hostname (topologyKey) as pods labeled `environment=development`

---

<!-- PDF p.428 -->
## Node Affinity

*(YAML: nodeAffinity required + preferred example)*

---

<!-- PDF p.429 -->
## Inter-pod Affinity and Anti-affinity

- "Run Pods on any node that has pods with key (environment=development), same hostname"
- "Don't run Pods on any node that has pods with key (module=DBServer), same storage type"

*(YAML: podAffinity + podAntiAffinity example)*

---

<!-- PDF p.430 -->
## Taint and Tolerations

- Node-side consideration — force/repel Pods from a Node
- **Taint** applies to a Node: protects it from running pods without a matching toleration
- **Toleration** applies to Pods: allows (does not require) scheduling onto tainted nodes
- Use cases: dedicated/maintenance nodes, special hardware nodes
- Taint effects: `PreferNoSchedule` / `NoSchedule` / `NoExecute`

---

<!-- PDF p.431 -->
## Taint and Tolerations

```bash
kubectl taint nodes <node> dedicated=Admin:NoSchedule
```

```yaml
tolerations:
  - key: "dedicated"
    operator: "Equal"
    value: "Admin"
    effect: "NoSchedule"
```

---

<!-- PDF p.432 -->
## Workshop: Orchestrator Assignment

- 3-node AWS cluster; node labels: storage=M2 / SSD / SAS
- Lab sections:
  - Part 1: nodeSelector
  - Part 2: Interlude
  - Part 3: Affinity (Node)
  - Part 4: Inter-Pod Affinity and Anti-affinity
  - Part 5: Taint and Tolerations

---

<!-- PDF p.433 -->
# Stateful Application Deployment

---

<!-- PDF p.434 -->
## Stateful Application Deployment

- Some applications need "state" to keep the application flow and store session
  information (e.g. login session id) on the web/middle-tier server
- Ex: Joomla, Wordpress, Mantis (bug tracking), etc.

---

<!-- PDF p.435 -->
## Stateful Application Deployment

- Considering:
  - Original HTTP protocol is **stateless**
  - Stateful apps keep sessions server-side + cookies client-side
  - Sessions in memory: fast/easy but consumes resources; problems with native mobile apps / centralization
  - Scaling requires routing traffic back to the "correct" server (state stickiness)
- Awareness:
  - Containers are naturally designed for **stateless** applications
  - Load-balancers don't know about application state inside

---

<!-- PDF p.436 -->
## Stateful Application Deployment

*(diagram: 3 web servers behind LB — session created on host .200; requests hitting .201/.202 → "session operate ???")*

---

<!-- PDF p.437 -->
## Stateful Application Deployment

- Solution?
  - SDS (Software-Defined Storage) → centralized storage pool shared by all nodes
  - Web/App servers: keep application path / session path on the storage pool — every server reads/writes the same place
  - Database servers: options depend on DB type — Active/Active, Active/Hot-Passive, Active/Cold-Passive, operators (e.g. KubeDB)

---

<!-- PDF p.438 -->
## Stateful Application Deployment

*(diagram: same 3 web servers now sharing storage `/var/www/html`, `/var/session` → consistent session handling)*

---

<!-- PDF p.439-440 -->
# Persistent Volume

*(divider + concept diagram)*

---

<!-- PDF p.441 -->
## Persistent Volume

- **Persistent Volume (PV)**:
  - Cluster resource acting like a piece of storage, independent from Pods
  - Lifecycle depends on the Pod using it via **PVC**
  - Multiple types via plugin/CSI support
- **Persistent Volume Claim (PVC)** — a request for storage:
  - Size: data size to claim
  - Access modes: `ReadWriteOnce (RWO)` / `ReadOnlyMany (ROX)` / `ReadWriteMany (RWX)`
- **StorageClass**: "profiling storage" concept — classify storage (IOPS, region, etc.)

---

<!-- PDF p.442 -->
## Persistent Volume

- Volume Life Cycle:
  - **Provision** (static: PV pool / dynamic: StorageClass)
  - **Binding**: PVC matched to PV (phase Available → Bound)
  - **Using**: pod references the claim
  - **Reclaim** (end of use, phase Released):
    - Retain (keep data)
    - Recycle (`rm -rf /path` — deprecated)
    - Delete (default for dynamic provisioning)

*(diagram: PV pool BKK/HK/NYK, PVC request, bind/use/reclaim flow)*

---

<!-- PDF p.443-444 -->
## Persistent Volume

- Static Provision *(YAML: PV + PVC nfs example)*
- Dynamic Provision *(YAML: StorageClass + PVC)*

---

<!-- PDF p.445-447 -->
## Persistent Volume

- Types of Persistent Volume / Access Methods

*(table: volume plugin × RWO/ROX/RWX support)*

---

<!-- PDF p.448-451 -->
## Persistent Volume

- **CSI (Container Storage Interface)** drivers — the standard for all storage now

*(tables/screenshots: CSI driver list)*

Ref: https://kubernetes-csi.github.io/docs/drivers.html

---

<!-- PDF p.452-455 -->
## Persistent Volume

- AWS EBS CSI driver install & IMDS configuration

*(screenshots: aws-ebs-csi-driver install, IMDS settings, EBS snapshots for EKS)*

Ref: https://github.com/kubernetes-sigs/aws-ebs-csi-driver

---

<!-- PDF p.456-457 -->
## Persistent Volume

- Static Provision: PersistentVolume ↔ PersistentVolumeClaims *(YAML)*
- Dynamic Provision: StorageClass ↔ PersistentVolumeClaims *(YAML — `aws_ebs_sc.yml`, `aws_pvc.yml`)*

---

<!-- PDF p.458 -->
## Persistent Volume

- CSI Storage Resizing (authenticated): Secret + StorageClass

```yaml
allowVolumeExpansion: true
```

- **NEW (alpha in 1.36): in-place EBS volume resize without pod restart**

---

<!-- PDF p.459-463 -->
## Persistent Volume

- Dynamic Provision lab: create SC → create PVC → watch PV auto-created → bind

*(terminal screenshots per step)*

---

<!-- PDF p.464-465 -->
## Persistent Volume

- Deployment reference:
  - Mount disk volume from **PVC** (`persistentVolumeClaim.claimName`)
  - Run initContainers process to prepare data
- Dynamic Provision on Deployment (`aws_webtest_deployall.yml`)

---

<!-- PDF p.466-469 -->
## Persistent Volume

- Create Deployment with PVC and verify data survives pod restarts

*(terminal screenshots)*

---

<!-- NEW for 1.36 -->
## Persistent Volume — OCI VolumeSource (stable in 1.36)

- Not everything needs a PV: read-only artifacts can ship as **OCI images**

```yaml
volumes:
  - name: dataset
    image:
      reference: registry.example/datasets/lab:v1
```

- Version, sign, and distribute static data like you do container images

---

<!-- PDF p.470 -->
## Workshop: Persistence Storage

- `WorkShop_2.7_Persistent_Storage` — NFS static + AWS EBS CSI dynamic provisioning (+ encrypted SC)

---

<!-- PDF p.471 -->
# StatefulSet

---

<!-- PDF p.472 -->
## StatefulSet

- What is StatefulSet?
  - Remember Deployment? StatefulSet is similar…
  - …but designed for applications that need:
    - Persistent Storage / stable storage & network
    - Ordered deployment (create), scale, rolling update
- Benefits from StatefulSet:
  - Sequential create/scale/update of Pods
  - Each Pod gets unique resources:
    - Pod name (`TEST-01`, `TEST-02`, …)
    - Dedicated storage (own PVC via `volumeClaimTemplates`)
    - Stable host/network identity

---

<!-- PDF p.473 -->
## StatefulSet

*(diagram: StatefulSet TEST with volumeClaimTemplates DB → Pods TEST-01/02/03 on Node1/2/3,
each with PVC DB-TEST-0X, fronted by a Headless Service)*

---

<!-- PDF p.474 -->
## StatefulSet

- StatefulSet + Headless Service *(YAML listings)*

---

<!-- PDF p.475 -->
## Recapture Day 2

- Fundamental of Kubernetes: Job and CronJob / Log and Monitoring
- **Gateway API Networking (APISIX)** — NGINX Ingress retired
- Security on Kubernetes: Network/Volume/Resource/Access/Security policies,
  Encryption Provider, **Mutating Admission Policies (GA)**, **User Namespaces (GA)**
- Kubernetes in real world: bare metal cluster setup (containerd 2.0, nftables, Cilium)
- Orchestrator Assignment: nodeSelector / Interlude / Affinity / Taints & Tolerations
- Stateful application deployment: PV / PVC / StatefulSet / **OCI VolumeSource**

---

<!-- PDF p.476 -->
# Thank You

**Kubernetes: Production Workload Orchestration — v1.36 (Haru) Edition**

By: Praparn L (eva10409@gmail.com)
Repo: https://github.com/praparn/kubernetes_202607
