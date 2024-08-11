# Autoscaling

Kubernetes autoscaling is the ability to automatically adjust the number of pods, or the resources allocated to pods, in a Kubernetes cluster based on the current load and demand. This helps ensure that applications run efficiently, respond to traffic spikes, and reduce resource wastage by scaling down during low demand.

There are three main types of autoscaling in Kubernetes:

### 1. **Horizontal Pod Autoscaler (HPA)**
   - **Functionality**: HPA automatically adjusts the number of pods in a deployment, replica set, or stateful set based on observed CPU utilization or other selected metrics (like memory, custom metrics, etc.).
   - **Example**: Suppose you have a web application with a deployment of 3 pods. You configure HPA to maintain CPU utilization at 50%. If traffic increases and CPU usage goes beyond 50%, HPA will increase the number of pods to, say, 6. Conversely, if the traffic drops and CPU utilization falls below 50%, HPA may scale down the pods to 2 or 3.
   - **Command to create HPA**:
     ```bash
     kubectl autoscale deployment <deployment-name> --cpu-percent=50 --min=2 --max=10
     ```

### 2. **Vertical Pod Autoscaler (VPA)**
   - **Functionality**: VPA automatically adjusts the CPU and memory requests and limits for containers in a pod based on actual usage, ensuring that the pods have the right amount of resources without over-provisioning or under-provisioning.
   - **Example**: You have a pod running a machine learning model inference service. Initially, you allocate 1 CPU and 2GB memory to the pod. Over time, VPA observes that the service needs more memory due to increased load, so it automatically adjusts the pod’s memory request to 4GB. If the load decreases, VPA can reduce the memory allocation back down.
   - **Command to create VPA**:
     ```yaml
     apiVersion: autoscaling.k8s.io/v1
     kind: VerticalPodAutoscaler
     metadata:
       name: <vpa-name>
     spec:
       targetRef:
         apiVersion: "apps/v1"
         kind:       Deployment
         name:       <deployment-name>
       updatePolicy:
         updateMode: "Auto"
     ```

### 3. **Cluster Autoscaler**
   - **Functionality**: Cluster Autoscaler adjusts the number of nodes in a Kubernetes cluster. It adds nodes when pods fail to schedule due to insufficient resources and removes nodes when they are underutilized for a certain period.
   - **Example**: Consider a Kubernetes cluster running on AWS with 3 nodes. If a new workload is introduced that requires more resources than the current nodes can provide, the Cluster Autoscaler will add an additional node to the cluster. Conversely, if workloads decrease and one of the nodes becomes idle, Cluster Autoscaler may remove that node to save costs.
   - **Configuring Cluster Autoscaler**: Cluster Autoscaler is typically configured when the cluster is set up, especially when using managed Kubernetes services like EKS, GKE, or AKS.

### Summary
- **HPA** scales the number of pods based on metrics like CPU or memory.
- **VPA** adjusts the resource requests and limits for containers within pods.
- **Cluster Autoscaler** scales the number of nodes in the cluster based on pod requirements.

These autoscaling mechanisms work together to maintain an efficient, responsive, and cost-effective Kubernetes environment.


