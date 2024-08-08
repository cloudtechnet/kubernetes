# Kubernetes StatefulSet

A **Kubernetes StatefulSet** is a Kubernetes API object used to manage stateful applications. Unlike a Deployment, which is suitable for stateless applications, StatefulSet is designed to manage stateful applications that require stable, unique network identifiers and stable, persistent storage.

### Key Features of StatefulSets:
1. **Stable, unique network identifiers**: Each pod in a StatefulSet has a stable hostname that persists through rescheduling.
2. **Stable, persistent storage**: Each pod in a StatefulSet can be assigned a persistent storage volume that retains data across pod rescheduling.
3. **Ordered, graceful deployment and scaling**: Pods are created in order, and scaling operations (up or down) are performed in a controlled sequence.
4. **Ordered, automated rolling updates**: Updates are applied in order, ensuring one pod is updated and running before proceeding to the next.

### Example with MySQL and PostgreSQL

#### MySQL StatefulSet Example

1. **Persistent Volume Claims (PVCs) for MySQL**:
```yaml
apiVersion: v1
kind: PersistentVolumeClaim
metadata:
  name: mysql-pv-claim
spec:
  accessModes:
    - ReadWriteOnce
  resources:
    requests:
      storage: 10Gi
```

2. **ConfigMap for MySQL**:
```yaml
apiVersion: v1
kind: ConfigMap
metadata:
  name: mysql-config
data:
  mysql.cnf: |
    [mysqld]
    sql_mode=STRICT_TRANS_TABLES,NO_ENGINE_SUBSTITUTION
```

3. **MySQL StatefulSet**:
```yaml
apiVersion: apps/v1
kind: StatefulSet
metadata:
  name: mysql
spec:
  serviceName: "mysql"
  replicas: 3
  selector:
    matchLabels:
      app: mysql
  template:
    metadata:
      labels:
        app: mysql
    spec:
      containers:
      - name: mysql
        image: mysql:5.7
        ports:
        - containerPort: 3306
          name: mysql
        env:
        - name: MYSQL_ROOT_PASSWORD
          value: rootpassword
        volumeMounts:
        - name: mysql-persistent-storage
          mountPath: /var/lib/mysql
        - name: config-volume
          mountPath: /etc/mysql/conf.d
      volumes:
      - name: config-volume
        configMap:
          name: mysql-config
  volumeClaimTemplates:
  - metadata:
      name: mysql-persistent-storage
    spec:
      accessModes: [ "ReadWriteOnce" ]
      resources:
        requests:
          storage: 10Gi
```

#### PostgreSQL StatefulSet Example

1. **Persistent Volume Claims (PVCs) for PostgreSQL**:
```yaml
apiVersion: v1
kind: PersistentVolumeClaim
metadata:
  name: postgres-pv-claim
spec:
  accessModes:
    - ReadWriteOnce
  resources:
    requests:
      storage: 10Gi
```

2. **ConfigMap for PostgreSQL**:
```yaml
apiVersion: v1
kind: ConfigMap
metadata:
  name: postgres-config
data:
  postgres.conf: |
    max_connections = 200
    shared_buffers = 256MB
    effective_cache_size = 768MB
```

3. **PostgreSQL StatefulSet**:
```yaml
apiVersion: apps/v1
kind: StatefulSet
metadata:
  name: postgres
spec:
  serviceName: "postgres"
  replicas: 3
  selector:
    matchLabels:
      app: postgres
  template:
    metadata:
      labels:
        app: postgres
    spec:
      containers:
      - name: postgres
        image: postgres:13
        ports:
        - containerPort: 5432
          name: postgres
        env:
        - name: POSTGRES_PASSWORD
          value: mypassword
        volumeMounts:
        - name: postgres-persistent-storage
          mountPath: /var/lib/postgresql/data
        - name: config-volume
          mountPath: /etc/postgresql/postgresql.conf
          subPath: postgres.conf
      volumes:
      - name: config-volume
        configMap:
          name: postgres-config
  volumeClaimTemplates:
  - metadata:
      name: postgres-persistent-storage
    spec:
      accessModes: [ "ReadWriteOnce" ]
      resources:
        requests:
          storage: 10Gi
```

### Explanation
1. **Persistent Volume Claims (PVCs)**: Ensure that each replica of MySQL and PostgreSQL has its own storage.
2. **ConfigMap**: Stores configuration data for MySQL and PostgreSQL.
3. **StatefulSet**: Defines the stateful application with stable network identifiers, persistent storage, and ordered deployment and scaling.

