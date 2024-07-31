## What is Kubernetes Ingress?

Kubernetes Ingress is a resource that manages external access to services within a Kubernetes cluster, typically HTTP and HTTPS. It provides a way to expose services externally and defines rules for routing traffic to the appropriate services based on the request's URL path or host. Ingress can handle load balancing, SSL termination, and name-based virtual hosting.

### Key Concepts of Kubernetes Ingress:

1. **Ingress Resource**:
   - Defines the rules and configurations for routing external HTTP and HTTPS traffic to services within the cluster.
   - Specifies how the traffic should be directed to the appropriate services based on URL paths or hostnames.

2. **Ingress Controller**:
   - A software component responsible for fulfilling Ingress rules.
   - Watches for changes to Ingress resources and configures the necessary load balancer and proxy settings.
   - Common Ingress controllers include NGINX, Traefik, and HAProxy.

3. **Ingress Rules**:
   - Specify how incoming requests should be routed.
   - Include hostnames, paths, and backends (services).

4. **Backend Services**:
   - The target services that receive traffic based on the Ingress rules.

5. **TLS/SSL Termination**:
   - Ingress can handle SSL termination, managing HTTPS traffic and decrypting it before passing it to the backend services.

### Example: Creating Two Deployments and Exposing Them Using Ingress

Let's create two deployments with two applications and configure an Ingress resource to manage their access.

### Step-by-Step Guide

#### 1. Set Up a Kubernetes Cluster

Ensure you have a running Kubernetes cluster and the `kubectl` command-line tool configured to interact with it.

#### 2. Deploy Two Applications

Create two deployments with their respective services.

**Deployment and Service for Application 1:**

```yaml
# app1-deployment.yaml
apiVersion: apps/v1
kind: Deployment
metadata:
  name: app1-deployment
spec:
  replicas: 2
  selector:
    matchLabels:
      app: app1
  template:
    metadata:
      labels:
        app: app1
    spec:
      containers:
      - name: app1
        image: nginx
        ports:
        - containerPort: 80

---

# app1-service.yaml
apiVersion: v1
kind: Service
metadata:
  name: app1-service
spec:
  selector:
    app: app1
  ports:
  - protocol: TCP
    port: 80
    targetPort: 80
  type: ClusterIP
```

**Deployment and Service for Application 2:**

```yaml
# app2-deployment.yaml
apiVersion: apps/v1
kind: Deployment
metadata:
  name: app2-deployment
spec:
  replicas: 2
  selector:
    matchLabels:
      app: app2
  template:
    metadata:
      labels:
        app: app2
    spec:
      containers:
      - name: app2
        image: httpd
        ports:
        - containerPort: 80

---

# app2-service.yaml
apiVersion: v1
kind: Service
metadata:
  name: app2-service
spec:
  selector:
    app: app2
  ports:
  - protocol: TCP
    port: 80
    targetPort: 80
  type: ClusterIP
```

Apply these configurations:

```bash
kubectl apply -f app1-deployment.yaml
kubectl apply -f app1-service.yaml
kubectl apply -f app2-deployment.yaml
kubectl apply -f app2-service.yaml
```

#### 3. Set Up Ingress Controller

If you don't have an Ingress controller installed, you can use the NGINX Ingress controller. Install it using the following command:

```bash
kubectl apply -f https://raw.githubusercontent.com/kubernetes/ingress-nginx/main/deploy/static/provider/cloud/deploy.yaml
```

#### 4. Create an Ingress Resource

Define an Ingress resource to route traffic to the two applications based on URL paths.

```yaml
# ingress-resource.yaml
apiVersion: networking.k8s.io/v1
kind: Ingress
metadata:
  name: app-ingress
  annotations:
    nginx.ingress.kubernetes.io/rewrite-target: /
spec:
  rules:
  - host: myapp.local
    http:
      paths:
      - path: /app1
        pathType: Prefix
        backend:
          service:
            name: app1-service
            port:
              number: 80
      - path: /app2
        pathType: Prefix
        backend:
          service:
            name: app2-service
            port:
              number: 80
```

Apply the Ingress resource:

```bash
kubectl apply -f ingress-resource.yaml
```

#### 5. Update `/etc/hosts`

For testing purposes, you can add the following entry to your `/etc/hosts` file to map `myapp.local` to your cluster's external IP:

```plaintext
<your-cluster-ip> myapp.local
```

#### 6. Test the Ingress Configuration

Open a browser and navigate to the following URLs to access the applications:

- `http://myapp.local/app1` for Application 1
- `http://myapp.local/app2` for Application 2

You should see the default pages for NGINX and HTTPD, respectively.

### Notes

- **Ingress Annotations**: Annotations in the Ingress resource can provide additional configurations like URL rewrites, load balancing algorithms, and SSL settings.
- **Ingress Controller Specifics**: Different Ingress controllers may have specific annotations and configurations.
- **Security**: Always ensure proper security configurations for production environments, such as TLS/SSL encryption and proper authentication mechanisms.

This setup demonstrates how Kubernetes Ingress can simplify and manage external access to multiple applications running within a cluster.
