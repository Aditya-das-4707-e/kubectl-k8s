<h1>Kubectl command</h1> 
<p>I make this file because this file easily accessable for everyone.</p>

# First create node
<p>First go kind-cluster folder and write this command to create node</p>

```
kind create cluster --name <choose a name> --config=config.yml
```
# To create namespace
  <p>Go to this section where namespace.yml file present</p>
  
```
kubectl apply -f namespace.yml
```
# To create pod inside the namespace 
<p>Go to this section where namespace.yml file present</p>

```
kubectl apply -f pod.yml
```
# To intaract with pod 
```
kubectl exec -it pod/<pod-name> -n <namespace> -- bash
```
# To debug this pod and show everything
```
kubectl describe pod/<pod-name> -n <namespace>
```
# To see each pods

```
kubectl get pods -n <namespace>
```
# Deployment 
<p>In deployment you first delete pod.yml file</p>

```
kubectl delete -f pod.yml
```
<p>Then you deploy</p>

```
kubectl apply -f deployment.yml
```
<p>To see deployment pods</p>

```
kubectl get deployment -n <namespace>
```
<p>To scale this deployment</p>

```
kubectl scale deployment.app/nginx-deployment -n nginx --replicas=5
```
<p>To see more information about pods</p>

```
kubectl get pods -n nginx -o wide
```
<p>To update image in deployment</p>

```
kubectl set image deployment/nginx-deployment -n nginx <container_name>=<image_name>:<version>
```
<p>To delete deployment sets</p>

```
kubectl delete -f deployment.yml
```
# ReplicaSets (optional use only / education purpose)
<p>To apply replicasets</p>

```
kubectl apply -f replicasets.yml
```
<p>To see replicaset pod</p>

```
kubectl get replicasets -n <namespace>
```
<p>To delete replicasets</p>

```
kubectl delete -f replicasets.yml
```
# DaemonSets 
<p>To apply daemonsets</p>

```
kubectl apply -f daemonsets.yml
```
<p>to see demonsets pods</p>

```
kubectl get pods -n <namespace>
```
<p>To delete deamonsets</p>

```
kubectl delete -f daemonsets.yml
```
# Job
<p>To apply job</p>

```
kubectl apply -f job.yml
```
<p>To see job</p>

```
kubectl get job -n <namespace>
```
<p>To see pod</p>

```
kubectl get pods -n <namespace>
```
