eadme · MD
# DSO202 — Practical 01: Kubernetes Cluster Fundamentals with Kind
 
This repository contains the deliverables for DSO202 Practical 01, which covers provisioning a local multi-node Kubernetes cluster with **Kind** and working through core Kubernetes concepts — namespaces, resource governance, Pods, Deployments, rolling updates/rollbacks, and Services — using both imperative and declarative `kubectl` workflows.
 
Full write-up, commands, and evidence are documented in [`report/practical-01-report.md`](report/practical-01-report.md).
 
## Prerequisites
 
| Tool | Version | Purpose |
|---|---|---|
| [Docker Engine / Docker Desktop](https://docs.docker.com/get-docker/) | 24.0 | Container runtime that hosts each Kind node |
| [Kind](https://kind.sigs.k8s.io/) | 0.32.0 | Creates and deletes local Kubernetes clusters |
| [kubectl](https://kubernetes.io/docs/tasks/tools/) | 1.35 / 1.36 | CLI used to communicate with the cluster |
 
Verify installations:
 
```bash
docker --version
kind --version
kubectl version --client
```
 
## Repository Structure
 
```
dso202-practical-01/
├── README.md
├── cluster/
│   └── kind-cluster.yaml          # Kind cluster topology (1 control-plane, 2 workers)
├── evidence/                      # Screenshots and captured output from each step
│   ├── 1.png ... 86.png
│   ├── final-state-all.txt
│   ├── final-state-events.txt
│   ├── final-state-nodes.txt
│   └── web-imperative-as-stored.yaml
├── manifests/                     # Declarative Kubernetes manifests, applied in lexical order
│   ├── 00-namespace.yaml
│   ├── 01-quota-and-limits.yaml
│   ├── 02-pod-web.yaml
│   ├── 03-deployment-web.yaml
│   ├── 04-service-clusterip.yaml
│   ├── 05-service-nodeport.yaml
│   └── 06-pod-client.yaml
└── report/
    └── practical-01-report.md     # Full step-by-step report with evidence
```
 
## Cluster Architecture
 
The cluster is a 3-node Kind topology, with each node running as a separate Docker container:
 
- **1 control-plane node** — manages the cluster, scheduling, and scaling decisions.
- **2 worker nodes** — run application workloads and provide compute resources.
## Getting Started
 
### 1. Create the cluster
 
```bash
kind create cluster --config cluster/kind-cluster.yaml
```
 
### 2. Verify the cluster
 
```bash
kubectl config current-context
kubectl cluster-info
kubectl get nodes -o wide
```
 
### 3. Apply the manifests
 
`kubectl apply -f` applies files in a directory in lexical order, which is why the manifests are numbered:
 
```bash
kubectl apply -f manifests/
kubectl get all
```
 
### 4. Access the application
 
- **Via NodePort** (from the host, no port-forward required):
```bash
  curl -s http://localhost:30080
```
 
- **Via port-forward** (for debugging a single Pod):
```bash
  kubectl port-forward pod/web-pod 8080:80
  curl http://localhost:8080
```
 
### 5. Set the default namespace (optional)
 
To avoid passing `-n dso202-practical` on every command:
 
```bash
kubectl config set-context --current --namespace=dso202-practical
```
 
## Cleanup
 
```bash
# Capture final state before teardown
kubectl get all -o wide > evidence/final-state-all.txt
kubectl get resourcequota,limitrange,endpointslice -o wide >> evidence/final-state-all.txt
kubectl get nodes -o wide > evidence/final-state-nodes.txt
kubectl get events --sort-by=.lastTimestamp > evidence/final-state-events.txt
 
# Delete workload objects declaratively
kubectl delete -f manifests/06-pod-client.yaml
kubectl delete -f manifests/05-service-nodeport.yaml
kubectl delete -f manifests/04-service-clusterip.yaml
kubectl delete -f manifests/03-deployment-web.yaml
kubectl delete -f manifests/02-pod-web.yaml
 
# Reset kubectl's default namespace
kubectl config set-context --current --namespace=default
 
# Delete the cluster
kind delete cluster --name dso202
```
 
## What This Practical Covers
 
- Provisioning a multi-node cluster with Kind
- Inspecting cluster architecture, nodes, and control-plane components
- Namespaces, `ResourceQuota`, and `LimitRange` enforcement
- Pods: imperative vs. declarative creation, labels, annotations, logs, `exec`, `port-forward`
- Deployments: the Deployment → ReplicaSet → Pod ownership chain, self-healing, scaling
- Rolling updates, rollback via revision history, and recovering from a failed rollout
- Services: `ClusterIP`, `NodePort`, EndpointSlices, load balancing, readiness-gated traffic, and diagnosing selector mismatches
- The `LoadBalancer` Service type's limitation inside Kind
## Reference
 
This practical follows the guide and manifest walkthroughs at:
 
- [DSO202 Practical 1 — Guide](https://hackmd.io/@sarojsanyasi/dso202-practical1-guide)
- [DSO202 Practical 1 — Manifest Files](https://hackmd.io/@sarojsanyasi/dso202-practical1-manifest-files)
- [Additional Notes](https://hackmd.io/@sarojsanyasi/HJtoSFwUGl)