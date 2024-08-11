# Kubernetes DaemonSets

**Kubernetes DaemonSets** ensure that a copy of a specific pod is running on all (or some) nodes in a Kubernetes cluster. They are typically used for deploying system daemons, such as log collectors, monitoring agents, or other services that need to run on every node. When a new node is added to the cluster, the DaemonSet automatically adds the specified pod to that node. If a node is removed, the pod is also removed.

**Use Cases:**
- Log collection: Running log collectors like Fluentd or Filebeat on every node.
- Monitoring: Running monitoring agents like Prometheus Node Exporter on every node.
- Networking: Running network plugins like CNI plugins.

**Example: Deploying Prometheus Node Exporter Using DaemonSet**

Here's a simple example of a DaemonSet configuration for deploying Prometheus Node Exporter, which is used to collect system metrics from nodes:

```yaml
apiVersion: apps/v1
kind: DaemonSet
metadata:
  name: node-exporter
  namespace: monitoring
spec:
  selector:
    matchLabels:
      app: node-exporter
  template:
    metadata:
      labels:
        app: node-exporter
    spec:
      containers:
      - name: node-exporter
        image: prom/node-exporter:v1.3.1
        ports:
        - containerPort: 9100
          name: metrics
        resources:
          limits:
            memory: "200Mi"
            cpu: "100m"
        volumeMounts:
        - name: proc
          mountPath: /host/proc
          readOnly: true
        - name: sys
          mountPath: /host/sys
          readOnly: true
      volumes:
      - name: proc
        hostPath:
          path: /proc
      - name: sys
        hostPath:
          path: /sys
```

**Explanation:**
- **apiVersion, kind, metadata:** Define the type of resource (DaemonSet) and provide metadata like the name and namespace.
- **spec:** Contains the specification for the DaemonSet.
  - **selector:** Specifies the pod labels to which the DaemonSet applies.
  - **template:** Defines the pod's template.
    - **containers:** Specifies the container(s) to run. In this case, it runs the `node-exporter` container.
    - **volumes:** Mounts the host’s `/proc` and `/sys` directories to the container for reading system metrics.

### Prometheus
Prometheus is an open-source monitoring and alerting toolkit designed for reliability and scalability. It collects and stores metrics as time series data, with each data point labeled by a set of key-value pairs called labels. Prometheus provides powerful querying and visualization capabilities, and it's widely used in cloud-native environments, particularly with Kubernetes.

**Key Features:**
- **Multi-dimensional data model:** Metrics are identified by a combination of metric names and key-value pairs (labels).
- **PromQL:** Prometheus Query Language used to query metrics.
- **Service discovery:** Automatically discovers targets in dynamic environments like Kubernetes.
- **Alerting:** Integrates with Alertmanager to handle alerts based on defined rules.

### Example of Prometheus Monitoring in Kubernetes
Prometheus is often used with Kubernetes to monitor applications, infrastructure, and services. Here’s how you might set up Prometheus to scrape metrics from a Kubernetes cluster:

1. **Prometheus Deployment:** Deploy Prometheus in the Kubernetes cluster.
2. **Service Discovery:** Use Kubernetes service discovery to automatically find services to monitor.
3. **Scrape Configurations:** Configure Prometheus to scrape metrics from different services.

**Prometheus Configuration Snippet:**
```yaml
scrape_configs:
  - job_name: 'kubernetes-apiservers'
    kubernetes_sd_configs:
    - role: endpoints
    relabel_configs:
    - source_labels: [__meta_kubernetes_namespace, __meta_kubernetes_service_name, __meta_kubernetes_endpoint_port_name]
      action: keep
      regex: default;kubernetes;https

  - job_name: 'node-exporter'
    kubernetes_sd_configs:
    - role: node
    relabel_configs:
    - action: labelmap
      regex: __meta_kubernetes_node_label_(.+)
    - target_label: __address__
      replacement: kubernetes.default.svc:9100
```

**Explanation:**
- **job_name:** Defines the job's name, like 'kubernetes-apiservers' or 'node-exporter'.
- **kubernetes_sd_configs:** Enables service discovery in Kubernetes.
- **relabel_configs:** Modifies or removes labels before sending them to Prometheus.

**Grafana Integration:**
Prometheus is often used with Grafana, a popular visualization tool, to create dashboards for visualizing metrics collected by Prometheus.

This combination of DaemonSets and Prometheus is a powerful way to monitor and manage the health and performance of your Kubernetes cluster.
