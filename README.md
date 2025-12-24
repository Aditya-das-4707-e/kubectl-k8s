# Kubectl command 
I make this file because this file easily accessable for everyone. 
# To create namespace 
```
kubectl apply -f namespace.yml
```
# To create pod inside the namespace 
```
kubectl apply -f pod.yml
```
# To intaract with pod 
```
kubectl exec -it pod/<pod-name> -n <namespace> -- bash
```
# To debug this pod and show everything
```
kubectl describe pod/nginx-pod -n nginx
```
