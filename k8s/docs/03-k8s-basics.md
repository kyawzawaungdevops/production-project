# Kubernetes Tutorial: Deploy Your First Production App

*Learn Kubernetes by deploying a real application with databases, monitoring, and scaling*

## 🎯 **What You'll Learn**

By the end of this tutorial, you'll know how to:
- **Deploy applications** on Kubernetes (the industry standard)
- **Manage databases** with persistent storage
- **Configure services** for internal communication
- **Scale applications** automatically based on demand
- **Monitor health** and troubleshoot issues

## ⏱️ **Time Required: 30-60 minutes**

## Why This Matters

Kubernetes is the industry standard for deploying applications at scale. Companies like Netflix, Airbnb, and GitHub use Kubernetes to manage thousands of services serving millions of users. Learning Kubernetes isn't just about technology—it's about understanding how modern applications are built and deployed.

**What this means for your career**: Kubernetes skills are in high demand. DevOps engineers with Kubernetes experience earn 20-30% more than those without it.

ℹ️ **Simple Explanation:** Kubernetes is like a smart manager for your applications. It automatically handles scaling (adding more copies when busy), health checks (restarting failed services), and updates (rolling out new versions without downtime).

## Do This

### Step 1: Create Your Kubernetes Cluster

```bash
# Create a local 3-node Kubernetes cluster
k3d cluster create dev-cluster \
  --servers 1 \
  --agents 2 \
  --port "8080:80@loadbalancer" \
  --port "8443:443@loadbalancer"
  OR

  k3d cluster create dev-cluster --servers 1 --agents 2 --port "8080:80@loadbalancer" --port "8443:443@loadbalancer" --k3s-arg --disable=traefik@server:0

  OR apply the k3d-config.yaml file
```

**Expected Output:**
```bash
INFO[0000] Prep: Network
INFO[0000] Created network 'k3d-dev-cluster'
INFO[0000] Created volume 'k3d-dev-cluster-images'
INFO[0000] Starting cluster 'dev-cluster'
INFO[0000] Starting the server node
INFO[0000] Starting the agent nodes
INFO[0000] Starting load balancer
INFO[0000] Starting helpers
INFO[0000] Cluster 'dev-cluster' created successfully!
```

```bash
# Verify cluster is running
kubectl get nodes

kubectl get nodes -o wide
# Should show 3 nodes: 1 server, 2 agents, all "Ready"
```

**Expected Output:**
```bash
NAME                    STATUS   ROLES                  AGE   VERSION        INTERNAL-IP     EXTERNAL-IP   OS-IMAGE   KERNEL-VERSION   CONTAINER-RUNTIME
k3d-dev-cluster-server-0   Ready    control-plane,master   2m    v1.28.0+k3s1   172.18.0.2     <none>        Alpine Linux v3.18  6.1.0-13-amd64   containerd://1.7.11
k3d-dev-cluster-agent-0    Ready    <none>                 2m    v1.28.0+k3s1   172.18.0.3     <none>        Alpine Linux v3.18  6.1.0-13-amd64   containerd://1.7.11
k3d-dev-cluster-agent-1    Ready    <none>                 2m    v1.28.0+k3s1   172.18.0.4     <none>        Alpine Linux v3.18  6.1.0-13-amd64   containerd://1.7.11
```

```bash
# Check cluster health
kubectl cluster-info
# Should show cluster endpoint and DNS
```

**Expected Output:**
```bash
Kubernetes control plane is running at https://0.0.0.0:6443
CoreDNS is running at https://0.0.0.0:6443/api/v1/namespaces/kube-system/services/kube-dns:dns/proxy
Metrics-server is running at https://0.0.0.0:6443/api/v1/namespaces/kube-system/services/kube-dns:dns/proxy
```

### Step 2: Deploy Your Application Configuration

```bash
# Create the application namespace (organization)
kubectl apply -f k8s/namespace.yaml
```

**Expected Output:**
```bash
namespace/application created
```

```bash
# Create configuration and secrets
kubectl apply -f k8s/configmap.yaml
kubectl apply -f k8s/secrets.yaml
```

**Expected Output:**
```bash
configmap/application-config created
configmap/frontend-config created
secret/application-secrets created
```

```bash
# Verify they were created
kubectl get configmap -n application
kubectl get secrets -n application
```

**Expected Output:**
```bash
NAME                DATA   AGE
frontend-config     1      30s
application-config   5      30s

NAME                  TYPE     DATA   AGE
application-secrets   Opaque   5      30s
```

### Step 3: Deploy Database Services

```bash
# Deploy PostgreSQL with persistent storage
kubectl apply -f k8s/postgres.yaml
```

**Expected Output:**
```bash
deployment.apps/application-postgres created
service/application-postgres created
persistentvolumeclaim/application-postgres-pvc created
```

```bash
# Deploy Redis for caching
kubectl apply -f k8s/redis.yaml
```

**Expected Output:**
```bash
deployment.apps/application-redis created
service/application-redis created
persistentvolumeclaim/application-redis-pvc created
```

```bash
# Wait for databases to be ready (this takes time!)
kubectl wait --for=condition=ready pod -l app=postgres -n application --timeout=180s
kubectl wait --for=condition=ready pod -l app=redis -n application --timeout=180s

# Verify databases are running
kubectl get pods -n application
# Should show postgres and redis pods with "1/1 Running"
```

**Expected Output:**
```bash
NAME                                    READY   STATUS    RESTARTS   AGE
application-postgres-7d8f9c8f9c-abc12   1/1     Running   0          2m
application-redis-8e9f0d1e2f-def34      1/1     Running   0          2m
```

### Step 4: Build and Deploy Application Services

**⚠️ CRITICAL: Build Images Locally AND Import to k3d!**

```bash
# Build your application images locally
docker build -t application-frontend:latest ./frontend
docker build -t application-backend:latest ./backend
```

**Expected Output:**
```bash
Building frontend
Step 1/8 : FROM nginx:alpine
 ---> 1234567890ab
Step 2/8 : COPY nginx.conf /etc/nginx/nginx.conf
 ---> Using cache
 ---> 1234567890ab
...
Successfully built 1234567890ab
Successfully tagged application-frontend:latest

Building backend
Step 1/12 : FROM node:18-alpine
 ---> 0987654321cd
...
Successfully built 0987654321cd
Successfully tagged application-backend:latest
```

```bash
# Import images into k3d cluster (CRITICAL!)
k3d image import application-frontend:latest -c dev-cluster
k3d image import application-backend:latest -c dev-cluster
```

**Expected Output:**
```bash
Importing image 'application-frontend:latest' into cluster 'dev-cluster'
Importing image 'application-backend:latest' into cluster 'dev-cluster'
```

```bash
# Deploy backend application
kubectl apply -f k8s/backend.yaml
```

**Expected Output:**
```bash
deployment.apps/application-backend created
service/application-backend created
```

```bash
# Deploy frontend application
kubectl apply -f k8s/frontend.yaml
```

**Expected Output:**
```bash
deployment.apps/application-frontend created
service/application-frontend created
```

```bash
# Wait for applications to be ready
kubectl wait --for=condition=ready pod -l app=backend -n application --timeout=180s
kubectl wait --for=condition=ready pod -l app=frontend -n application --timeout=180s

# Verify all pods are running
kubectl get pods -n application
```

**Expected Output:**
```bash
NAME                                    READY   STATUS    RESTARTS   AGE
application-postgres-7d8f9c8f9c-abc12   1/1     Running   0          5m
application-redis-8e9f0d1e2f-def34      1/1     Running   0          5m
application-backend-7d8f9c8f9c-abc12    1/1     Running   0          2m
application-frontend-7d8f9c8f9c-abc12   1/1     Running   0          2m
```

### Step 5: Test Your Kubernetes Application

```bash
# Test backend API through port-forward
kubectl port-forward service/backend 3001:3001 -n application &
```

**Expected Output:**
```bash
Forwarding from 127.0.0.1:3001 -> 3001
```

```bash
# Test API health
curl http://localhost:3001/health
```

**Expected Output:**
```json
{
  "status": "healthy",
  "services": {
    "database": "connected",
    "redis": "connected"
  },
  "timestamp": "2024-01-15T10:30:00.000Z"
}
```

```bash
# Test frontend through port-forward
kubectl port-forward service/frontend 3000:80 -n application &
```

**Expected Output:**
```bash
Forwarding from 127.0.0.1:3000 -> 80
```

## You Should See...

**Cluster Status:**
```bash
NAME                    STATUS   ROLES                  AGE   VERSION
k3d-dev-cluster-server-0   Ready    control-plane,master   5m    v1.28.0+k3s1
k3d-dev-cluster-agent-0    Ready    <none>                 5m    v1.28.0+k3s1
k3d-dev-cluster-agent-1    Ready    <none>                 5m    v1.28.0+k3s1
```

**Application Pods:**
```bash
NAME                                    READY   STATUS    RESTARTS   AGE
application-postgres-7d8f9c8f9c-abc12   1/1     Running   0          5m
application-redis-8e9f0d1e2f-def34      1/1     Running   0          5m
application-backend-7d8f9c8f9c-abc12    1/1     Running   0          2m
application-frontend-7d8f9c8f9c-abc12   1/1     Running   0          2m
```

**Services:**
```bash
NAME                    TYPE        CLUSTER-IP      EXTERNAL-IP   PORT(S)    AGE
application-backend      ClusterIP   10.43.123.45    <none>        3001/TCP   2m
application-frontend     ClusterIP   10.43.234.56    <none>        80/TCP     2m
application-postgres     ClusterIP   10.43.345.67    <none>        5432/TCP   5m
application-redis        ClusterIP   10.43.456.78    <none>        6379/TCP   5m
```

## ✅ Checkpoint

Your Kubernetes application is working when:
- ✅ All 4 pods show "1/1 Running" status
- ✅ Backend API responds at `http://localhost:3001/health`
- ✅ Frontend loads at `http://localhost:3000`
- ✅ Database connections work (no errors in pod logs)
- ✅ Redis connections work (no errors in pod logs)

## If It Fails

### Symptom: Pods stuck in "Pending" status
**Cause:** Insufficient cluster resources or image pull issues
**Command to confirm:** `kubectl describe pod <pod-name> -n application`
**Fix:**
```bash
# Check pod events for specific errors
kubectl describe pod backend-xxx -n application

# Common fix: Ensure images are imported to k3d
k3d image import application-backend:latest -c dev-cluster
k3d image import application-frontend:latest -c dev-cluster
```

### Symptom: Backend pods in "CrashLoopBackOff"
**Cause:** Application startup errors or missing environment variables
**Command to confirm:** `kubectl logs <pod-name> -n application`
**Fix:**
```bash
# Check pod logs for errors
kubectl logs backend-xxx -n application

# Verify secrets and configmaps exist
kubectl get secrets,configmap -n application

# Restart the deployment
kubectl rollout restart deployment/backend -n application
```

### Symptom: Database connection failed
**Cause:** PostgreSQL not ready or service not accessible
**Command to confirm:** `kubectl logs <postgres-pod> -n application`
**Fix:**
```bash
# Check database logs
kubectl logs application-postgres-xxx -n application

# Verify service is accessible
kubectl exec -it backend-xxx -n application -- env | grep DB_HOST

# Wait longer for database initialization
kubectl wait --for=condition=ready pod -l app=postgres -n application --timeout=300s
```

### Symptom: Frontend shows "Cannot connect to game server"
**Cause:** Backend service not accessible or CORS issues
**Command to confirm:** `kubectl get svc -n application`
**Fix:**
```bash
# Verify backend service exists
kubectl get svc application-backend -n application

# Check if backend pods are ready
kubectl get pods -l app=backend -n application

# Test service connectivity
kubectl exec -it application-frontend-xxx -n application -- curl http://application-backend:3001/health
```

### Symptom: Network connectivity issues between k3d nodes
**Cause:** Cluster network configuration problems or node communication failures
**Command to confirm:** `kubectl logs <pod-name> -n application` shows "proxy error" or "502 Bad Gateway"
**Fix:**
```bash
# Check cluster node status
kubectl get nodes -o wide

# If nodes show Ready but pods can't communicate, recreate cluster
k3d cluster delete dev-cluster
k3d cluster create dev-cluster --servers 1 --agents 2 --port "8080:80@loadbalancer" --port "8443:443@loadbalancer" --k3s-arg --disable=traefik@server:0

# Redeploy all components
kubectl apply -f k8s/namespace.yaml
kubectl apply -f k8s/configmap.yaml
kubectl apply -f k8s/secrets.yaml
kubectl apply -f k8s/postgres.yaml
kubectl apply -f k8s/redis.yaml
```

### Symptom: Frontend nginx proxy not routing API calls to backend
**Cause:** Missing nginx proxy configuration for `/api/*` endpoints
**Command to confirm:** `curl http://localhost:3000/api/health` returns 404 or frontend shows "Cannot connect to game server"
**Fix:**
```bash
# Update frontend nginx.conf to include API proxy
# Add this location block to nginx.conf:
location /api/ {
    proxy_pass http://backend:3001;
    proxy_http_version 1.1;
    proxy_set_header Host $host;
    proxy_set_header X-Real-IP $remote_addr;
    proxy_set_header X-Forwarded-For $proxy_add_x_forwarded_for;
    proxy_set_header X-Forwarded-Proto $scheme;
}

# Rebuild and redeploy frontend
docker build -t application-frontend:latest ./frontend
k3d image import application-frontend:latest -c dev-cluster
kubectl rollout restart deployment/frontend -n application
```

### Symptom: Service names mismatch between deployment and port-forward
**Cause:** Services created with different names than expected (e.g., `backend` vs `application-backend`)
**Command to confirm:** `kubectl get svc -n application` shows different service names
**Fix:**
```bash
# Check actual service names
kubectl get svc -n application

# Use correct service names for port-forward
kubectl port-forward service/backend 3001:3001 -n application &
kubectl port-forward service/frontend 3000:80 -n application &

# Or update service names in YAML files to match expectations
```

## 💡 **Reset/Rollback Commands**

If you need to start over or fix issues:

**Common Error Messages & Solutions:**
- **"Error from server (NotFound): services 'application-backend' not found"** → Use actual service names: `backend`, `frontend`
- **"proxy error from 127.0.0.1:6443 while dialing 172.18.0.5:10250, code 502"** → Recreate cluster due to network issues
- **"Cannot connect to game server" in frontend** → Missing nginx API proxy configuration
- **"connect ECONNREFUSED 10.43.x.x:5432"** → Database not ready, wait for postgres pod

```bash
# Delete all resources in the namespace
kubectl delete namespace application

# Recreate namespace
kubectl apply -f k8s/namespace.yaml

# Restart specific deployment
kubectl rollout restart deployment/backend -n application

# Scale deployment down and up
kubectl scale deployment backend --replicas=0 -n application
kubectl scale deployment hbackend --replicas=1 -n application

# View logs for troubleshooting
kubectl logs -f deployment/backend -n application
kubectl logs -f deployment/frontend -n application
```

## Clean Up Before Moving Forward

```bash
# Stop port-forwarding (if running)
pkill -f "kubectl port-forward"

# Verify everything is still running
kubectl get pods -n application
# Should show all pods in Running status
```

## What You Learned

You've successfully transformed your Docker Compose application into Kubernetes, including:
- **Cluster management** with k3d (lightweight Kubernetes)
- **Resource definitions** (Pods, Services, Deployments)
- **Configuration management** (ConfigMaps, Secrets)
- **Persistent storage** (PersistentVolumeClaims)
- **Service discovery** (ClusterIP services)
- **Health monitoring** (pod status, logs)

## Professional Skills Gained

- **Kubernetes deployment** fundamentals
- **Multi-container orchestration** in production-like environment
- **Resource management** and scaling
- **Service mesh** concepts (internal communication)
- **Configuration management** best practices
- **Troubleshooting** Kubernetes applications

---

*Kubernetes basics milestone completed successfully. Application running in cluster, ready for [04-ingress.md](04-ingress.md).*
