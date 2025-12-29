# Kubernetes Resource Quotas and Limits Guide

## Overview
Resource management in Kubernetes ensures that containers have enough resources to run while preventing any single container from consuming all available cluster resources. This guide explains how to configure and use resource requests and limits effectively.

**Think of it like this:** Imagine an apartment building with limited water and electricity. Resource requests are like guaranteeing each apartment gets a minimum amount, while limits prevent any apartment from using so much that others suffer.

---

## Understanding Resources in Your Deployment

### Your Current Configuration
Looking at your `deployment.yml`, you have:

```yaml
resources:
  requests:
    cpu: 100m
    memory: 128Mi
  limits:
    cpu: 200m
    memory: 256Mi
```

### What Are Resource Requests?

**Resource Requests** are the minimum guaranteed resources that Kubernetes will allocate to your container.

- **Purpose**: Tells the Kubernetes scheduler how much CPU and memory your container needs to function properly
- **Guarantee**: Your container will always have at least this amount available
- **Scheduling**: Kubernetes won't schedule your pod on a node that doesn't have these resources available

**In your deployment:**
- `cpu: 100m` = 100 millicores (0.1 CPU cores)
- `memory: 128Mi` = 128 Mebibytes of RAM

### What Are Resource Limits?

**Resource Limits** are the maximum resources your container can use.

- **Purpose**: Prevents containers from consuming excessive resources
- **Protection**: Protects other containers and the node from resource starvation
- **Enforcement**: If a container tries to exceed limits, it will be throttled (CPU) or terminated (memory)

**In your deployment:**
- `cpu: 200m` = 200 millicores (0.2 CPU cores maximum)
- `memory: 256Mi` = 256 Mebibytes of RAM maximum

---

## CPU Resource Units

### Understanding Millicores

CPU resources are measured in "cores" or "millicores" (m):

- **1 CPU core** = 1000 millicores (1000m)
- **100m** = 0.1 cores = 10% of one CPU core
- **500m** = 0.5 cores = 50% of one CPU core
- **1000m** = 1 core = 100% of one CPU core

### Examples

```yaml
cpu: "100m"   # 10% of a CPU core
cpu: "500m"   # 50% of a CPU core
cpu: "1"      # 1 full CPU core
cpu: "2"      # 2 full CPU cores
```

### When CPU Limits Are Exceeded

When a container tries to use more CPU than its limit:
- The container is **throttled** (slowed down)
- The container continues to run but performs slower
- No crashes or restarts occur

---

## Memory Resource Units

### Understanding Memory Units

Memory can be specified in several units:

- **Ki** = Kibibytes (1024 bytes)
- **Mi** = Mebibytes (1024 Ki = 1,048,576 bytes)
- **Gi** = Gibibytes (1024 Mi = 1,073,741,824 bytes)

### Examples

```yaml
memory: "64Mi"    # 64 Mebibytes
memory: "128Mi"   # 128 Mebibytes
memory: "256Mi"   # 256 Mebibytes
memory: "1Gi"     # 1 Gibibyte
memory: "2Gi"     # 2 Gibibytes
```

### When Memory Limits Are Exceeded

When a container tries to use more memory than its limit:
- The container is **terminated** (killed)
- Kubernetes shows `OOMKilled` (Out Of Memory Killed) status
- The container will be restarted according to the restart policy

---

## How to Choose the Right Values

### Step 1: Understand Your Application

**Ask yourself:**
1. What does my application do? (web server, database, batch processing)
2. How much traffic will it handle?
3. What are the performance requirements?

### Step 2: Start with Conservative Estimates

For a basic web application like nginx:

**Light traffic (personal projects):**
```yaml
requests:
  cpu: 50m
  memory: 64Mi
limits:
  cpu: 100m
  memory: 128Mi
```

**Medium traffic (small business):**
```yaml
requests:
  cpu: 100m
  memory: 128Mi
limits:
  cpu: 200m
  memory: 256Mi
```

**High traffic (production):**
```yaml
requests:
  cpu: 500m
  memory: 512Mi
limits:
  cpu: 1000m
  memory: 1Gi
```

### Step 3: Monitor and Adjust

After deployment, monitor your pods to see actual resource usage:

```bash
# View real-time resource usage
kubectl top pods -n nginx

# Describe pod to see resource allocation
kubectl describe pod <pod-name> -n nginx
```

---

## Practical Examples

### Example 1: Development Environment

For testing and development:

```yaml
resources:
  requests:
    cpu: 50m      # Minimal CPU needed
    memory: 64Mi  # Small memory footprint
  limits:
    cpu: 100m     # Prevent runaway processes
    memory: 128Mi # Protect other dev workloads
```

**When to use:** Local development, feature testing, CI/CD pipelines

### Example 2: Production Web Server

For serving real user traffic:

```yaml
resources:
  requests:
    cpu: 200m     # Ensures responsive performance
    memory: 256Mi # Handles concurrent requests
  limits:
    cpu: 500m     # Room for traffic spikes
    memory: 512Mi # Prevents memory leaks from crashing node
```

**When to use:** Production websites, REST APIs, microservices

### Example 3: Background Job Processor

For batch processing or queue workers:

```yaml
resources:
  requests:
    cpu: 100m     # Can wait if CPU is busy
    memory: 256Mi # Processes data in memory
  limits:
    cpu: 1000m    # Can use full core when available
    memory: 1Gi   # Large data processing
```

**When to use:** Data processing, report generation, email sending

### Example 4: Database Container

For databases like PostgreSQL or MySQL:

```yaml
resources:
  requests:
    cpu: 500m     # Consistent query performance
    memory: 1Gi   # Database caching
  limits:
    cpu: 2000m    # Complex queries need CPU
    memory: 2Gi   # Large result sets
```

**When to use:** Databases, caching layers, search engines

---

## Resource Requests vs Limits Best Practices

### The Golden Rules

1. **Always set both requests and limits**
   - Requests alone: No upper bound protection
   - Limits alone: May not get scheduled efficiently

2. **Limits should be higher than requests**
   ```yaml
   requests:
     cpu: 100m
     memory: 128Mi
   limits:
     cpu: 200m      # 2x the request
     memory: 256Mi  # 2x the request
   ```

3. **Keep the ratio reasonable**
   - Good: Limit is 2-4x the request
   - Avoid: Limit is 10x+ the request (leads to resource wastage)

4. **Match application patterns**
   - Steady workload: Requests ≈ Limits
   - Bursty workload: Limits > Requests (allow headroom)

### Common Mistakes to Avoid

❌ **Setting limits too low**
```yaml
resources:
  requests:
    cpu: 100m
  limits:
    cpu: 100m  # No room for bursts
```

❌ **Not setting requests**
```yaml
resources:
  limits:
    cpu: 200m
    memory: 256Mi
  # No requests defined - scheduler doesn't know minimum needs
```

❌ **Huge gap between requests and limits**
```yaml
resources:
  requests:
    cpu: 10m
  limits:
    cpu: 2000m  # 200x difference - wasteful
```

✅ **Good balance**
```yaml
resources:
  requests:
    cpu: 100m
    memory: 128Mi
  limits:
    cpu: 200m      # 2x headroom
    memory: 256Mi  # 2x headroom
```

---

## Monitoring Resource Usage

### Check Current Resource Consumption

```bash
# Install metrics server (if not already installed)
kubectl apply -f https://github.com/kubernetes-sigs/metrics-server/releases/latest/download/components.yaml

# View resource usage for all pods
kubectl top pods -n nginx

# View resource usage for specific pod
kubectl top pod <pod-name> -n nginx

# View node resource usage
kubectl top nodes
```

### Expected Output

```
NAME                                CPU(cores)   MEMORY(bytes)
nginx-deployment-6f59995b97-7k8lq   5m          50Mi
nginx-deployment-6f59995b97-thqxp   4m          48Mi
```

### Analyzing the Results

**If usage is consistently near limits:**
- Your containers are resource-constrained
- Consider increasing limits
- May see performance degradation or OOMKilled events

**If usage is far below requests:**
- You're wasting resources
- Consider lowering requests to save cluster capacity
- Other workloads could use those resources

---

## Troubleshooting Resource Issues

### Problem 1: Pod Stuck in Pending State

**Symptom:**
```bash
kubectl get pods -n nginx
NAME                                READY   STATUS    RESTARTS   AGE
nginx-deployment-6f59995b97-abc123  0/1     Pending   0          5m
```

**Cause:** Not enough resources on any node to satisfy requests

**Solution:**
```bash
# Check pod events
kubectl describe pod <pod-name> -n nginx

# Look for: "Insufficient cpu" or "Insufficient memory"

# Options:
# 1. Reduce resource requests
# 2. Add more nodes to cluster
# 3. Remove or scale down other workloads
```

### Problem 2: OOMKilled (Out of Memory)

**Symptom:**
```bash
kubectl get pods -n nginx
NAME                                READY   STATUS      RESTARTS   AGE
nginx-deployment-6f59995b97-abc123  0/1     OOMKilled   5          10m
```

**Cause:** Container exceeded memory limit

**Solution:**
```bash
# Check memory usage before crash
kubectl describe pod <pod-name> -n nginx

# Increase memory limit:
limits:
  memory: 512Mi  # Increase from 256Mi
```

### Problem 3: CPU Throttling

**Symptom:** Application is slow but not crashing

**Cause:** Container hitting CPU limit and being throttled

**Check:**
```bash
# Look at CPU usage
kubectl top pod <pod-name> -n nginx

# If consistently at or near limit, increase:
limits:
  cpu: 500m  # Increase from 200m
```

---

## Quality of Service (QoS) Classes

Kubernetes assigns QoS classes based on your resource configuration. This determines which pods get evicted first when nodes run out of resources.

### Guaranteed (Highest Priority)

**Configuration:**
```yaml
resources:
  requests:
    cpu: 100m
    memory: 128Mi
  limits:
    cpu: 100m      # Same as request
    memory: 128Mi  # Same as request
```

**When to use:** Critical production workloads that must never be evicted

### Burstable (Medium Priority)

**Configuration:**
```yaml
resources:
  requests:
    cpu: 100m
    memory: 128Mi
  limits:
    cpu: 200m      # Higher than request
    memory: 256Mi  # Higher than request
```

**When to use:** Most production workloads (your current configuration)

### BestEffort (Lowest Priority)

**Configuration:**
```yaml
# No resources defined
```

**When to use:** Non-critical batch jobs, can be killed anytime

---

## Advanced: Creating ResourceQuotas

ResourceQuotas limit the total resources a namespace can consume.

### Create a ResourceQuota File

Create `resource-quota.yml`:

```yaml
apiVersion: v1
kind: ResourceQuota
metadata:
  name: nginx-quota
  namespace: nginx
spec:
  hard:
    requests.cpu: "4"
    requests.memory: 8Gi
    limits.cpu: "8"
    limits.memory: 16Gi
    pods: "10"
```

### Apply the Quota

```bash
kubectl apply -f resource-quota.yml
```

### Check Quota Usage

```bash
kubectl describe resourcequota nginx-quota -n nginx
```

### What This Does

- **Namespace-level limits:** Prevents any namespace from using too many resources
- **Forces resource definition:** Pods without resources defined will be rejected
- **Capacity planning:** Helps manage multi-tenant clusters

---

## Quick Reference

### Common Resource Values

| Application Type | CPU Request | CPU Limit | Memory Request | Memory Limit |
|-----------------|-------------|-----------|----------------|--------------|
| Static Website  | 10m         | 50m       | 32Mi           | 64Mi         |
| Web Server      | 100m        | 200m      | 128Mi          | 256Mi        |
| API Server      | 200m        | 500m      | 256Mi          | 512Mi        |
| Database        | 500m        | 2000m     | 1Gi            | 2Gi          |
| Batch Job       | 100m        | 1000m     | 256Mi          | 1Gi          |

### Useful Commands

```bash
# View resource usage
kubectl top pods -n nginx

# Describe pod resources
kubectl describe pod <pod-name> -n nginx

# Edit deployment resources
kubectl edit deployment nginx-deployment -n nginx

# Scale deployment
kubectl scale deployment nginx-deployment --replicas=3 -n nginx

# View resource quotas
kubectl describe quota -n nginx
```

---

## Summary

You've learned how to:
1. ✅ Set resource requests to guarantee minimum resources
2. ✅ Set resource limits to prevent overconsumption
3. ✅ Choose appropriate values for different workload types
4. ✅ Monitor and troubleshoot resource issues
5. ✅ Understand QoS classes and their impact

## Key Takeaways

- **Requests** = Minimum guarantee (affects scheduling)
- **Limits** = Maximum allowed (affects runtime)
- **CPU** = Throttled when exceeded
- **Memory** = Killed (OOMKilled) when exceeded
- **Monitor** = Use `kubectl top` to track usage
- **Adjust** = Increase if constrained, decrease if wasteful

---

## Additional Resources

- [Kubernetes Resource Management](https://kubernetes.io/docs/concepts/configuration/manage-resources-containers/)
- [Quality of Service Classes](https://kubernetes.io/docs/tasks/configure-pod-container/quality-service-pod/)
- [Resource Quotas](https://kubernetes.io/docs/concepts/policy/resource-quotas/)
- [Vertical Pod Autoscaler](https://github.com/kubernetes/autoscaler/tree/master/vertical-pod-autoscaler) - Automatically adjusts resources

---

**Remember:** Resource management is about finding the right balance between performance, cost, and reliability. Start conservative, monitor actively, and adjust based on real-world usage patterns!