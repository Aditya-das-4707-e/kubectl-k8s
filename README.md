# Kubectl Command Reference Guide

I created this file to make Kubernetes commands easily accessible for everyone learning Kubernetes.

---

## First Create Node

**What it does:** Creates a Kubernetes cluster using Kind (Kubernetes in Docker).

**Use case:** Setting up a local development environment for testing Kubernetes applications.

**Destination:** Creates a local Kubernetes cluster on your machine.

**Simple explanation:** Think of this as building a playground where your apps will live. Just like you need a playground before you can play, you need a cluster before you can run apps.

### Steps:
1. Install [Kind](https://github.com/Aditya-das-4707-e/install-kind-kubectl.sh)
2. Navigate to the `kind-cluster` folder
3. Run this command:

```bash
kind create cluster --name <choose-a-name> --config=config.yml
```

**Example:**
```bash
kind create cluster --name my-first-cluster --config=config.yml
```

---

## To Create Namespace

**What it does:** Creates a separate workspace (namespace) in your cluster.

**Use case:** Organizing your apps into different groups, like having different folders for different projects.

**Destination:** Creates a logical boundary inside your Kubernetes cluster.

**Simple explanation:** A namespace is like a separate room in your house. You can keep your toys in one room and books in another. Similarly, you can keep different apps in different namespaces.

### Steps:
1. Navigate to the folder where `namespace.yml` file is present
2. Run this command:

```bash
kubectl apply -f namespace.yml
```

**Example:**
```bash
kubectl apply -f namespace.yml
# Output: namespace/nginx created
```

---

## To See Namespaces

**What it does:** Shows all the namespaces in your cluster.

**Use case:** Checking what namespaces exist in your cluster.

**Destination:** Displays list in your terminal.

**Simple explanation:** This is like looking at the labels on different rooms in your house to see what's inside.

### Command:
```bash
kubectl get ns
```

**Example output:**
```bash
NAME                 STATUS   AGE
default              Active   5d
nginx                Active   3d14h
kube-system          Active   5d
```

---

## To Create Pod Inside the Namespace

**What it does:** Creates a pod (smallest unit that runs your app) inside a specific namespace.

**Use case:** Running a single instance of your application.

**Destination:** Creates a pod in the specified namespace.

**Simple explanation:** A pod is like a small box that holds your app. Just like you put a toy in a box, Kubernetes puts your app in a pod.

### Steps:
1. Navigate to the folder where `pod.yml` file is present
2. Run this command:

```bash
kubectl apply -f pod.yml
```

**Example:**
```bash
kubectl apply -f pod.yml
# Output: pod/my-app-pod created
```

---

## To Interact with Pod

**What it does:** Opens a terminal inside your running pod.

**Use case:** Troubleshooting, checking logs, or testing things inside the pod.

**Destination:** Connects you to the pod's command line.

**Simple explanation:** This is like opening the box and looking inside to see what's happening with your toy.

### Command:
```bash
kubectl exec -it pod/<pod-name> -n <namespace> -- bash
```

**Example:**
```bash
kubectl exec -it pod/nginx-pod -n nginx -- bash
# You'll see: root@nginx-pod:/#
```

---

## To Debug Pod and Show Everything

**What it does:** Shows detailed information about a pod including events, status, and errors.

**Use case:** Finding out why a pod is not working properly.

**Destination:** Displays detailed information in your terminal.

**Simple explanation:** This is like getting a report card for your pod - it tells you everything about how it's doing.

### Command:
```bash
kubectl describe pod/<pod-name> -n <namespace>
```

**Example:**
```bash
kubectl describe pod/nginx-pod -n nginx
# Shows: Name, Namespace, Status, IP, Events, etc.
```

---

## To See Each Pod

**What it does:** Lists all pods in a specific namespace.

**Use case:** Checking which pods are running and their status.

**Destination:** Displays list in your terminal.

**Simple explanation:** This shows you all the boxes (pods) in a particular room (namespace).

### Command:
```bash
kubectl get pods -n <namespace>
```

**Example:**
```bash
kubectl get pods -n nginx

# Output:
NAME                    READY   STATUS    RESTARTS   AGE
nginx-pod               1/1     Running   0          2m
```

---

## Deployment

**What it does:** Manages multiple copies of your app automatically.

**Use case:** Running multiple instances of your app for reliability and handling more traffic.

**Destination:** Creates and manages pods automatically.

**Simple explanation:** Instead of managing one toy at a time, a deployment is like having a toy factory that makes sure you always have the right number of toys, and if one breaks, it makes a new one automatically.

### Step 1: Delete the pod file (because deployment will manage pods for us)
```bash
kubectl delete -f pod.yml
```

**Example:**
```bash
kubectl delete -f pod.yml
# Output: pod "nginx-pod" deleted
```

### Step 2: Create the deployment
```bash
kubectl apply -f deployment.yml
```

**Example:**
```bash
kubectl apply -f deployment.yml
# Output: deployment.apps/nginx-deployment created
```

### Step 3: See deployment status
```bash
kubectl get deployment -n <namespace>
```

**Example:**
```bash
kubectl get deployment -n nginx

# Output:
NAME               READY   UP-TO-DATE   AVAILABLE   AGE
nginx-deployment   2/2     2            2           5m
```

### Step 4: Scale the deployment (increase or decrease pod copies)

**What it does:** Changes the number of pod copies running.

**Use case:** Handling more traffic by adding more pods, or saving resources by reducing pods.

```bash
kubectl scale deployment/<deployment-name> -n <namespace> --replicas=5
```

**Example:**
```bash
kubectl scale deployment/nginx-deployment -n nginx --replicas=5
# Output: deployment.apps/nginx-deployment scaled
```

### Step 5: See more information about pods
```bash
kubectl get pods -n <namespace> -o wide
```

**Example:**
```bash
kubectl get pods -n nginx -o wide

# Output shows: NAME, READY, STATUS, RESTARTS, AGE, IP, NODE
```

### Step 6: Update image in deployment

**What it does:** Updates your app to a new version without downtime.

**Use case:** Deploying new features or bug fixes.

```bash
kubectl set image deployment/<deployment-name> -n <namespace> <container-name>=<image-name>:<version>
```

**Example:**
```bash
kubectl set image deployment/nginx-deployment -n nginx nginx=nginx:1.21
# Output: deployment.apps/nginx-deployment image updated
```

### Step 7: Delete deployment
```bash
kubectl delete -f deployment.yml
```

**Example:**
```bash
kubectl delete -f deployment.yml
# Output: deployment.apps "nginx-deployment" deleted
```

---

## When You Add Many Services

**What it does:** Sets up an Ingress controller to route traffic to different services.

**Use case:** Running multiple apps and accessing them through different URLs or paths.

**Destination:** Creates an entry point for all your services.

**Simple explanation:** Think of this as a receptionist at a building who directs visitors to different offices. The Ingress controller directs internet traffic to the right app.

### To Add Ingress NGINX Controller

**Step 1:** Add all services in the same namespace.

**Step 2:** Navigate to your kind cluster folder and run:

```bash
kubectl apply -f https://kind.sigs.k8s.io/examples/ingress/deploy-ingress-nginx.yaml
```

**Example output:**
```bash
aditya@Aditya:~/devops/kubernetes/kind-cluster$ kubectl apply -f https://kind.sigs.k8s.io/examples/ingress/deploy-ingress-nginx.yaml
namespace/ingress-nginx created
serviceaccount/ingress-nginx created
serviceaccount/ingress-nginx-admission created
role.rbac.authorization.k8s.io/ingress-nginx created
role.rbac.authorization.k8s.io/ingress-nginx-admission created
clusterrole.rbac.authorization.k8s.io/ingress-nginx created
clusterrole.rbac.authorization.k8s.io/ingress-nginx-admission created
rolebinding.rbac.authorization.k8s.io/ingress-nginx created
rolebinding.rbac.authorization.k8s.io/ingress-nginx-admission created
clusterrolebinding.rbac.authorization.k8s.io/ingress-nginx created
clusterrolebinding.rbac.authorization.k8s.io/ingress-nginx-admission created
configmap/ingress-nginx-controller created
service/ingress-nginx-controller created
service/ingress-nginx-controller-admission created
deployment.apps/ingress-nginx-controller created
job.batch/ingress-nginx-admission-create created
job.batch/ingress-nginx-admission-patch created
ingressclass.networking.k8s.io/nginx created
validatingwebhookconfiguration.admissionregistration.k8s.io/ingress-nginx-admission created
```

**Step 3:** Check that the namespace was created:

```bash
kubectl get ns
```

**Example output:**
```bash
aditya@Aditya:~/devops/kubernetes/kind-cluster$ kubectl get ns
NAME                 STATUS   AGE
default              Active   5d
ingress-nginx        Active   4m46s
kube-node-lease      Active   5d
kube-public          Active   5d
kube-system          Active   5d
local-path-storage   Active   5d
nginx                Active   3d14h
```

You'll see a new namespace called **ingress-nginx**.

**Step 4:** Check pods in the ingress-nginx namespace:

```bash
kubectl get pods -n ingress-nginx
```

**Example output:**
```bash
aditya@Aditya:~/devops/kubernetes/kind-cluster$ kubectl get pods -n ingress-nginx
NAME                                        READY   STATUS      RESTARTS   AGE
ingress-nginx-admission-create-gn86m        0/1     Completed   0          8m23s
ingress-nginx-admission-patch-fcnmt         0/1     Completed   2          8m23s
ingress-nginx-controller-66fdf84d85-6jpdl   1/1     Running     0          8m23s
```

**Understanding STATUS:**
- **STATUS: Completed** → These are jobs that finished their work
- **STATUS: Running** → These are pods that are actively running

**Step 5:** Check services in the ingress-nginx namespace:

```bash
kubectl get service -n ingress-nginx
```

**Example output:**
```bash
aditya@Aditya:~/devops/kubernetes/kind-cluster$ kubectl get service -n ingress-nginx
NAME                                 TYPE           CLUSTER-IP      EXTERNAL-IP   PORT(S)                      AGE
ingress-nginx-controller             LoadBalancer   10.96.179.123   <pending>     80:32686/TCP,443:30343/TCP   14m
ingress-nginx-controller-admission   ClusterIP      10.96.220.81    <none>        443/TCP                      14m
```

Notice the **TYPE: LoadBalancer** service. This is what we'll expose later.

**Step 6:** Create an ingress.yml file in your code folder:

```yaml
apiVersion: networking.k8s.io/v1
kind: Ingress
metadata:
  name: <choose-your-name>
  # Example: nginx-notes-ingress
  namespace: <enter-common-namespace-for-both-services>
  # Example: nginx
  annotations:
    nginx.ingress.kubernetes.io/rewrite-target: /  # Optional: Use when both apps run on invisible ports like 80
spec:
  rules: 
  - http: 
      paths: 
      - pathType: Prefix 
        path: <enter-your-path>
        # Example: /nginx
        backend: 
          service:
            name: nginx-service
            port:
              number: 80
      - pathType: Prefix
        path: <enter-your-path>
        # Example: /notes
        backend: 
          service:
            name: notes-app-service
            port:
              number: 8000
```

**Step 7:** Apply the ingress configuration:

```bash
kubectl apply -f ingress.yml
```

**Example:**
```bash
kubectl apply -f ingress.yml
# Output: ingress.networking.k8s.io/nginx-notes-ingress created
```

**Step 8:** Check your ingress:

```bash
kubectl get ingress -n <namespace>
```

**Example output:**
```bash
aditya@Aditya:~/devops/kubernetes/kind-cluster/nginx$ kubectl get ingress -n nginx
NAME                  CLASS    HOSTS   ADDRESS     PORTS   AGE
nginx-notes-ingress   <none>   *       localhost   80      71s
```

**Step 9:** See everything in the namespace:

```bash
kubectl get all -n <namespace>
```

**Example output:**
```bash
aditya@Aditya:~/devops/kubernetes/kind-cluster/nginx$ kubectl get all -n nginx
NAME                                      READY   STATUS    RESTARTS      AGE
pod/nginx-deployment-6f59995b97-7k8lq     1/1     Running   2 (81m ago)   15h
pod/nginx-deployment-6f59995b97-thqxp     1/1     Running   2 (81m ago)   15h
pod/notes-app-deployment-978b5946-t2knn   1/1     Running   0             48m

NAME                        TYPE        CLUSTER-IP      EXTERNAL-IP   PORT(S)    AGE
service/nginx-service       ClusterIP   10.96.68.204    <none>        80/TCP     13h
service/notes-app-service   ClusterIP   10.96.184.149   <none>        8000/TCP   48m

NAME                                   READY   UP-TO-DATE   AVAILABLE   AGE
deployment.apps/nginx-deployment       2/2     2            2           15h
deployment.apps/notes-app-deployment   1/1     1            1           48m

NAME                                            DESIRED   CURRENT   READY   AGE
replicaset.apps/nginx-deployment-6f59995b97     2         2         2       15h
replicaset.apps/notes-app-deployment-978b5946   1         1         1       48m
```

**Step 10:** Expose the ingress service:

First, check the service:
```bash
kubectl get service -n ingress-nginx
```

**Example output:**
```bash
aditya@Aditya:~/devops/kubernetes/kind-cluster/nginx$ kubectl get service -n ingress-nginx
NAME                                 TYPE           CLUSTER-IP      EXTERNAL-IP   PORT(S)                      AGE
ingress-nginx-controller             LoadBalancer   10.96.179.123   <pending>     80:32686/TCP,443:30343/TCP   44m
ingress-nginx-controller-admission   ClusterIP      10.96.220.81    <none>        443/TCP                      44m
```

We need to expose the **ingress-nginx-controller** service.

**Step 11:** Port forward to access the service:

```bash
sudo -E kubectl port-forward service/ingress-nginx-controller -n ingress-nginx 80:80
```

**Note:** If you see "port already in use" error, visit my [Port Cleanup Tool](https://github.com/Aditya-das-4707-e/Ghost-Port-Cleanup-Tool).

**Ingress Example:** Visit [Dajango Notes App](https://github.com/Aditya-das-4707-e/django-notes-app-k8s).

**Success!** Now you can access your apps through:
- `http://localhost/nginx` for nginx app
- `http://localhost/notes` for notes app

---

### Go to Next Step

---

## ReplicaSets (Optional - Educational Purpose Only)

**What it does:** Ensures a specific number of pod copies are always running.

**Use case:** Maintaining availability of your app. (Note: Use Deployments instead in production)

**Destination:** Creates and monitors pods.

**Simple explanation:** Like a babysitter who makes sure there are always exactly 3 toys on the table. If one disappears, the babysitter adds a new one.

### To apply ReplicaSets:
```bash
kubectl apply -f replicasets.yml
```

**Example:**
```bash
kubectl apply -f replicasets.yml
# Output: replicaset.apps/nginx-replicaset created
```

### To see ReplicaSets:
```bash
kubectl get replicasets -n <namespace>
```

**Example:**
```bash
kubectl get replicasets -n nginx

# Output:
NAME               DESIRED   CURRENT   READY   AGE
nginx-replicaset   3         3         3       2m
```

### To delete ReplicaSets:
```bash
kubectl delete -f replicasets.yml
```

**Example:**
```bash
kubectl delete -f replicasets.yml
# Output: replicaset.apps "nginx-replicaset" deleted
```

---

## DaemonSets

**What it does:** Runs one copy of a pod on every node in your cluster.

**Use case:** Running monitoring agents, log collectors, or network tools on all nodes.

**Destination:** Creates pods on all nodes.

**Simple explanation:** Like putting one security camera in every room of your house automatically.

### To apply DaemonSets:
```bash
kubectl apply -f daemonsets.yml
```

**Example:**
```bash
kubectl apply -f daemonsets.yml
# Output: daemonset.apps/nginx-daemonset created
```

### To see DaemonSet pods:
```bash
kubectl get pods -n <namespace>
```

**Example:**
```bash
kubectl get pods -n nginx

# Output (you'll see one pod per node):
NAME                     READY   STATUS    RESTARTS   AGE
nginx-daemonset-abc123   1/1     Running   0          1m
nginx-daemonset-def456   1/1     Running   0          1m
```

### To delete DaemonSets:
```bash
kubectl delete -f daemonsets.yml
```

**Example:**
```bash
kubectl delete -f daemonsets.yml
# Output: daemonset.apps "nginx-daemonset" deleted
```

---

## Job

**What it does:** Runs a task once until it completes successfully.

**Use case:** Running batch processing, data migration, or one-time scripts.

**Destination:** Creates a pod that runs until completion.

**Simple explanation:** Like asking someone to clean the garage once. They work until it's done, then they stop.

### To apply Job:
```bash
kubectl apply -f job.yml
```

**Example:**
```bash
kubectl apply -f job.yml
# Output: job.batch/data-processor created
```

### To see Job:
```bash
kubectl get job -n <namespace>
```

**Example:**
```bash
kubectl get job -n nginx

# Output:
NAME             COMPLETIONS   DURATION   AGE
data-processor   1/1           45s        2m
```

### To see pods created by the Job:
```bash
kubectl get pods -n <namespace>
```

**Example:**
```bash
kubectl get pods -n nginx

# Output:
NAME                   READY   STATUS      RESTARTS   AGE
data-processor-abc123  0/1     Completed   0          3m
```

### To see the logs:
```bash
kubectl logs pod/<job-pod-name> -n <namespace>
```

**Example:**
```bash
kubectl logs pod/data-processor-abc123 -n nginx

# Output: (Your job's output will appear here)
Processing started...
Data processed successfully!
Job completed.
```

### To delete the Job:
```bash
kubectl delete -f job.yml
```

**Example:**
```bash
kubectl delete -f job.yml
# Output: job.batch "data-processor" deleted
```

---

## CronJob

**What it does:** Runs a task on a schedule (like a cron job in Linux).

**Use case:** Running backups, sending reports, or cleaning up old data regularly.

**Destination:** Creates jobs on schedule that create pods.

**Simple explanation:** Like setting an alarm to water your plants every morning at 8 AM automatically.

### To apply CronJob:
```bash
kubectl apply -f cron-job.yml
```

**Example:**
```bash
kubectl apply -f cron-job.yml
# Output: cronjob.batch/daily-backup created
```

### To see CronJob:
```bash
kubectl get cronjob -n <namespace>
```

**Example:**
```bash
kubectl get cronjob -n nginx

# Output:
NAME           SCHEDULE      SUSPEND   ACTIVE   LAST SCHEDULE   AGE
daily-backup   0 2 * * *     False     0        17h             5d
```

**Understanding the schedule:** `0 2 * * *` means run at 2:00 AM every day.

### To see pods created by CronJob:
```bash
kubectl get pods -n <namespace>
```

**Example:**
```bash
kubectl get pods -n nginx

# Output (shows recent job runs):
NAME                          READY   STATUS      RESTARTS   AGE
daily-backup-27890123-abc12   0/1     Completed   0          2h
daily-backup-27900124-def34   0/1     Completed   0          26h
```

### To delete CronJob:
```bash
kubectl delete -f cron-job.yml
```

**Example:**
```bash
kubectl delete -f cron-job.yml
# Output: cronjob.batch "daily-backup" deleted
```

---

## (1) Persistent Volume

**What it does:** Creates storage space that survives even when pods are deleted.

**Use case:** Storing database data, user uploads, or any data that needs to persist.

**Destination:** Creates storage on your node.

**Simple explanation:** Like having a storage locker that keeps your stuff safe even if you move to a different apartment. Your data stays there.

### To create Persistent Volume:

**Important:** Update the storage path in the YAML file with your own path before applying.

```bash
kubectl apply -f persistentVolume.yml
```

**Example:**
```bash
kubectl apply -f persistentVolume.yml
# Output: persistentvolume/local-pv created
```

### To see Persistent Volume:
```bash
kubectl get pv
```

**Example:**
```bash
kubectl get pv

# Output:
NAME       CAPACITY   ACCESS MODES   RECLAIM POLICY   STATUS      CLAIM   AGE
local-pv   5Gi        RWO            Retain           Available           30s
```

**Understanding STATUS:**
- **Available** → Ready to be used but not yet claimed
- **Bound** → Currently in use by a pod

### To delete Persistent Volume:
```bash
kubectl delete pv/local-pv
```

**Example:**
```bash
kubectl delete pv/local-pv
# Output: persistentvolume "local-pv" deleted
```

---

## (2) Persistent Volume Claim

**What it does:** Requests storage from available Persistent Volumes.

**Use case:** Your pod uses this to ask for storage space.

**Destination:** Connects your pod to persistent storage.

**Simple explanation:** Like filling out a form to rent that storage locker. Once approved, you can use it.

### To create Persistent Volume Claim:
```bash
kubectl apply -f persistentVolumeClaim.yml
```

**Example:**
```bash
kubectl apply -f persistentVolumeClaim.yml
# Output: persistentvolumeclaim/local-pvc created
```

### To see Persistent Volume status:
```bash
kubectl get pv
```

**Example:**
```bash
kubectl get pv

# Output:
NAME       CAPACITY   ACCESS MODES   RECLAIM POLICY   STATUS   CLAIM             AGE
local-pv   5Gi        RWO            Retain           Bound    nginx/local-pvc   2m
```

Now the STATUS shows **Bound**, meaning the storage is claimed and in use!

### To see Persistent Volume Claim:
```bash
kubectl get pvc -n <namespace>
```

**Example:**
```bash
kubectl get pvc -n nginx

# Output:
NAME        STATUS   VOLUME     CAPACITY   ACCESS MODES   AGE
local-pvc   Bound    local-pv   5Gi        RWO            1m
```

### To delete Persistent Volume Claim:
```bash
kubectl delete pvc/local-pvc -n <namespace>
```

**Example:**
```bash
kubectl delete pvc/local-pvc -n nginx
# Output: persistentvolumeclaim "local-pvc" deleted
```

---

## Service

**What it does:** Provides a stable network address to access your pods.

**Use case:** Allowing other apps or users to access your app reliably.

**Destination:** Creates a network endpoint for your pods.

**Simple explanation:** Like giving your house a permanent address. Even if you renovate inside (restart pods), people can still find you at the same address.

### To apply Service:
```bash
kubectl apply -f service.yml
```

**Example:**
```bash
kubectl apply -f service.yml
# Output: service/nginx-service created
```

### To see all resources:
```bash
kubectl get all -n <namespace>
```

**Example:**
```bash
kubectl get all -n nginx

# Output:
NAME                        TYPE        CLUSTER-IP      EXTERNAL-IP   PORT(S)   AGE
service/nginx-service       ClusterIP   10.96.68.204    <none>        80/TCP    2m

NAME                               READY   UP-TO-DATE   AVAILABLE   AGE
deployment.apps/nginx-deployment   2/2     2            2           15h

NAME                                    READY   STATUS    RESTARTS   AGE
pod/nginx-deployment-6f59995b97-7k8lq   1/1     Running   0          15h
pod/nginx-deployment-6f59995b97-thqxp   1/1     Running   0          15h
```

### To access service in browser (Local Computer):

**For your local laptop/desktop:**

```bash
kubectl port-forward service/<your-service-name> -n <namespace> <local-port>:80
```

**Example:**
```bash
kubectl port-forward service/nginx-service -n nginx 8080:80

# Now access: http://localhost:8080
```

### To access service in browser (Cloud EC2 Instance):

**For cloud servers (AWS EC2, etc.):**

```bash
sudo -E kubectl port-forward service/<your-service-name> -n <namespace> <local-port>:80 --address=0.0.0.0
```

**Example:**
```bash
sudo -E kubectl port-forward service/nginx-service -n nginx 8080:80 --address=0.0.0.0

# Now access: http://<your-ec2-public-ip>:8080
```

**Note:** Make sure your security group allows inbound traffic on the port you're using!

---

## Quick Reference Summary

| Command | Purpose |
|---------|---------|
| `kubectl get ns` | List all namespaces |
| `kubectl get pods -n <namespace>` | List all pods in a namespace |
| `kubectl get all -n <namespace>` | List all resources in a namespace |
| `kubectl describe pod/<name> -n <namespace>` | Get detailed pod information |
| `kubectl logs pod/<name> -n <namespace>` | View pod logs |
| `kubectl exec -it pod/<name> -n <namespace> -- bash` | Access pod terminal |
| `kubectl delete -f <file.yml>` | Delete resources from a file |
| `kubectl apply -f <file.yml>` | Create/update resources from a file |

---

## Tips for Beginners

1. **Always specify the namespace** with `-n <namespace>` to avoid confusion
2. **Use descriptive names** for your resources to find them easily later
3. **Start simple** - create one pod, then move to deployments
4. **Check status frequently** using `kubectl get` commands
5. **Read error messages carefully** - they usually tell you what's wrong
6. **Use `kubectl describe`** when something isn't working - it shows detailed events

---

## Common Issues and Solutions

### Issue: Port already in use
**Solution:** Use my [Port Cleanup Tool](https://github.com/Aditya-das-4707-e/Ghost-Port-Cleanup-Tool)

### Issue: Pods stuck in "Pending" state
**Solution:** Check with `kubectl describe pod/<pod-name> -n <namespace>` to see what's wrong

### Issue: Can't access service
**Solution:** Make sure you're using the correct port forwarding command for your environment (local vs cloud)

### Issue: Image pull errors
**Solution:** Check if the image name and version are correct in your YAML file

---

**Happy Learning Kubernetes! 🚀**
