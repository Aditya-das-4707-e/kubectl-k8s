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

## Step 2: Create the Service

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

### Why create the service first?
StatefulSets use the service to give each pod a unique DNS name like:
- `mysql-statefulset-0.mysql-service.mysql.svc.cluster.local`
- `mysql-statefulset-1.mysql-service.mysql.svc.cluster.local`

### Expected Output
```
aditya@Aditya:~/devops/kubernetes/kind-cluster/mysql$ kubectl apply -f service.yml 
service/mysql-service created
```

### Use Cases
- **Database replication**: Master and slave pods need stable addresses
- **Cluster communication**: Pods need to find each other reliably
- **Client connections**: Applications connect to specific database instances

---

## Step 3: Create the StatefulSet

### Command
```bash
kubectl apply -f statefulset.yml
```

### What does this command do?
- Reads your `statefulset.yml` configuration
- Creates MySQL pods one by one in order (0, then 1, then 2)
- Attaches persistent storage to each pod
- Sets up environment variables for MySQL

### Why use StatefulSet for databases?
- **Data persistence**: Each pod gets its own storage that survives restarts
- **Stable identity**: Pods keep the same name and network address
- **Ordered deployment**: Ensures primary database starts before replicas

### Expected Output
```
aditya@Aditya:~/devops/kubernetes/kind-cluster/mysql$ kubectl apply -f statefulset.yml 
statefulset.apps/mysql-statefulset created
```

---

## Step 4: Verify the Pods

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
aditya@Aditya:~/devops/kubernetes/kind-cluster/mysql$ kubectl get pods -n mysql 
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

## Step 5: Check the Service

### Command
```bash
kubectl get svc -n mysql
```

**Note:** `svc` is shorthand for `service`. Both commands work the same way!

### What does this command do?
Displays all services in the mysql namespace, showing how your pods can be accessed.

### Expected Output
```
aditya@Aditya:~/devops/kubernetes/kind-cluster/mysql$ kubectl get svc -n mysql
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

### Step 6: List Available Pods

### Command
```bash
kubectl get pods -n mysql
```

### What does this command do?
Shows you all available MySQL pods so you can choose which one to access.

### Expected Output
```
aditya@Aditya:~/devops/kubernetes/kind-cluster/mysql$ kubectl get pods -n mysql
NAME                  READY   STATUS    RESTARTS   AGE
mysql-statefulset-0   1/1     Running   0          42m
mysql-statefulset-1   1/1     Running   0          41m
mysql-statefulset-2   1/1     Running   0          40m
```

---

### Step 7: Enter the Pod Shell

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
aditya@Aditya:~/devops/kubernetes/kind-cluster/mysql$ kubectl exec -it mysql-statefulset-0 -n mysql -- bash
bash-5.1# 
```

**Success!** You're now inside the MySQL pod. The prompt changed to `bash-5.1#`.

### Use Cases
- **Troubleshooting**: Check logs and configurations
- **Data inspection**: Query databases directly
- **Debugging**: Test connectivity and configurations
- **Maintenance**: Run database operations manually

---

### Step 8: Login to MySQL

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

The password is defined in your `statefulset.yml` file in the environment variables section:

```yaml
env:
  - name: MYSQL_ROOT_PASSWORD
    value: root 
```

**The password is:** `root`

### Changing the Password

If you want to use a different password, edit the `statefulset.yml` file:

```yaml
env:
  - name: MYSQL_ROOT_PASSWORD
    value: admin 
```

Then reapply the configuration:
```bash
kubectl apply -f statefulset.yml
```

### Expected Output
```
aditya@Aditya:~/devops/kubernetes/kind-cluster/mysql$ kubectl exec -it mysql-statefulset-0 -n mysql -- bash
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

### Step 9: View Databases

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

Look at your `statefulset.yml` file environment variables:

```yaml
env:
  - name: MYSQL_ROOT_PASSWORD
    value: root 
  - name: MYSQL_DATABASE
    value: devops
```

The `MYSQL_DATABASE` variable automatically creates a database named 'devops' when MySQL starts!

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
```

---

## Exiting the Pod

### Step 10: Exit MySQL

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

### Step 11: Exit the Pod Shell

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

## Managing Pods

## Deleting a Pod

### Why Delete a Pod?
- **Testing resilience**: Verify that StatefulSet recreates the pod
- **Forcing restart**: When a pod is stuck or misbehaving
- **Updating configuration**: Sometimes requires a fresh start
- **Troubleshooting**: Eliminate problematic pods

### Step 12: Choose a Pod to Delete

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

### Step 13: Delete the Pod

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
5. Your data is preserved!

### Expected Output
```
aditya@Aditya:~/devops/kubernetes/kind-cluster/mysql$ kubectl delete pod mysql-statefulset-0 -n mysql
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
Unlike regular Deployments that would create a pod with a random name, StatefulSets recreate the exact same pod name (`mysql-statefulset-0`) and reconnect it to its original storage. Your database data is safe!

---

## Important StatefulSet Concepts

### Persistent Volume Claims (PVCs)

StatefulSets automatically create PVCs for each pod. Check them with:

```bash
kubectl get pvc -n mysql
```

Each pod has its own dedicated storage that persists even when the pod is deleted.

### Scaling StatefulSets

To change the number of replicas:

```bash
kubectl scale statefulset mysql-statefulset --replicas=5 -n mysql
```

This will add mysql-statefulset-3 and mysql-statefulset-4.

### Updating StatefulSets

When you update the StatefulSet configuration:

```bash
kubectl apply -f statefulset.yml
```

Pods are updated in **reverse order** (2, then 1, then 0) to minimize disruption.

---

## Troubleshooting Tips

### Pod Won't Start

Check pod details:
```bash
kubectl describe pod mysql-statefulset-0 -n mysql
```

Check pod logs:
```bash
kubectl logs mysql-statefulset-0 -n mysql
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
kubectl describe pvc <pvc-name> -n mysql
```

---

## Best Practices

1. **Always use persistent storage** for databases
2. **Set resource limits** to prevent pods from consuming all cluster resources
3. **Use secrets** for sensitive data like passwords (not plain text in YAML)
4. **Regular backups** - StatefulSets persist data, but you still need backups
5. **Monitor pod health** with liveness and readiness probes
6. **Test disaster recovery** by deleting pods and verifying data persistence

---

## Clean Up

To remove everything:

```bash
# Delete the StatefulSet
kubectl delete statefulset mysql-statefulset -n mysql

# Delete the Service
kubectl delete service mysql-service -n mysql

# Delete the Namespace (removes everything inside)
kubectl delete namespace mysql
```

**Warning:** This will permanently delete all data unless you have backups!

---

## Summary

You've learned how to:
1. ✅ Create a namespace for organization
2. ✅ Deploy a headless service for StatefulSet networking
3. ✅ Create a MySQL StatefulSet with persistent storage
4. ✅ Access and interact with MySQL pods
5. ✅ Understand pod identity and data persistence
6. ✅ Manage pods safely (delete and automatic recreation)

## Key Takeaways

- **StatefulSets** maintain stable pod identities and persistent storage
- **Headless services** provide unique DNS names for each pod
- **Pods are ordered** and created/deleted sequentially
- **Data persists** even when pods are deleted and recreated
- **Each pod** gets its own persistent volume

---

## Additional Resources

- [Kubernetes StatefulSet Documentation](https://kubernetes.io/docs/concepts/workloads/controllers/statefulset/)
- [MySQL Docker Hub](https://hub.docker.com/_/mysql)
- [Kubernetes Persistent Volumes](https://kubernetes.io/docs/concepts/storage/persistent-volumes/)
- [StatefulSet Basics Tutorial](https://kubernetes.io/docs/tutorials/stateful-application/basic-stateful-set/)

---
