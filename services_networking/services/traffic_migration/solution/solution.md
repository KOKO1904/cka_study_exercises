# Hybrid Service and Traffic Migration

### First we create a clusterIP service for the new backend

```bash
apiVersion: v1
kind: Service
metadata:
  name: backend-v2-service
spec:
  type: ClusterIP
  selector:
    app: backend-v2
  ports:
  - port: 5000
    targetPort: 5000
```
### Next, we create a Service for the legacy backend. Since the backend is external, the endpoints will be defined manually.

```bash
# Service for the legacy backend
apiVersion: v1
kind: Service
metadata:
  name: backend-v1-service
spec:
  type: ClusterIP
  ports:
  - port: 8080
    targetPort: 8080

# We use this command to check the ip of the pods:
NAME                          READY   STATUS    RESTARTS   AGE   IP           NODE       NOMINATED NODE   READINESS GATES
backend-v1-55f8668d8b-j6v2n   1/1     Running   0          47m   10.244.0.5   minikube   <none>           <none>
backend-v2-d875f6c69-hsgc2    1/1     Running   0          47m   10.244.0.3   minikube   <none>           <none>
backend-v2-d875f6c69-mz8vc    1/1     Running   0          47m   10.244.0.4   minikube   <none>           <none>
frontend-8447956b55-zbj9f     1/1     Running   0          47m   10.244.0.6   minikube   <none>           <none>

#Endpoints
apiVersion: v1
kind: Endpoints
metadata:
  name: backend-v1-service
subnets:
- addresses:
  - ip: 10.244.0.5 # Change the ip here with the ip of your pod
  ports:
  - port: 8080

```

### Since the hybrid service will connect to our ClusterIP services, we need to manually specify the endpoints.

```bash
# Hybrid Service
apiVersion: v1
kind: Service
metadata:
  name: hybrid-service
spec:
  type: ClusterIP
  ports:
  - name: legacy
    targetPort: 8080
    port: 8080
  - name: new
    targetPort: 5000
    port: 5000

# Hybrid Service Endpoints
apiVersion: v1
kind: Endpoints
metadata:
  name: hybrid-service
subsets:
- addresses:
  - ip: 10.244.0.5 # Change the ip here with the ip of your pod
  ports:
  - port: 8080
- addresses:
  - ip: 10.244.0.3 # Change the ip here with the ip of your pod
  - ip: 10.244.0.4 # Change the ip here with the ip of your pod
  ports:
  - port: 5000
```
### After that, we create the LoadBalancer service for the frontend deployment

```bash
apiVersion: v1
kind: Service
metadata:
  name: frontend-service
spec:
  type: LoadBalancer
  selector:
    app: frontend
  ports:
  - port: 80
    targetPort: 3000
    nodePort: 32001

```

### Finally, we validate that all components are working correctly

```bash
# First, we check that all pods are running, we also check services
kubectl get pods -o wide
kubectl get svc

# We test the hybrid service endpoints from inside the cluster(Since the frontend image is nginx, we can use curl to test the endpoints).
kubectl exec -it <frontend-pod> -- curl http://hybrid-service:8080
kubectl exec -it <frontend-pod> -- curl http://hybrid-service:5000

# Test the frontend via LoadBalancer
minikube tunnel
kubectl get svc frontend-service

NAME               TYPE           CLUSTER-IP      EXTERNAL-IP    PORT(S)        AGE
frontend-service   LoadBalancer   10.96.42.101   10.244.0.10   80:32001/TCP     5m

# We can use the external IP to access it in the browser.
http://10.244.0.10

# Finally we validate “50/50” routing
kubectl exec -it <frontend-pod> -- sh -c "for i in {1..10}; do curl http://hybrid-service:8080; done"
```
