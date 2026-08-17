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

### Prove that the LimitRange is doing its work. Start a Pod imperatively with no resource declaration at all, then read back what the cluster actually stored.
```bash
kubectl run limitrange-check --image=nginx:1.30-alpine --restart=Never
kubectl get pod limitrange-check -o jsonpath='{.spec.containers[0].resources}'
echo
```

![24](../evidence/24.png)

### Delete the check Pod, since it is not part of the deliverable.

```bash
kubectl delete pod limitrange-check
```
![25](../evidence/25.png)

## Stage 4 — Pods

### Create a Pod with one command, then inspect it.

```bash
kubectl run web-imperative \
  --image=nginx:1.30-alpine \
  --restart=Never \
  --port=80 \
  --labels='app=web,tier=frontend,managed-by=imperative'

```
![26](../evidence/26.png)


### Watch the Pod reach Running. Press Ctrl+C to stop watching.

```bash
kubectl get pod web-imperative --watch
```
![27](../evidence/27.png)

### Capture the imperative Pod as a manifest. This is the bridge between the two styles.

```bash
kubectl get pod web-imperative -o yaml > evidence/web-imperative-as-stored.yaml
```
![28](../evidence/28.png)

### Delete the imperative Pod.

```bash
kubectl delete pod web-imperative
```
![29](../evidence/29.png)


## The declarative route

### Copy Listing 4 into manifests/02-pod-web.yaml and read the comments in it before applying. Then apply it.

```bash
kubectl apply -f manifests/02-pod-web.yaml
```
![30](../evidence/30.png)

### Apply the same file a second time without changing it.
```bash
kubectl apply -f manifests/02-pod-web.yaml
```
![31](../evidence/31.png)

### Confirm placement and the assigned Pod IP.

```bash 
 kubectl get pod web-pod -o wide
```
![32](../evidence/32.png)

###  Confirm the resource declarations took effect.

```bash
kubectl get pod web-pod -o jsonpath='{.spec.containers[0].resources}'
echo
```
![33](../evidence/33.png)

### These values came from the manifest, not from the LimitRange. Verify by re-reading the quota:
```bash
kubectl describe resourcequota dso202-quota | grep -E 'requests.cpu|requests.memory|pods'
```
![34](../evidence/34.png)

### Describe the Pod and read the Events: section at the bottom.
```bash
kubectl describe pod web-pod
```
![35](../evidence/35.png)

### Display labels alongside the Pod list.
```bash
kubectl get pods --show-labels
```
![36](../evidence/36.png)

### Select Pods by label rather than by name. This is how every controller and every Service finds its Pods.

```bash
kubectl get pods -l app=web
kubectl get pods -l tier=frontend,managed-by=declarative
kubectl get pods -l 'tier in (frontend,backend)'
kubectl get pods -l app!=web
```
![37](../evidence/37.png)

### Add and then remove a label at runtime. The trailing hyphen removes a label.
```bash
kubectl label pod web-pod environment=practical
kubectl get pods -l environment=practical
kubectl label pod web-pod environment-
```
![38](../evidence/38.png)

###  Add an annotation and observe that it cannot be selected on.

```bash
kubectl annotate pod web-pod dso202.rub.edu.bt/practical="01"
kubectl get pod web-pod -o jsonpath='{.metadata.annotations}'
echo
```
![39](../evidence/39.png)

### Read the container's log stream. -f follows it; --tail limits the history.

```bash
kubectl logs web-pod --tail=5
```
![40](../evidence/40.png)

### Open a shell inside the running container. Because nginx:1.30-alpine is Alpine-based, the shell is sh, not bash.

```bash
kubectl exec -it web-pod -- sh
```
![41](../evidence/41.png)
![42](../evidence/42.png)

### Run a single command inside the container without an interactive session.

![43](../evidence/43.png)

### Forward a local port to the Pod. This opens a tunnel from the host, through the API server, to the Pod. It is intended for debugging, never for exposing an application.

```bash
kubectl port-forward pod/web-pod 8080:80
```
![44](../evidence/44.png)

Leave that command running, and in a second terminal:
```bash
curl http://localhost:8080
```
![45](../evidence/45.png)

### Use kubectl explain whenever a field is unfamiliar. It reads the schema from the live cluster, so its answer is always correct for the running version.

```bash
kubectl explain pod.spec.containers.resources
kubectl explain pod.spec.containers.livenessProbe --recursive | head -30
```
![46](../evidence/46.png)

## Deployments

### Generate a Deployment manifest imperatively, purely for comparison. Then read Listing 8 and note everything the generated version omits: labels beyond the bare minimum, resource declarations, probes, a rollout strategy, and comments.

```bash
kubectl create deployment web-deployment \
  --image=nginx:1.30-alpine --replicas=3 --dry-run=client -o yaml | head -25

```
![47](../evidence/47.png)

### Copy Listing 5 into manifests/03-deployment-web.yaml and apply it.
```bash
kubectl apply -f manifests/03-deployment-web.yaml
kubectl rollout status deployment/web-deployment
```
![48](../evidence/48.png)

### Observe the three-level ownership chain in the cluster.

```bash
kubectl get deployment,replicaset,pod -l app=web
```
![49](../evidence/49.png)

###  Confirm the ownership relationship rather than inferring it from names.
```bash
kubectl get replicaset -l app=web -o jsonpath='{.items[0].metadata.ownerReferences[0].kind}/{.items[0].metadata.ownerReferences[0].name}{"\n"}'
```
![50](../evidence/50.png)

### kubectl get pods -l app=web -o wide --no-headers | awk '{print $1, $7}'

```bash
kubectl get pods -l app=web -o wide --no-headers | awk '{print $1, $7}'

```
![51](../evidence/51.png)

### Demonstrate self-healing. Delete one Pod and immediately list them again.


```bash
victim=$(kubectl get pods -l app=web -o jsonpath='{.items[0].metadata.name}')
echo "deleting $victim"
kubectl delete pod "$victim"
kubectl get pods -l app=web

```

![52](../evidence/52.png)

A replacement Pod appears within seconds, with a new random suffix but the same ReplicaSet prefix. Record the before-and-after listing in the report; this is the single clearest illustration of reconciliation in the practical.

```bash
kubectl get events --field-selector reason=SuccessfulCreate --sort-by=.lastTimestamp | tail -3

```
![53](../evidence/53.png)


##  Scaling

### Scale imperatively, then read the result.
```bash
kubectl scale deployment web-deployment --replicas=5
kubectl get deployment web-deployment
```
![54](../evidence//54.png)

###  Return to three replicas the declarative way, by re-applying the unmodified manifest.

```bash

kubectl apply -f manifests/06-deployment-web.yaml
kubectl get deployment web-deployment
```
![55](../evidence/55.png)

## Rolling update and rollback

### Watch the rollout in a second terminal.

```bash
kubectl get pods -l app=web --watch
```
![55](../evidence/56.png)

### In the first terminal, change the image version. nginx:1.31-alpine is the current mainline release; nginx:1.30-alpine is the stable release used so far.
![57](../evidence/57.png)

### Confirm two ReplicaSets now exist, one of them scaled to zero.

```bash
kubectl get replicaset -l app=web
```
![58](../evidence/58.png)

### Read the revision history.

```bash
kubectl rollout history deployment/web-deployment
```
![59](../evidence/59.png)


### Inspect one revision in detail.

```bash
kubectl rollout history deployment/web-deployment --revision=1 | grep -i image
```
![60](../evidence/60.png)

### Roll back to the previous revision and confirm.

```bash
kubectl rollout undo deployment/web-deployment
kubectl rollout status deployment/web-deployment
kubectl get deployment web-deployment -o jsonpath='{.spec.template.spec.containers[0].image}{"\n"}'
```
![61](../evidence/61.png)

### Demonstrate a failed rollout and recover from it. This is deliberate: a rollout that cannot succeed must be recognised quickly.
```bash
kubectl set image deployment/web-deployment web=nginx:9.99-does-not-exist
kubectl rollout status deployment/web-deployment --timeout=60s
```
![62](../evidence/62.png)

```bash
kubectl get pods -l app=web
```
![63](../evidence/63.png)

### Diagnose and undo.
```bash
kubectl describe pod -l app=web | grep -A3 'Failed'
kubectl rollout undo deployment/web-deployment
kubectl rollout status deployment/web-deployment
kubectl get pods -l app=web
```
![64](../evidence/64.png)

### Restore the declared state, so that the repository and the cluster agree again.
```bash
kubectl apply -f manifests/03-deployment-web.yaml
kubectl diff -f manifests/03-deployment-web.yaml && echo "cluster matches manifest"
```
![65](../evidence/65.png)

## Stage 6 — Services

### Copy Listing 6 into manifests/04-service-clusterip.yaml and apply it.
```bash
kubectl apply -f manifests/04-service-clusterip.yaml
kubectl get service web-clusterip
```
![66](../evidence/66.png)

### Read the EndpointSlice the controller generated. This is the list of addresses the Service will actually send traffic to.
```bash
kubectl get endpointslice -l kubernetes.io/service-name=web-clusterip
```
![67](../evidence/67.png)

### Copy Listing 8 into manifests/06-pod-client.yaml and apply it. This Pod exists only to issue requests from inside the cluster.
```bash
kubectl apply -f manifests/09-pod-client.yaml
kubectl wait --for=condition=Ready pod/client-pod --timeout=60s
```
![68](../evidence/68.png)

### Resolve the Service name from inside the cluster.
```bash
kubectl exec client-pod -- nslookup web-clusterip
```
![69](../evidence/69.png)

### Send requests through the Service.
```bash
kubectl exec client-pod -- wget -qO- http://web-clusterip | head -4
```
![70](../evidence/70.png)

### Demonstrate load balancing. Each Pod's hostname is its Pod name, so writing a distinct page into each Pod makes the distribution visible.
```bash
for pod in $(kubectl get pods -l app=web -o name); do
  kubectl exec "${pod#pod/}" -- sh -c 'echo "served by $HOSTNAME" > /usr/share/nginx/html/index.html'
done

for i in $(seq 1 9); do
  kubectl exec client-pod -- wget -qO- http://web-clusterip
done | sort | uniq -c
```
![71](../evidence/71.png)

### Demonstrate that readiness gates traffic. Break the readiness probe on one Pod and watch it leave the EndpointSlice. The readiness probe in Listing 8 requests /index.html, so deleting that file makes the probe fail. Its liveness probe is a TCP check, which keeps passing, so the container is not restarted.
```bash
target=$(kubectl get pods -l app=web -o jsonpath='{.items[0].metadata.name}')
kubectl exec "$target" -- rm -f /usr/share/nginx/html/index.html
sleep 15
kubectl get pod "$target"
kubectl get endpointslice -l kubernetes.io/service-name=web-clusterip
```
![72](../evidence/72.png)

### The Pod is still Running with RESTARTS at 0, but it has been removed from the EndpointSlice and will receive no further traffic. Repeat the nine-request loop from Step 6 and confirm that only two Pod names now appear. Then restore it:
```bash 
kubectl exec "$target" -- sh -c 'echo "served by $HOSTNAME" > /usr/share/nginx/html/index.html'
sleep 10
kubectl get endpointslice -l kubernetes.io/service-name=web-clusterip
```
![73](../evidence/73.png)

### Demonstrate the most common Service fault: a selector that matches nothing.
```bash
kubectl create service clusterip broken-service --tcp=80:80
kubectl get endpointslice -l kubernetes.io/service-name=broken-service
```
![74](../evidence/74.png)


### The imperative command generated the selector app=broken-service, which no Pod carries, so the EndpointSlice is empty and every request to the Service will be refused. An empty EndpointSlice is the diagnostic signature of a selector mismatch. Remove it:
```bash
kubectl delete service broken-service
```
![75](../evidence/75.png)

## NodePort
### Copy Listing 7 into manifests/05-service-nodeport.yaml and apply it. It fixes nodePort: 30080, which is the port Listing 1 published from the control-plane container to the host.
```bash
kubectl apply -f manifests/05-service-nodeport.yaml
kubectl get service
```
![76](../evidence/76.png)


###  Reach the application from the host machine, outside the cluster, with no port-forward running.
```bash
curl -s http://localhost:30080
```
![77](../evidence/77.png)

### Confirm that the node port is open on every node, not only the one published to the host.
```bash
docker exec dso202-worker curl -s http://localhost:30080
```
![78](../evidence/78.png)

### Confirm that a LoadBalancer Service cannot complete in kind, so that the behaviour is recognised rather than mistaken for a fault.
```bash
kubectl create service loadbalancer lb-demo --tcp=80:80
kubectl get service lb-demo
kubectl delete service lb-demo
```
![79](../evidence/79.png)

## CleanUp

### Capture final evidence before deleting anything.
```bash
mkdir -p evidence
kubectl get all -o wide > evidence/final-state-all.txt
kubectl get resourcequota,limitrange,endpointslice -o wide >> evidence/final-state-all.txt
kubectl get nodes -o wide > evidence/final-state-nodes.txt
kubectl get events --sort-by=.lastTimestamp > evidence/final-state-events.txt
```
![80](../evidence/80.png)

### Delete the workload objects declaratively, in reverse order of creation. Deleting from the same files that created the objects is the check that no object was created outside version control.
```bash
kubectl delete -f manifests/06-pod-client.yaml
kubectl delete -f manifests/05-service-nodeport.yaml
kubectl delete -f manifests/04-service-clusterip.yaml
kubectl delete -f manifests/03-deployment-web.yaml
kubectl delete -f manifests/02-pod-web.yaml
```
![81](../evidence/81.png)

### Confirm the namespace is empty apart from the quota and limit range:
```bash
kubectl get all
```
![82](../evidence/82.png)

### Rebuild everything from the repository in one command, to prove reproducibility. kubectl apply -f accepts a directory and applies the files in lexical order, which is exactly why the filenames are numbered.
```bash
kubectl apply -f manifests/
kubectl get all
```
![83](../evidence/83.png)


### Reset the kubectl default namespace, so that later work is not silently placed in this namespace.
```bash
kubectl config set-context --current --namespace=default
```
![84](../evidence/84.png)


### Delete the cluster.
```bash
kind delete cluster --name dso202
kind get clusters
docker ps
```
![85](../evidence/85.png)

### Optionally reclaim the node image, which is roughly 1 GB. Keeping it makes the next cluster creation far faster, so remove it only if disk space is short.
```bash
kind get clusters
```
![86](../evidence/86.png)























