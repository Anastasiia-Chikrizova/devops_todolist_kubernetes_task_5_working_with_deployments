# INSTRUCTION.md

## Deploying the Application to Kubernetes

### Step 1: Prepare the Resources
To deploy the application, you will need to create the following Kubernetes resources:
- Deployment
- Service
- HorizontalPodAutoscaler (HPA)

All resources should be described in YAML manifest files.

---

### Step 2: Define Resource Requests and Limits
In the `Deployment` manifest, you need to specify **resource requests** and **limits** for the containers. This ensures the application operates stably and manages resource usage in the cluster effectively.

Example configuration:
```yaml
resources:
  requests:
    cpu: "500m"
    memory: "256Mi"
  limits:
    cpu: "1"
    memory: "512Mi"
```

**Explanation:**
- **`requests`:** The minimum amount of resources required for the application to work. We specify `500m` for CPU and `256Mi` for memory to ensure the application has sufficient resources to handle standard workloads.
- **`limits`:** These define the maximum resources the container can utilize. We set `1` for CPU and `512Mi` for memory to prevent overuse of cluster resources and allow smooth scaling under peak loads.

---

### Step 3: Configure HorizontalPodAutoscaler (HPA)
The HPA automatically adjusts the number of pods in your application based on workload metrics such as CPU utilization.

Example HPA configuration:
```yaml
apiVersion: autoscaling/v2
kind: HorizontalPodAutoscaler
metadata:
  name: my-app-hpa
spec:
  scaleTargetRef:
    apiVersion: apps/v1
    kind: Deployment
    name: my-app-deployment
  minReplicas: 2
  maxReplicas: 10
  metrics:
    - type: Resource
      resource:
        name: cpu
        target:
          type: Utilization
          averageUtilization: 75
```

**Explanation:**
- **`minReplicas: 2`**: A minimum of 2 replicas ensures high availability of the application.
- **`maxReplicas: 10`**: A maximum of 10 replicas to prevent unbounded resource consumption.
- **`averageUtilization: 75%`**: Pods will scale up if CPU utilization exceeds 75% of the allocated requests.

---

### Step 4: Define Deployment Update Strategy
The `Deployment` resource in Kubernetes supports update strategies like `RollingUpdate`, which replaces pods gradually, minimizing downtime during updates.

Example strategy configuration:
```yaml
strategy:
  type: RollingUpdate
  rollingUpdate:
    maxUnavailable: 1
    maxSurge: 2
```

**Strategy Explanation:**
- **`maxUnavailable: 1`**: During an update, at most 1 pod can be temporarily unavailable.
- **`maxSurge: 2`**: During an update, up to 2 extra pods can be created temporarily.
  This configuration minimizes risks during updates and ensures there’s no interruption to the service's availability.

---

### Step 5: Create a Kubernetes Service to Access the Application
To make the application accessible, create a `Service` resource. Depending on requirements, you may choose `ClusterIP`, `NodePort`, or `LoadBalancer`.

Example Service configuration:
```yaml
apiVersion: v1
kind: Service
metadata:
  name: my-app-service
spec:
  selector:
    app: my-app
  type: LoadBalancer
  ports:
    - protocol: TCP
      port: 80
      targetPort: 8080
```

**Explanation:**
- **`type: LoadBalancer`**: This type is suitable for exposing the application to external users.
- **`port: 80`**: The public-facing port.
- **`targetPort: 8080`**: The port exposed by the application inside the container.

---

### Step 6: Deploy the Application
1. Apply the YAML manifests:
   ```bash
   kubectl apply -f deployment.yaml
   kubectl apply -f service.yaml
   kubectl apply -f hpa.yaml
   ```

2. Check the status of resources:
   ```bash
   kubectl get pods
   kubectl get svc
   ```

---

### Step 7: Access the Application
Once deployed successfully:
- If a `LoadBalancer` service is used, obtain the external IP with the following command:
  ```bash
  kubectl get svc my-app-service
  ```
- Access the application in a browser or via a tool like `curl` using:
  ```
  http://<EXTERNAL-IP>
  ```

For a `NodePort` service, use the node's IP address and port.

---

## Summary
By following these steps, you will successfully deploy the application to Kubernetes, configure auto-scaling, and ensure stable operational behavior under various workloads.