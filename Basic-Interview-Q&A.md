# Basic Kubernetes Interview Questions and Answers

### 1. What is Kubernetes and what are its main components?

**Answer:**
Kubernetes is an open-source container orchestration platform that automates the deployment, scaling, and management of containerized applications. It groups containers into logical units for easy management and discovery.

**Main Components:**

1. **Master Node**:
   - **API Server**: Exposes the Kubernetes API, the front end of the control plane.
   - **Scheduler**: Assigns workloads to nodes based on resource availability.
   - **Controller Manager**: Runs controller processes to regulate the state of the cluster.
   - **etcd**: A key-value store that stores all cluster data.

2. **Worker Node**:
   - **Kubelet**: An agent that runs on each node in the cluster and ensures containers are running in a Pod.
   - **Kube-proxy**: Maintains network rules on nodes and handles network communication to and from Pods.
   - **Container Runtime**: Software responsible for running containers (e.g., Docker, containerd).

3. **Pods**: The smallest deployable units of computing that can be created and managed in Kubernetes.

4. **Services**: An abstraction that defines a logical set of Pods and a policy by which to access them.

5. **Namespaces**: Virtual clusters within a Kubernetes cluster to partition resources and manage multiple environments.

### 2. How does Kubernetes manage containerized applications?

**Answer:**
Kubernetes manages containerized applications through the following key functionalities:

1. **Deployment**: Ensures that the desired number of application instances are running. Uses Deployment controllers to handle rolling updates and rollbacks.

2. **Scaling**: Automatically scales applications up or down based on resource usage or manual intervention using Horizontal Pod Autoscaler.

3. **Load Balancing**: Distributes traffic evenly across multiple instances of an application using Services and Ingress controllers.

4. **Self-Healing**: Restarts failed containers, replaces containers, kills containers that don't respond to user-defined health checks, and prevents containers from serving traffic until they're ready.

5. **Configuration Management**: Manages configuration data and secrets for applications using ConfigMaps and Secrets.

6. **Storage Orchestration**: Mounts storage systems, such as local storage, public cloud providers, and network storage in Pods using Persistent Volumes (PVs) and Persistent Volume Claims (PVCs).

### 3. What is a Pod in Kubernetes, and why is it important?

**Answer:**
A Pod is the smallest deployable unit in Kubernetes. It represents a single instance of a running process in a cluster and can contain one or more containers. 

**Importance:**
- Pods provide an abstraction that enables Kubernetes to manage applications and their components more effectively.
- Containers in a Pod share the same network namespace, IP address, and port space, which facilitates easy inter-container communication.
- Pods enable resource sharing among containers, such as volumes and secrets.
- They allow applications to be deployed, scaled, and managed efficiently as a unit.

### 4. Explain the difference between Deployments, StatefulSets, and DaemonSets.

**Answer:**

1. **Deployments**:
   - Manages stateless applications.
   - Ensures that the desired number of Pod replicas are running.
   - Supports rolling updates and rollbacks.
   - Example Use Case: Web servers, APIs.

2. **StatefulSets**:
   - Manages stateful applications.
   - Ensures that Pod replicas have a unique, stable identity and persistent storage.
   - Maintains the order and uniqueness of Pods.
   - Example Use Case: Databases, distributed file systems.

3. **DaemonSets**:
   - Ensures that a copy of a Pod runs on all (or some) nodes.
   - Typically used for logging, monitoring, or other node-level services.
   - Example Use Case: Node monitoring agents, log collectors.

### 5. How do you manage Secrets in Kubernetes?

**Answer:**
Secrets in Kubernetes are used to store and manage sensitive information such as passwords, OAuth tokens, and SSH keys.

**Steps to Manage Secrets:**

1. **Create a Secret**: You can create a Secret using a YAML file or via the `kubectl create secret` command.
   ```yaml
   apiVersion: v1
   kind: Secret
   metadata:
     name: my-secret
   type: Opaque
   data:
     username: dXNlcm5hbWU=  # base64 encoded
     password: cGFzc3dvcmQ=  # base64 encoded
   ```

   Command:
   ```bash
   kubectl create secret generic my-secret --from-literal=username=user --from-literal=password=pass
   ```

2. **Accessing Secrets in Pods**: Secrets can be mounted as volumes or exposed as environment variables within Pods.
   ```yaml
   apiVersion: v1
   kind: Pod
   metadata:
     name: mypod
   spec:
     containers:
     - name: mycontainer
       image: myimage
       env:
       - name: USERNAME
         valueFrom:
           secretKeyRef:
             name: my-secret
             key: username
       - name: PASSWORD
         valueFrom:
           secretKeyRef:
             name: my-secret
             key: password
   ```

### 6. What is the role of Ingress in Kubernetes?

**Answer:**
Ingress in Kubernetes is an API object that manages external access to the services within a cluster, typically HTTP/HTTPS.

**Functions of Ingress:**
- Provides load balancing for HTTP and HTTPS traffic.
- Offers name-based virtual hosting, allowing multiple services to be accessed under the same IP address based on the hostname.
- Facilitates SSL/TLS termination, handling encryption/decryption of traffic.
- Implements path-based routing, directing traffic to different services based on the request URI.

**Example Ingress Resource:**
```yaml
apiVersion: networking.k8s.io/v1
kind: Ingress
metadata:
  name: example-ingress
spec:
  rules:
  - host: myapp.example.com
    http:
      paths:
      - path: /
        pathType: Prefix
        backend:
          service:
            name: my-service
            port:
              number: 80
```

### 7. How does Kubernetes handle resource allocation?

**Answer:**
Kubernetes uses resource requests and limits to manage resource allocation for Pods:

1. **Resource Requests**: Specifies the minimum amount of CPU and memory that a container requires. The scheduler uses these values to decide which node to place a Pod on.

2. **Resource Limits**: Specifies the maximum amount of CPU and memory that a container can use. If a container exceeds its limit, it may be throttled or terminated.

**Example of Resource Requests and Limits:**
```yaml
apiVersion: v1
kind: Pod
metadata:
  name: resource-demo
spec:
  containers:
  - name: mycontainer
    image: myimage
    resources:
      requests:
        memory: "64Mi"
        cpu: "250m"
      limits:
        memory: "128Mi"
        cpu: "500m"
```

### 8. How do you monitor a Kubernetes cluster?

**Answer:**
Monitoring a Kubernetes cluster involves observing the health and performance of the cluster components and the applications running on it. Common tools and approaches include:

1. **Metrics Server**: Collects resource metrics from the kubelet and exposes them via the Metrics API.
2. **Prometheus**: An open-source monitoring system that collects metrics from Kubernetes components and applications.
3. **Grafana**: A visualization tool that can be used with Prometheus to create dashboards and alerts.
4. **Logging**: Tools like Fluentd, Elasticsearch, and Kibana (EFK stack) for log aggregation, storage, and analysis.
5. **Kubernetes Dashboard**: A web-based UI for monitoring the state of the cluster and managing applications.

**Setting Up Prometheus and Grafana:**
- Deploy Prometheus using Helm:
  ```bash
  helm install prometheus stable/prometheus
  ```
- Deploy Grafana using Helm:
  ```bash
  helm install grafana stable/grafana
  ```
### 9.Types of Node Affinity

**Node affinity has two types:**

**RequiredDuringSchedulingIgnoredDuringExecution**: This type of node affinity is a hard requirement, meaning the Pod will only be scheduled on nodes that satisfy the specified conditions. If no such nodes are available, the Pod will not be scheduled.

**PreferredDuringSchedulingIgnoredDuringExecution**: This type of node affinity is a soft requirement. Kubernetes will try to schedule the Pod on nodes that satisfy the specified conditions, but if no such nodes are available, the Pod can still be scheduled on other nodes.

### Syntax and Usage

Node affinity is specified in the "Affinity" field of a pods specification.

### Example of YAML configuration:

```yaml
apiVersion: v1
kind: Pod
metadata:
  name: node-affinity-pod
spec:
  containers:
  - name: nginx
    image: nginx
  affinity:
    nodeAffinity:
      requiredDuringSchedulingIgnoredDuringExecution:
        nodeSelectorTerms:
        - matchExpressions:
          - key: disktype
            operator: In
            values:
            - ssd
      preferredDuringSchedulingIgnoredDuringExecution:
        - weight: 1
          preference:
            matchExpressions:
            - key: region
              operator: In
              values:
              - us-west-1
```
