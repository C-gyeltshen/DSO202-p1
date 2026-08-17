## Creating basic practicl folder structure
![1](../evidence/1.png)

## Stage 0: Prerequisites and Verification

We need the following
1. **Docker Engine or Docker Desktop** : (Version: 	24.0) Provides the container runtime in which kind runs each node.
2. **Kind** : (version: 0.32.0) Create cluster and delete clusters.
3. **kubectl** : (version : 1.35 or 1.36) Talks to clsudter.

![2](../evidence/2.png)

## Stage 1: Creating the Three-Node Cluster
Since we dont have nodes configured for practicals we are usning our laptop to host three different containers which will act as nodes for our cluster. We will be having one control plane node and two worker nodes.
1. Control Plane Node : This node will be responsible for managing the cluster and making decisions about scheduling and scaling.
2. Worker Nodes : These nodes will run the actual application workloads and provide the compute resources for the cluster.


### Steps
### Step 1: Writing .ymal file to create three different cotainers for countrol plane and 2 worker nodes.

```
cd cluster
touch kind-cluster.yaml
```
![3](../evidence/3.png)

### Step 2: Creating the cluster containing 3 containers (1 control plane and 2 worker nodes) by runnning the `kind-cluster.yaml` file by running the below command
```bash
kind create cluster --config cluster/kind-cluster.yaml
```
![4](../evidence/4.png)

`Checking`
```bash
docker ps
```
![5](../evidence/5.png)

### Confirm the same three nodes as Docker containers

```bash
docker ps --format 'table {{.Names}}\t{{.Image}}\t{{.Status}}'
```
![6](../evidence/6.png)

### Confirm that kubectl is pointing at the new cluster. kind adds a context named kind-<cluster-name> and selects it automatically.

```bash
kubectl config current-context
```
![7](../evidence/7.png)

## Step 3: Inspecting the Cluster and Its Components

### Ask the cluster where its control plane is.

```bash
kubectl cluster-info
```
![8](../evidence/8.png)

### list the nodes
```bash
kubectl get nodes
```
![9](../evidence/9.png)

### Add columns to the same query. -o wide is the fastest way to get more detail without switching to full YAML output.

```bash 
kubectl get nodes -o wide
```
![10](../evidence/10.png)

### Read one node in detail and locate the labls.
```bash
kubectl describe node worker-node-1
```
![11](../evidence/11.png)

```bash
kubectl get node worker-node-1 -o jsonpath='{.metadata.labels}' | tr ',' '\n'
```
![12](../evidence/12.png)

### list all namespace 
```bash 
kubectl get namespaces
```
![13](../evidence/13.png)

### List all the control plan components
```bash
kubectl get pods -n kube-system -o wide
```
![14](../evidence/14.png)

### Read the log of one control-plane component. 
```bash
kubectl logs -n kube-system kube-scheduler-control-plane --tail=10
```
![15](../evidence/15.png)

###  List every kind of object the cluster knows about, and note which are namespaced.

```bash
kubectl api-resources --namespaced=true -o name | head -20
kubectl api-resources --namespaced=false -o name | head -10
```
![16](../evidence/16.png)


## Stage 4: Namespaces, Resource Quotas, and Limit Ranges

### Create the namespace imperatively first, purely to see the imperative form, then delete it. The declarative version in Listing 2 is the graded artefact
![17](../evidence/17.png)

### Generate a manifest from an imperative command without executing it. Compare the result with Listing 2 and note that Listing 2 adds labels, which the generated version does not.
![18](../evidence/18.png)

### Copy Listing 2 into manifests/00-namespace.yaml and apply it.
```bash
kubectl apply -f manifests/00-namespace.yaml
```
![19](../evidence/19.png)

### Set the namespace as the default for the current context. Every subsequent command in this practical then omits -n.

```bash
kubectl config set-context --current --namespace=dso202-practical
kubectl config view --minify --output 'jsonpath={..namespace}'
```
![20](../evidence/20.png)

### Copy Listing 3 into manifests/01-quota-and-limits.yaml and apply it. The file contains two objects separated by ---.

```bash
kubectl apply -f manifests/01-quota-and-limits.yaml
```
![21](../evidence/21.png)

### kubectl describe resourcequota dso202-quota
```bash
kubectl describe resourcequota dso202-quota
```
![22](../evidence/22.png)

### Read the limit range.

```bash
kubectl describe limitrange dso202-limits
```
![23](../evidence/23.png)


## Stage 4 — Pods

Create a Pod with one command, then inspect it.















