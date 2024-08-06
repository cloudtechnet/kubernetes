# Kubernetes Namespaces

**Namespaces** in Kubernetes are a way to divide cluster resources between multiple users (via resource quota). They provide a mechanism for isolating groups of resources within a single cluster. Namespaces can be used to create environments for development, testing, and production within the same cluster. Each namespace can have its own set of policies, resource quotas, and limits.

#### Key Features of Namespaces:
1. **Isolation**: Resources in one namespace are invisible and inaccessible from other namespaces unless explicitly allowed.
2. **Resource Management**: Administrators can set resource quotas and limits for each namespace to ensure fair resource distribution.
3. **Access Control**: Different namespaces can have different access control policies.
4. **Scalability**: Namespaces help in managing resources at scale by organizing them logically.
5. **Multi-tenancy**: Useful for organizations to host multiple teams or projects within a single cluster while maintaining isolation.

#### Namespace Commands:
- **Create Namespace**: `kubectl create namespace <namespace-name>`
- **Delete Namespace**: `kubectl delete namespace <namespace-name>`
- **List Namespaces**: `kubectl get namespaces`
- **Use a Namespace**: `kubectl config set-context --current --namespace=<namespace-name>`

### Realtime Examples

#### Example 1: Development and Production Environments

**Scenario**: A company has a Kubernetes cluster that they use for both development and production environments. They want to ensure that the resources used for development do not interfere with the production resources.

1. **Create Namespaces**:
   ```sh
   kubectl create namespace development
   kubectl create namespace production
   ```

2. **Deploy Applications**:
   - Development environment:
     ```sh
     kubectl apply -f dev-app-deployment.yaml -n development
     ```
   - Production environment:
     ```sh
     kubectl apply -f prod-app-deployment.yaml -n production
     ```

3. **Resource Quotas**:
   - Development environment:
     ```yaml
     apiVersion: v1
     kind: ResourceQuota
     metadata:
       name: dev-quota
       namespace: development
     spec:
       hard:
         cpu: "20"
         memory: 50Gi
         pods: "10"
     ```
     ```sh
     kubectl apply -f dev-quota.yaml
     ```
   - Production environment:
     ```yaml
     apiVersion: v1
     kind: ResourceQuota
     metadata:
       name: prod-quota
       namespace: production
     spec:
       hard:
         cpu: "100"
         memory: 200Gi
         pods: "50"
     ```
     ```sh
     kubectl apply -f prod-quota.yaml
     ```

By setting up these namespaces and quotas, the development team can work without affecting the production environment, ensuring stability and isolation.

#### Example 2: Multi-Tenant Applications

**Scenario**: An organization is running a SaaS platform and needs to deploy isolated environments for each client. Each client should have its own namespace to ensure data and resource isolation.

1. **Create Namespaces**:
   ```sh
   kubectl create namespace clientA
   kubectl create namespace clientB
   ```

2. **Deploy Applications for Each Client**:
   - Client A:
     ```sh
     kubectl apply -f clientA-app-deployment.yaml -n clientA
     ```
   - Client B:
     ```sh
     kubectl apply -f clientB-app-deployment.yaml -n clientB
     ```

3. **Network Policies**:
   - Restricting inter-namespace traffic to ensure isolation:
     ```yaml
     apiVersion: networking.k8s.io/v1
     kind: NetworkPolicy
     metadata:
       name: default-deny
       namespace: clientA
     spec:
       podSelector: {}
       policyTypes:
       - Ingress
       - Egress
     ```
     ```sh
     kubectl apply -f clientA-network-policy.yaml
     ```

By using namespaces and network policies, the organization can ensure that each client's environment is isolated and secure, preventing data leaks and resource contention.

### Conclusion

Kubernetes namespaces are a powerful feature that helps in organizing and isolating resources within a cluster. They are essential for managing complex environments, ensuring security, and enabling multi-tenancy. By properly utilizing namespaces, organizations can achieve better resource management and operational efficiency in their Kubernetes clusters.
