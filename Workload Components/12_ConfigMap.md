# 📘 What is a ConfigMap in Kubernetes?

- **ConfigMap** is a Kubernetes object that allows you to **store non-confidential data** (like configuration settings, environment variables, app settings) in **key-value pairs**.
- It **decouples configuration artifacts** from application code.
- This way, **you don't have to bake config** into your Docker images or deployments.

👉 Think of it as **externalized app configuration** — *"Change settings without touching your app code."*

---

# 📍 Why is ConfigMap important in Realtime Projects?

- **Different Environments** (dev, test, prod) = Different settings.
- **App Portability**: Same Docker image, different configs.
- **Rolling Updates**: Update config separately without restarting pods if needed.
- **Team Separation**: Dev team focuses on app code, Ops team manages configs.
- **Microservices Setup**: Each service reads its own config, even if deployed together.

---

# 🛠️ Realtime Example Scenario

Imagine you have a **Python Flask Application** running inside Kubernetes.  
It needs:
- A **database URL**
- **Logging level** (INFO, DEBUG, etc.)

Instead of hardcoding these values, you want the app to **read from environment variables**.

✅ We will store these environment variables inside a **ConfigMap** and inject them into the app at runtime.

---

# 📋 Step-by-Step Implementation

### 1. Create a ConfigMap

We create a ConfigMap YAML file, say `app-config.yaml`:

```yaml
apiVersion: v1
kind: ConfigMap
metadata:
  name: app-config
data:
  DATABASE_URL: "postgresql://user:password@dbserver:5432/mydb"
  LOG_LEVEL: "DEBUG"
```

✅ **Outcome:** ConfigMap object `app-config` created inside Kubernetes cluster.

---

### 2. Apply the ConfigMap

```bash
kubectl apply -f app-config.yaml
```

✅ **Outcome:** ConfigMap deployed.

---

### 3. Create a Deployment that uses the ConfigMap

Here’s a `deployment.yaml` where we **inject the ConfigMap values as environment variables**:

```yaml
apiVersion: apps/v1
kind: Deployment
metadata:
  name: flask-app
spec:
  replicas: 2
  selector:
    matchLabels:
      app: flask-app
  template:
    metadata:
      labels:
        app: flask-app
    spec:
      containers:
      - name: flask-container
        image: yourdockerhubusername/flask-app:latest
        ports:
        - containerPort: 5000
        env:
        - name: DATABASE_URL
          valueFrom:
            configMapKeyRef:
              name: app-config
              key: DATABASE_URL
        - name: LOG_LEVEL
          valueFrom:
            configMapKeyRef:
              name: app-config
              key: LOG_LEVEL
```

✅ **Outcome:** Flask application running in Kubernetes with `DATABASE_URL` and `LOG_LEVEL` environment variables injected.

---

### 4. Verify Everything

Check if ConfigMap is created:

```bash
kubectl get configmaps
kubectl describe configmap app-config
```

Check environment variables inside the pod:

```bash
kubectl exec -it <pod-name> -- env | grep DATABASE_URL
kubectl exec -it <pod-name> -- env | grep LOG_LEVEL
```

✅ **Outcome:** App container has those environment variables available.

---

### 5. (Optional) Update ConfigMap Dynamically

Suppose you want to change the logging level from `DEBUG` to `INFO`.

You can **edit** the ConfigMap:

```bash
kubectl edit configmap app-config
```

**BUT**: Pods won't automatically reload updated ConfigMaps!  
You need to either:
- Restart pods manually.
- Or use advanced techniques like **mounting ConfigMaps as volumes** and watching for changes.

---

# 🎯 Real World Final Outcome

| Feature | How ConfigMaps Help |
|:--------|:--------------------|
| Environment-specific configuration | Switch settings between Dev/QA/Prod without changing app code |
| Externalized configuration | Clean separation between config and application |
| Easy to update | Modify config without rebuilding images |
| Safe rollbacks | Use `kubectl rollout undo deployment/flask-app` if new configs break things |

---

# ✨ Bonus: Mount ConfigMap as Volume Example

Instead of injecting as env vars, you can mount it as files.

Example:

```yaml
volumeMounts:
- name: config-volume
  mountPath: /etc/config
volumes:
- name: config-volume
  configMap:
    name: app-config
```

✅ Then inside your container, `/etc/config/DATABASE_URL` will be a file containing your value.

---

# 🧠 Key Points to Remember for Interview or Project

- ConfigMaps are for **non-secret** config. (Use **Secrets** for passwords.)
- ConfigMaps can be **used as env vars or files**.
- **Updating a ConfigMap doesn't automatically update running pods**.
- Helps in **12-factor app design** (externalizing config).

---
