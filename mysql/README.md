# Kubernetes StatefulSet Guide - MySQL Database

## Overview
This guide teaches you how to deploy a MySQL database using StatefulSets in Kubernetes. A StatefulSet is special because it maintains stable identities for your pods, making it perfect for databases that need to remember their data.

**Think of it like this:** Imagine you have three friends named Alice-1, Alice-2, and Alice-3. Even if Alice-1 goes home and comes back, she's still Alice-1 with all her memories intact. That's how StatefulSets work - each pod keeps its identity and data!

---

## What is a StatefulSet?

### Simple Explanation
A StatefulSet is like a special manager for pods that need:
- A **fixed name** (like mysql-0, mysql-1, mysql-2)
- **Persistent storage** (data that doesn't disappear when the pod restarts)
- **Ordered startup** (pods start one by one, not all at once)

### When to Use StatefulSets
- **Databases**: MySQL, PostgreSQL, MongoDB
- **Message queues**: Kafka, RabbitMQ
- **Distributed systems**: Elasticsearch, ZooKeeper
- Any application that needs stable network identities or persistent storage

### StatefulSet vs Deployment
| Feature | StatefulSet | Deployment |
|---------|-------------|------------|
| Pod Names | Predictable (mysql-0, mysql-1) | Random (mysql-abc123) |
| Storage | Persistent per pod | Shared or temporary |
| Startup Order | Sequential | All at once |
| Best For | Databases | Stateless apps |

---

## Prerequisites
- A running Kubernetes cluster (Kind, Minikube, or any cluster)
- kubectl installed and configured
- Basic understanding of Kubernetes concepts

---

## Architecture Overview

Our MySQL deployment consists of five key components:

1. **Namespace**: Isolated environment for MySQL resources
2. **ConfigMap**: Non-sensitive configuration data (database name)
3. **Secret**: Sensitive data (root password)
4. **Service**: Network access to StatefulSet pods
5. **StatefulSet**: Manages MySQL pods with persistent storage

---

## Step 1: Create the Namespace

### What is a Namespace?
A namespace is like a separate room in your Kubernetes house. It keeps your MySQL resources organized and separated from other applications.

### Command
```bash
kubectl apply -f namespace.yml
```

### What does this command do?
- Reads your `namespace.yml` file
- Creates a new namespace in your cluster
- Provides an isolated environment for your MySQL resources

### Why use namespaces?
- **Organization**: Keep related resources together
- **Isolation**: Prevent naming conflicts with other projects
- **Resource management**: Set limits and quotas per namespace
- **Access control**: Manage permissions at the namespace level

### Expected Output
```
namespace/mysql created
```

---

## Step 2: Create the ConfigMap

### What is a ConfigMap?
A ConfigMap stores non-sensitive configuration data in key-value pairs. It's like a settings file that your application can read.

### Command
```bash
kubectl apply -f configmap.yml
```

### What does this command do?
- Creates a ConfigMap named `mysql-config-map` in the mysql namespace
- Stores the database name (`devops`) that MySQL will automatically create
- Makes this configuration available to your pods

### Why use ConfigMaps?
- **Separation of concerns**: Keep configuration separate from application code
- **Easy updates**: Change configuration without rebuilding container images
- **Reusability**: Share configuration across multiple pods
- **Non-sensitive data**: Perfect for database names, feature flags, or URLs

### Your ConfigMap Structure
```yaml
apiVersion: v1
kind: ConfigMap
metadata:
  name: mysql-config-map
  namespace: mysql
data:
  MYSQL_DATABASE: devops
```

### Expected Output
```
configmap/mysql-config-map created
```

### Verify the ConfigMap
```bash
kubectl get configmap -n mysql
```

You should see:
```
NAME               DATA   AGE
mysql-config-map   1      10s
```

---

## Step 3: Create the Secret

### What is a Secret?
A Secret stores sensitive data like passwords, tokens, or SSH keys in base64-encoded format. It's more secure than putting passwords directly in your YAML files.

### Command
```bash
kubectl apply -f secrets.yml
```

### What does this command do?
- Creates a Secret named `mysql-secret` in the mysql namespace
- Stores the base64-encoded root password for MySQL
- Keeps sensitive data encrypted at rest (if encryption is enabled in your cluster)

### Why use Secrets?
- **Security**: Passwords aren't visible in plain text in configuration files
- **Access control**: You can restrict who can view Secrets
- **Best practice**: Never hardcode passwords in YAML files
- **Rotation**: Easy to update passwords without changing application code

### Your Secret Structure
```yaml
apiVersion: v1
kind: Secret
metadata:
  name: mysql-secret
  namespace: mysql
type: Opaque
data:
  MYSQL_ROOT_PASSWORD: cm9vdAo=
```

### Understanding Base64 Encoding
The password is base64-encoded, not encrypted. The value `cm9vdAo=` decodes to `root`.

### How to encode your own password
```bash
echo -n 'your-password' | base64
```

**Important**: Use `-n` flag to avoid encoding a newline character!

### Expected Output
```
secret/mysql-secret created
```

### Verify the Secret
```bash
kubectl get secret -n mysql
```

You should see:
```
NAME           TYPE     DATA   AGE
mysql-secret   Opaque   1      5s
```

### View Secret Details (without revealing values)
```bash
kubectl describe secret mysql-secret -n mysql
```

### Decode the Secret (for verification)
```bash
kubectl get secret mysql-secret -n mysql -o jsonpath='{.data.MYSQL_ROOT_PASSWORD}' | base64 --decode
```

---

## Step 4: Create the Service

### What is a Kubernetes Service?
A service is like a phone number for your pods. Even if pods get replaced, the service provides a stable way to reach them.

### Command
```bash
kubectl apply -f service.yml
```

### What does this command do?
- Creates a headless service (ClusterIP: None)
- Provides network identity to each StatefulSet pod
- Enables pod-to-pod communication using DNS names

### Why create the service before StatefulSet?
StatefulSets use the service to give each pod a unique DNS name like:
- `mysql-statefulset-0.mysql-service.mysql.svc.cluster.local`
- `mysql-statefulset-1.mysql-service.mysql.svc.cluster.local`

### Expected Output
```
service/mysql-service created
```

### Use Cases
- **Database replication**: Master and slave pods need stable addresses
- **Cluster communication**: Pods need to find each other reliably
- **Client connections**: Applications connect to specific database instances

---

## Step 5: Create the StatefulSet

### Command
```bash
kubectl apply -f statefulset.yml
```

### What does this command do?
- Reads your `statefulset.yml` configuration
- Creates MySQL pods one by one in order (0, then 1, then 2)
- Attaches persistent storage to each pod
- Injects environment variables from ConfigMap and Secret

### Key Changes in Updated StatefulSet
The StatefulSet now uses ConfigMap and Secret instead of hardcoded values:

**Before (hardcoded values):**
```yaml
env:
  - name: MYSQL_ROOT_PASSWORD
    value: root
  - name: MYSQL_DATABASE
    value: devops
```

**After (using ConfigMap and Secret):**
```yaml
env:
  - name: MYSQL_ROOT_PASSWORD
    valueFrom:
      secretKeyRef:
        name: mysql-secret
        key: MYSQL_ROOT_PASSWORD
  - name: MYSQL_DATABASE
    valueFrom: 
      configMapKeyRef:
        name: mysql-config-map
        key: MYSQL_DATABASE
```

### Why this approach is better?
- **Security**: Passwords are stored in Secrets, not visible in YAML files
- **Flexibility**: Change configuration without modifying StatefulSet
- **Best practice**: Follows Kubernetes security recommendations
- **Maintainability**: Easier to manage in production environments

### Expected Output
```
statefulset.apps/mysql-statefulset created
```

---

## Step 6: Verify All Resources

### Check All Resources in Namespace
```bash
kubectl get all -n mysql
```

### Expected Output
```
NAME                      READY   STATUS    RESTARTS   AGE
pod/mysql-statefulset-0   1/1     Running   0          3m8s
pod/mysql-statefulset-1   1/1     Running   0          2m21s
pod/mysql-statefulset-2   1/1     Running   0          95s

NAME                    TYPE        CLUSTER-IP   EXTERNAL-IP   PORT(S)    AGE
service/mysql-service   ClusterIP   None         <none>        3306/TCP   5m

NAME                                 READY   AGE
statefulset.apps/mysql-statefulset   3/3     3m8s
```

### Verify ConfigMap and Secret
```bash
kubectl get configmap,secret -n mysql
```

### Expected Output
```
NAME                         DATA   AGE
configmap/mysql-config-map   1      10m

NAME                  TYPE     DATA   AGE
secret/mysql-secret   Opaque   1      9m
```

### Understanding the Output
- **Pods**: All three pods are running sequentially
- **Service**: Headless service is available on port 3306
- **StatefulSet**: 3/3 replicas are ready
- **ConfigMap & Secret**: Both configuration resources are created

---

## Step 7: Verify the Pods

### Command
```bash
kubectl get pods -n mysql
```

### What does this command do?
- Lists all pods in the `mysql` namespace
- Shows their current status (Running, Pending, etc.)
- Displays how many times they've restarted

### What to look for
- **READY 1/1**: Pod is fully operational
- **STATUS Running**: Pod is working correctly
- **Sequential names**: Pods are numbered 0, 1, 2...

### Expected Output
```
NAME                  READY   STATUS    RESTARTS   AGE
mysql-statefulset-0   1/1     Running   0          3m8s
mysql-statefulset-1   1/1     Running   0          2m21s
mysql-statefulset-2   1/1     Running   0          95s
```

### Understanding the Output
- **mysql-statefulset-0**: The first pod (often the primary/master)
- **mysql-statefulset-1**: The second pod
- **mysql-statefulset-2**: The third pod
- Notice they started one after another, not all at once!

---

## Step 8: Check the Service

### Command
```bash
kubectl get svc -n mysql
```

**Note:** `svc` is shorthand for `service`. Both commands work the same way!

### What does this command do?
Displays all services in the mysql namespace, showing how your pods can be accessed.

### Expected Output
```
NAME            TYPE        CLUSTER-IP   EXTERNAL-IP   PORT(S)    AGE
mysql-service   ClusterIP   None         <none>        3306/TCP   23m
```

### Understanding the Output
- **TYPE ClusterIP**: Service is only accessible within the cluster
- **CLUSTER-IP None**: This is a "headless" service (no load balancing)
- **PORT 3306**: Standard MySQL port
- **None** means each pod gets its own DNS address instead of one shared IP

### Why Headless Service?
For StatefulSets, we often use headless services because:
- We need to access specific pods (like a primary database)
- Each pod has unique data or a specific role
- Direct pod-to-pod communication is required

---

## Accessing a Pod

### Step 9: List Available Pods

### Command
```bash
kubectl get pods -n mysql
```

### What does this command do?
Shows you all available MySQL pods so you can choose which one to access.

### Expected Output
```
NAME                  READY   STATUS    RESTARTS   AGE
mysql-statefulset-0   1/1     Running   0          42m
mysql-statefulset-1   1/1     Running   0          41m
mysql-statefulset-2   1/1     Running   0          40m
```

---

### Step 10: Enter the Pod Shell

### Command
```bash 
kubectl exec -it <pod-name> -n mysql -- bash
```

### Command Breakdown
- `kubectl exec`: Execute a command in a pod
- `-it`: Interactive terminal (lets you type commands)
- `<pod-name>`: The pod you want to access
- `-n mysql`: Specifies the namespace
- `-- bash`: Opens a bash shell inside the pod

### Example
```bash
kubectl exec -it mysql-statefulset-0 -n mysql -- bash
```

### What does this command do?
Opens an interactive terminal session inside the specified pod, allowing you to run commands as if you're logged into that container.

### Expected Output
```
bash-5.1# 
```

**Success!** You're now inside the MySQL pod. The prompt changed to `bash-5.1#`.

### Use Cases
- **Troubleshooting**: Check logs and configurations
- **Data inspection**: Query databases directly
- **Debugging**: Test connectivity and configurations
- **Maintenance**: Run database operations manually

---

### Step 11: Login to MySQL

Now that you're inside the pod, you need to connect to the MySQL database.

### Command
```bash
mysql -u root -p 
```

### Command Breakdown
- `mysql`: MySQL client command
- `-u root`: Login as the 'root' user
- `-p`: Prompt for password

### Finding the Password

The password is defined in your `secrets.yml` file (base64-encoded):

```yaml
data:
  MYSQL_ROOT_PASSWORD: cm9vdAo=
```

**The password is:** `root`

### How the Password Gets to the Pod

The StatefulSet configuration references the Secret:

```yaml
env:
  - name: MYSQL_ROOT_PASSWORD
    valueFrom:
      secretKeyRef:
        name: mysql-secret
        key: MYSQL_ROOT_PASSWORD
```

Kubernetes automatically decodes the base64 value and injects it as an environment variable into the pod.

### Changing the Password

If you want to use a different password:

1. Encode your new password:
```bash
echo -n 'mynewpassword' | base64
```

2. Update `secrets.yml` with the new encoded value:
```yaml
data:
  MYSQL_ROOT_PASSWORD: bXluZXdwYXNzd29yZA==
```

3. Apply the updated Secret:
```bash
kubectl apply -f secrets.yml
```

4. Restart the StatefulSet pods for changes to take effect:
```bash
kubectl rollout restart statefulset mysql-statefulset -n mysql
```

### Expected Output
```
bash-5.1# mysql -u root -p
Enter password: 
Welcome to the MySQL monitor.  Commands end with ; or \g.
Your MySQL connection id is 8
Server version: 8.0.44 MySQL Community Server - GPL

Copyright (c) 2000, 2025, Oracle and/or its affiliates.

Oracle is a registered trademark of Oracle Corporation and/or its
affiliates. Other names may be trademarks of their respective
owners.

Type 'help;' or '\h' for help. Type '\c' to clear the current input statement.

mysql> 
```

**Congratulations!** You're now logged into MySQL and can run database commands.

---

## Working with MySQL

### Step 12: View Databases

### Command
```mysql
show databases;
```

### What does this command do?
Lists all databases available in this MySQL instance.

### Expected Output
```
mysql> show databases;
+--------------------+
| Database           |
+--------------------+
| devops             |
| information_schema |
| mysql              |
| performance_schema |
| sys                |
+--------------------+
5 rows in set (0.02 sec)
```

### Why is 'devops' database there?

The database name comes from your ConfigMap:

```yaml
apiVersion: v1
kind: ConfigMap
metadata:
  name: mysql-config-map
  namespace: mysql
data:
  MYSQL_DATABASE: devops
```

The StatefulSet reads this ConfigMap:

```yaml
env:
  - name: MYSQL_DATABASE
    valueFrom: 
      configMapKeyRef:
        name: mysql-config-map
        key: MYSQL_DATABASE
```

MySQL automatically creates the 'devops' database when the container starts!

### Common MySQL Commands

Once logged in, you can use these commands:

```mysql
-- Select a database to use
USE devops;

-- Show tables in current database
SHOW TABLES;

-- Create a new database
CREATE DATABASE myapp;

-- Create a table
CREATE TABLE users (
    id INT PRIMARY KEY,
    name VARCHAR(100)
);

-- View table structure
DESCRIBE users;

-- Insert data
INSERT INTO users VALUES (1, 'Alice');

-- Query data
SELECT * FROM users;
```

---

## Exiting the Pod

### Step 13: Exit MySQL

### Command
```mysql
exit
```

### What does this command do?
Logs you out of the MySQL client and returns you to the bash shell inside the pod.

### Expected Output
```
mysql> exit
Bye
bash-5.1#
```

---

### Step 14: Exit the Pod Shell

### Command
```bash
exit
```

### What does this command do?
Closes the bash shell and returns you to your local terminal.

### Expected Output
```
bash-5.1# exit
exit
aditya@Aditya:~/devops/kubernetes/kind-cluster/mysql$ 
```

**You're back!** Notice the prompt changed back to your local machine.

---

## Managing Configuration

### Updating ConfigMap

To change the database name:

1. Edit `configmap.yml`:
```yaml
data:
  MYSQL_DATABASE: production
```

2. Apply the changes:
```bash
kubectl apply -f configmap.yml
```

3. Restart pods to use new configuration:
```bash
kubectl rollout restart statefulset mysql-statefulset -n mysql
```

### Updating Secrets

To change the root password:

1. Encode new password:
```bash
echo -n 'newpassword' | base64
```

2. Update `secrets.yml` with the encoded value

3. Apply and restart:
```bash
kubectl apply -f secrets.yml
kubectl rollout restart statefulset mysql-statefulset -n mysql
```

### View Current Configuration in Pods

Check environment variables inside a pod:

```bash
kubectl exec mysql-statefulset-0 -n mysql -- env | grep MYSQL
```

---

## Managing Pods

### Deleting a Pod

### Why Delete a Pod?
- **Testing resilience**: Verify that StatefulSet recreates the pod
- **Forcing restart**: When a pod is stuck or misbehaving
- **Updating configuration**: Sometimes requires a fresh start
- **Troubleshooting**: Eliminate problematic pods

### Step 15: Choose a Pod to Delete

### Command
```bash
kubectl get pods -n mysql
```

### Expected Output
```
NAME                  READY   STATUS    RESTARTS   AGE
mysql-statefulset-0   1/1     Running   0          1h
mysql-statefulset-1   1/1     Running   0          59m
mysql-statefulset-2   1/1     Running   0          58m
```

---

### Step 16: Delete the Pod

### Command
```bash
kubectl delete pod <pod-name> -n mysql
```

### Example
```bash
kubectl delete pod mysql-statefulset-0 -n mysql
```

### What happens when you delete a StatefulSet pod?
1. The pod is terminated immediately
2. StatefulSet controller detects the missing pod
3. **Automatically creates a new pod with the SAME name**
4. The new pod attaches to the existing persistent storage
5. ConfigMap and Secret are automatically injected again
6. Your data is preserved!

### Expected Output
```
pod "mysql-statefulset-0" deleted
```

### Verify the Recreation

Wait a few seconds and check again:

```bash
kubectl get pods -n mysql
```

You'll see `mysql-statefulset-0` is back, but with AGE showing it's newly created!

```
NAME                  READY   STATUS    RESTARTS   AGE
mysql-statefulset-0   1/1     Running   0          15s  ← New pod!
mysql-statefulset-1   1/1     Running   0          59m
mysql-statefulset-2   1/1     Running   0          58m
```

### Magic of StatefulSets!
Unlike regular Deployments that would create a pod with a random name, StatefulSets recreate the exact same pod name (`mysql-statefulset-0`) and reconnect it to its original storage. Your database data is safe, and all configuration from ConfigMap and Secret is automatically reapplied!

---

## Important StatefulSet Concepts

### Persistent Volume Claims (PVCs)

StatefulSets automatically create PVCs for each pod. Check them with:

```bash
kubectl get pvc -n mysql
```

Expected output:
```
NAME                       STATUS   VOLUME                                     CAPACITY   ACCESS MODES
mysql-data-mysql-statefulset-0   Bound    pvc-abc123...   1Gi        RWO
mysql-data-mysql-statefulset-1   Bound    pvc-def456...   1Gi        RWO
mysql-data-mysql-statefulset-2   Bound    pvc-ghi789...   1Gi        RWO
```

Each pod has its own dedicated storage that persists even when the pod is deleted.

### Scaling StatefulSets

To change the number of replicas:

```bash
kubectl scale statefulset mysql-statefulset --replicas=5 -n mysql
```

This will add mysql-statefulset-3 and mysql-statefulset-4. New pods automatically:
- Get their configuration from ConfigMap
- Get their password from Secret
- Create their own PVC for storage

### Updating StatefulSets

When you update the StatefulSet configuration:

```bash
kubectl apply -f statefulset.yml
```

Pods are updated in **reverse order** (2, then 1, then 0) to minimize disruption.

---

## Deployment Order Summary

When deploying, follow this specific order:

1. **Namespace** - Creates the isolated environment
2. **ConfigMap** - Non-sensitive configuration must exist before StatefulSet
3. **Secret** - Sensitive data must exist before StatefulSet
4. **Service** - Must exist before StatefulSet for DNS names
5. **StatefulSet** - Last, as it depends on all above resources

### Quick Deploy Script

Create a file `deploy.sh`:

```bash
#!/bin/bash
kubectl apply -f namespace.yml
kubectl apply -f configmap.yml
kubectl apply -f secrets.yml
kubectl apply -f service.yml
kubectl apply -f statefulset.yml

echo "Waiting for pods to be ready..."
kubectl wait --for=condition=ready pod -l app=mysql -n mysql --timeout=300s
echo "MySQL StatefulSet deployed successfully!"
```

Make it executable and run:
```bash
chmod +x deploy.sh
./deploy.sh
```

---

## Troubleshooting Tips

### Pod Won't Start

Check if ConfigMap and Secret exist:
```bash
kubectl get configmap,secret -n mysql
```

Check pod details:
```bash
kubectl describe pod mysql-statefulset-0 -n mysql
```

Look for errors related to missing ConfigMap or Secret:
```
Warning  Failed     Error: configmap "mysql-config-map" not found
Warning  Failed     Error: secret "mysql-secret" not found
```

Check pod logs:
```bash
kubectl logs mysql-statefulset-0 -n mysql
```

### Configuration Not Applied

Verify ConfigMap contents:
```bash
kubectl get configmap mysql-config-map -n mysql -o yaml
```

Verify Secret contents (decoded):
```bash
kubectl get secret mysql-secret -n mysql -o jsonpath='{.data.MYSQL_ROOT_PASSWORD}' | base64 --decode
```

Check environment variables in running pod:
```bash
kubectl exec mysql-statefulset-0 -n mysql -- env | grep MYSQL
```

### Connection Issues

Test connectivity from another pod:
```bash
kubectl run -it --rm debug --image=mysql:8.0 --restart=Never -n mysql -- mysql -h mysql-service -u root -p
```

### Storage Issues

Check persistent volume claims:
```bash
kubectl get pvc -n mysql
kubectl describe pvc mysql-data-mysql-statefulset-0 -n mysql
```

---

## Security Best Practices

### 1. Use Secrets for Sensitive Data ✅
You're already doing this! Passwords are stored in Secrets, not hardcoded.

### 2. Enable Secret Encryption at Rest
Ask your cluster admin to enable encryption for Secrets in etcd.

### 3. Restrict Secret Access
Use RBAC to limit who can view Secrets:

```yaml
apiVersion: rbac.authorization.k8s.io/v1
kind: Role
metadata:
  name: secret-reader
  namespace: mysql
rules:
- apiGroups: [""]
  resources: ["secrets"]
  verbs: ["get", "list"]
```

### 4. Use Sealed Secrets or External Secret Managers
Consider using tools like:
- **Sealed Secrets**: Encrypt secrets in Git
- **HashiCorp Vault**: Enterprise secret management
- **AWS Secrets Manager** / **Azure Key Vault**: Cloud provider solutions

### 5. Rotate Passwords Regularly
Change the Secret value periodically and restart pods.

### 6. Use Strong Passwords
Never use `root` as a password in production! Use complex passwords:

```bash
# Generate a strong password
openssl rand -base64 32
```

### 7. Limit Network Access
Use NetworkPolicies to restrict which pods can access MySQL:

```yaml
apiVersion: networking.k8s.io/v1
kind: NetworkPolicy
metadata:
  name: mysql-network-policy
  namespace: mysql
spec:
  podSelector:
    matchLabels:
      app: mysql
  policyTypes:
  - Ingress
  ingress:
  - from:
    - namespaceSelector:
        matchLabels:
          name: application-namespace
    ports:
    - protocol: TCP
      port: 3306
```

---

## Best Practices Summary

1. ✅ **Use Secrets** for sensitive data like passwords
2. ✅ **Use ConfigMaps** for non-sensitive configuration
3. ✅ **Always use persistent storage** for databases
4. ✅ **Set resource limits** to prevent pods from consuming all cluster resources
5. ✅ **Regular backups** - StatefulSets persist data, but you still need backups
6. ✅ **Monitor pod health** with liveness and readiness probes
7. ✅ **Test disaster recovery** by deleting pods and verifying data persistence
8. ✅ **Follow deployment order**: Namespace → ConfigMap → Secret → Service → StatefulSet
9. ✅ **Use strong passwords** and rotate them regularly
10. ✅ **Implement RBAC** to control access to Secrets

---

## Clean Up

To remove everything:

```bash
# Delete the StatefulSet (pods will be removed)
kubectl delete statefulset mysql-statefulset -n mysql

# Delete the Service
kubectl delete service mysql-service -n mysql

# Delete ConfigMap and Secret
kubectl delete configmap mysql-config-map -n mysql
kubectl delete secret mysql-secret -n mysql

# Delete PVCs (this deletes your data!)
kubectl delete pvc --all -n mysql

# Delete the Namespace (removes everything inside)
kubectl delete namespace mysql
```

**Warning:** This will permanently delete all data unless you have backups!

### Alternative: Delete Everything at Once

```bash
kubectl delete namespace mysql
```

This removes the namespace and everything inside it, including the StatefulSet, Service, ConfigMap, Secret, and PVCs.

---

## Summary

You've learned how to:
1. ✅ Create a namespace for organization
2. ✅ Use ConfigMaps for non-sensitive configuration
3. ✅ Use Secrets for sensitive data like passwords
4. ✅ Deploy a headless service for StatefulSet networking
5. ✅ Create a MySQL StatefulSet with persistent storage
6. ✅ Access and interact with MySQL pods
7. ✅ Understand pod identity and data persistence
8. ✅ Manage pods safely (delete and automatic recreation)
9. ✅ Update configuration without changing application code
10. ✅ Follow security best practices

## Key Takeaways

- **StatefulSets** maintain stable pod identities and persistent storage
- **ConfigMaps** store non-sensitive configuration data
- **Secrets** store sensitive data like passwords in base64-encoded format
- **Separation of concerns**: Configuration is separate from application deployment
- **Security**: Never hardcode passwords in YAML files
- **Headless services** provide unique DNS names for each pod
- **Pods are ordered** and created/deleted sequentially
- **Data persists** even when pods are deleted and recreated
- **Each pod** gets its own persistent volume
- **Configuration is dynamic**: Update ConfigMaps and Secrets without changing StatefulSet

---

## File Structure

Your final project structure should look like this:

```
mysql/
├── namespace.yml
├── configmap.yml
├── secrets.yml
├── service.yml
├── statefulset.yml
└── README.md
```

---

## Additional Resources

- [Kubernetes StatefulSet Documentation](https://kubernetes.io/docs/concepts/workloads/controllers/statefulset/)
- [Kubernetes ConfigMaps](https://kubernetes.io/docs/concepts/configuration/configmap/)
- [Kubernetes Secrets](https://kubernetes.io/docs/concepts/configuration/secret/)
- [MySQL Docker Hub](https://hub.docker.com/_/mysql)
- [Kubernetes Persistent Volumes](https://kubernetes.io/docs/concepts/storage/persistent-volumes/)
- [StatefulSet Basics Tutorial](https://kubernetes.io/docs/tutorials/stateful-application/basic-stateful-set/)
- [Security Best Practices for Kubernetes](https://kubernetes.io/docs/concepts/security/)

---
