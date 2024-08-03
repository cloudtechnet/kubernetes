# Scenario based Interview Questions and Answers

### Scenario 1: Pod CrashLoopBackOff

**Question:** Your application's Pods are in a `CrashLoopBackOff` state. How do you troubleshoot and resolve this issue?

**Answer:**

1. **Check Pod Status:**
   Use the `kubectl describe pod <pod-name>` command to get detailed information about the Pod's status, events, and reasons for the crash.
   ```bash
   kubectl describe pod <pod-name>
   ```

2. **View Pod Logs:**
   Inspect the logs of the crashing container to identify any error messages or issues. Use the `kubectl logs <pod-name> -c <container-name>` command.
   ```bash
   kubectl logs <pod-name> -c <container-name>
   ```

3. **Identify the Issue:**
   Common reasons for `CrashLoopBackOff` include:
   - **Application Error**: The application inside the container has an error causing it to crash.
   - **Configuration Error**: Incorrect configurations, such as environment variables or secrets, leading to startup failures.
   - **Resource Limits**: The container might be hitting its resource limits (CPU, memory).

4. **Check Events:**
   Review the events in the namespace to see if there are any clues about why the Pod is failing.
   ```bash
   kubectl get events --sort-by=.metadata.creationTimestamp
   ```

5. **Fix the Issue:**
   Based on the logs and error messages, make the necessary corrections:
   - **Application Code**: If the issue is within the application code, debug and fix the application.
   - **Configuration**: Update the Pod's configuration in the deployment YAML.
   - **Resource Limits**: Adjust the resource requests and limits if the container is being throttled or killed due to resource constraints.

6. **Redeploy the Pod:**
   After making the necessary changes, redeploy the Pod.
   ```bash
   kubectl apply -f <deployment-yaml>
   ```

**Example:** If the logs show an error related to a missing environment variable, update the deployment YAML to include the necessary environment variable:
```yaml
spec:
  containers:
  - name: mycontainer
    image: myimage
    env:
    - name: MY_ENV_VAR
      value: "myvalue"
```

### Scenario 2: Node Not Ready

**Question:** One of the nodes in your Kubernetes cluster is in a `NotReady` state. How do you troubleshoot and resolve this issue?

**Answer:**

1. **Check Node Status:**
   Use the `kubectl get nodes` command to check the status of the nodes in the cluster.
   ```bash
   kubectl get nodes
   ```

2. **Describe the Node:**
   Use the `kubectl describe node <node-name>` command to get detailed information about the node's status and events.
   ```bash
   kubectl describe node <node-name>
   ```

3. **Review Node Events:**
   Look for any recent events that might indicate why the node is not ready.
   ```bash
   kubectl get events --field-selector involvedObject.kind=Node,involvedObject.name=<node-name>
   ```

4. **Check Node Conditions:**
   Pay attention to the `Ready` condition and other conditions like `DiskPressure`, `MemoryPressure`, `NetworkUnavailable`.
   
5. **Log into the Node:**
   SSH into the node to check system logs and services.
   ```bash
   ssh <node-ip>
   ```

6. **Check Kubelet Logs:**
   Inspect the kubelet logs to identify any issues with the node agent.
   ```bash
   journalctl -u kubelet
   ```

7. **Common Issues:**
   - **Network Issues**: Check the network connectivity and ensure the node can communicate with the master nodes.
   - **Disk Pressure**: Verify if the node has enough disk space.
     ```bash
     df -h
     ```
   - **Memory Pressure**: Ensure the node has sufficient memory.
     ```bash
     free -m
     ```

8. **Restart Services:**
   If necessary, restart the kubelet service and other critical services.
   ```bash
   systemctl restart kubelet
   ```

9. **Drain and Rejoin Node:**
   If the node is still not ready, drain the node and rejoin it to the cluster.
   ```bash
   kubectl drain <node-name> --ignore-daemonsets
   kubectl delete node <node-name>
   # Rejoin the node (commands may vary based on the setup)
   ```

**Example:** If the node has DiskPressure, clean up unnecessary files or logs to free up space.

### Scenario 3: High Latency in Service

**Question:** Users are experiencing high latency when accessing your service. How do you troubleshoot and resolve this issue?

**Answer:**

1. **Check Service Performance:**
   Use `kubectl get svc <service-name>` to check the service details.
   ```bash
   kubectl get svc <service-name>
   ```

2. **Inspect Pod Metrics:**
   Check the resource usage of the Pods behind the service using metrics tools like Prometheus and Grafana.

3. **Check Pod Logs:**
   Inspect the logs of the Pods to identify any slow operations or errors.
   ```bash
   kubectl logs <pod-name>
   ```

4. **Review Deployment Configuration:**
   Ensure that the Pods have adequate resource requests and limits.
   ```yaml
   resources:
     requests:
       memory: "128Mi"
       cpu: "250m"
     limits:
       memory: "256Mi"
       cpu: "500m"
   ```

5. **Check Node Health:**
   Ensure the nodes hosting the Pods are not under resource pressure (CPU, memory, disk).

6. **Load Balancer Configuration:**
   Verify the configuration of the load balancer (if using one) to ensure it's distributing traffic evenly.

7. **Network Latency:**
   Check for network latency between nodes and the clients using tools like `ping` and `traceroute`.

8. **Scaling the Deployment:**
   If the current number of replicas is insufficient, scale the deployment to handle more traffic.
   ```bash
   kubectl scale deployment <deployment-name> --replicas=<desired-replicas>
   ```

9. **Investigate External Dependencies:**
   If the application relies on external services (e.g., databases), ensure those services are performing well and not causing bottlenecks.

**Example:** If the logs indicate that the database queries are slow, investigate the database performance and optimize queries or database configuration.

### Scenario 4: Persistent Volume Issues

**Question:** Your application is unable to mount a Persistent Volume (PV). How do you troubleshoot and resolve this issue?

**Answer:**

1. **Check PVC and PV Status:**
   Use `kubectl get pvc` and `kubectl get pv` to check the status of the Persistent Volume Claim (PVC) and Persistent Volume.
   ```bash
   kubectl get pvc
   kubectl get pv
   ```

2. **Describe PVC and PV:**
   Use `kubectl describe pvc <pvc-name>` and `kubectl describe pv <pv-name>` to get detailed information.
   ```bash
   kubectl describe pvc <pvc-name>
   kubectl describe pv <pv-name>
   ```

3. **Verify StorageClass:**
   Ensure that the StorageClass specified in the PVC matches the available StorageClass.
   ```yaml
   spec:
     storageClassName: my-storage-class
   ```

4. **Check Events:**
   Review the events related to the PVC and PV for any error messages.
   ```bash
   kubectl get events --field-selector involvedObject.kind=PersistentVolumeClaim,involvedObject.name=<pvc-name>
   ```

5. **Node Accessibility:**
   Ensure that the node where the Pod is scheduled can access the storage backend.

6. **Inspect Storage Backend:**
   Verify the status of the storage backend (e.g., NFS server, cloud storage) to ensure it's operational.

7. **Correct Access Modes:**
   Ensure the access modes (ReadWriteOnce, ReadOnlyMany, ReadWriteMany) of the PVC match those supported by the PV.
   ```yaml
   spec:
     accessModes:
     - ReadWriteOnce
   ```

8. **Check Mount Paths:**
   Ensure the mount paths in the Pod spec are correct.
   ```yaml
   volumeMounts:
   - mountPath: "/data"
     name: my-volume
   ```

**Example:** If the PV is stuck in the `Released` state, it may need to be manually reclaimed or deleted and recreated.

```bash
kubectl delete pv <pv-name>
kubectl apply -f <pv-definition.yaml>
```

### Scenario 5: Service Unavailable

**Question:** Your service is returning `503 Service Unavailable` errors. How do you troubleshoot and resolve this issue?

**Answer:**

1. **Check Service and Endpoints:**
   Use `kubectl get svc <service-name>` and `kubectl get endpoints <service-name>` to check the service and its endpoints.
   ```bash
   kubectl get svc <service-name>
   kubectl get endpoints <service-name>
   ```

2. **Describe Service:**
   Use `kubectl describe svc <service-name>` to get detailed information about the service.
   ```bash
   kubectl describe svc <service-name>
   ```

3. **Pod Readiness:**
   Ensure that the Pods behind the service
