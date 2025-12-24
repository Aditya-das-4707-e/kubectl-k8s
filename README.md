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
<p>To see each pods</p>

```
kubectl get pods -n <namespace>
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
kubectl set image deployment/nginx-deployment -n nginx <image_name>=<version>
```
