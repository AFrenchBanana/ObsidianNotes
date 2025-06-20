* Specify a designed metric
* Set a target min and max
* Target CPI is expressed as a percentage of the Pods CPU request
* Will never create more or less than the min and max. 

# Metrics
* Metrics server is a solution for collecting metrics
* Several manifest files are used to deploy metric servers. 
https://github.com/kubernetes-sigs/metric-server

```Shell
kubectl apply -f metrics-server/
serviceaccount/metrics-server unchanged
clusterrole.rbac.authorization.k8s.io/system:aggregated-metrics-reader unchanged
clusterrole.rbac.authorization.k8s.io/system:metrics-server unchanged
rolebinding.rbac.authorization.k8s.io/metrics-server-auth-reader unchanged
clusterrolebinding.rbac.authorization.k8s.io/metrics-server:system:auth-delegator unchanged
clusterrolebinding.rbac.authorization.k8s.io/system:metrics-server unchanged
service/metrics-server unchanged
deployment.apps/metrics-server configured
apiservice.apiregistration.k8s.io/v1beta1.metrics.k8s.io unchanged
```

* Check metrics 
```Shell
kubectl top pods -n deployments
NAME                            CPU(cores)   MEMORY(bytes)
app-tier-7cb79c78-4bq7b         2m           45Mi
app-tier-7cb79c78-7f8d7         3m           45Mi
app-tier-7cb79c78-ddz4f         2m           45Mi
app-tier-7cb79c78-kbdp2         2m           44Mi
app-tier-7cb79c78-rvcbt         4m           42Mi
data-tier-76c598cd55-qctnf      5m           4Mi
support-tier-6df7464869-24mv7   9m           0Mi
support-tier-6df7464869-6hb6f   10m          0Mi
support-tier-6df7464869-9mkn9   10m          0Mi
support-tier-6df7464869-ptjmt   10m          0Mi
```
* Each pod has number of resources and replicas. 
```YAML
apiVersion: v1
kind: Service
metadata:
  name: app-tier
  labels:
    app: microservices
spec:
  ports:
  - port: 8080
  selector:
    tier: app
---
apiVersion: apps/v1
kind: Deployment
metadata:
  name: app-tier
  labels:
    app: microservices
    tier: app
spec:
  replicas: 1
  selector:
    matchLabels:
      tier: app
  template:
    metadata:
      labels:
        app: microservices
        tier: app
    spec:
      containers:
      - name: server
        image: lrakai/microservices:server-v1
        ports:
          - containerPort: 8080
        env:
          - name: REDIS_URL
            # Environment variable service discovery
            # Naming pattern:
            #   IP address: <all_caps_service_name>_SERVICE_HOST
            #   Port: <all_caps_service_name>_SERVICE_PORT
            #   Named Port: <all_caps_service_name>_SERVICE_PORT_<all_caps_port_name>
            value: redis://$(DATA_TIER_SERVICE_HOST):$(DATA_TIER_SERVICE_PORT_REDIS)
            # In multi-container example value was
            # value: redis://localhost:6379](<apiVersion: v1
kind: Service
metadata:
  name: app-tier
  labels:
    app: microservices
spec:
  ports:
  - port: 8080
  selector:
    tier: app
---
apiVersion: apps/v1
kind: Deployment
metadata:
  name: app-tier
  labels:
    app: microservices
    tier: app
spec:
  replicas: 5
  selector:
    matchLabels:
      tier: app
  template:
    metadata:
      labels:
        app: microservices
        tier: app
    spec:
      containers:
      - name: server
        image: lrakai/microservices:server-v1
        ports:
          - containerPort: 8080
        resources:
          requests:
            cpu: 20m # 20 milliCPU / 0.02 CPU
        env:
          - name: REDIS_URL
            # Environment variable service discovery
            # Naming pattern:
            #   IP address: %3Call_caps_service_name%3E_SERVICE_HOST
            #   Port: <all_caps_service_name>_SERVICE_PORT
            #   Named Port: <all_caps_service_name>_SERVICE_PORT_<all_caps_port_name>
            value: redis://$(DATA_TIER_SERVICE_HOST):$(DATA_TIER_SERVICE_PORT_REDIS)
            # In multi-container example value was
            # value: redis://localhost:6379>)
```

* Apply will update already created pods. 
```Shell
 kubectl apply -f 6.1-app_tier_cpu_request.yaml -n deployments
```
```Shell
kubectl get -n deployments deployments app-tier
NAME       READY   UP-TO-DATE   AVAILABLE   AGE
app-tier   5/5     5            5           13m
```

Horizontal autoscaler
```YAML
apiVersion: autoscaling/v1
kind: HorizontalPodAutoscaler
metadata:
  name: app-tier
  labels:
    app: microservices
    tier: app
spec:
  maxReplicas: 5
  minReplicas: 1
  scaleTargetRef:
    apiVersion: apps/v1
    kind: Deployment
    name: app-tier
  targetCPUUtilizationPercentage: 70
# Equivalent to
# kubectl autoscale deployment app-tier --max=5 --min=1 --cpu-percent=70
```
```Shell
kubectl describe hpa
Name:                                                  app-tier
Namespace:                                             default
Labels:                                                app=microservices
                                                       tier=app
Annotations:                                           <none>
CreationTimestamp:                                     Fri, 20 Jun 2025 10:34:38 +0000
Reference:                                             Deployment/app-tier
Metrics:                                               ( current / target )
  resource cpu on pods  (as a percentage of request):  <unknown> / 70%
Min replicas:                                          1
Max replicas:                                          5
Deployment pods:                                       0 current / 0 desired
Conditions:
  Type         Status  Reason          Message
  ----         ------  ------          -------
  AbleToScale  False   FailedGetScale  the HPA controller was unable to get the target's current scale: deployments/scale.apps "app-tier" not found
Events:
  Type     Reason          Age                From                       Message
  ----     ------          ----               ----                       -------
  Warning  FailedGetScale  15s (x2 over 30s)  horizontal-pod-autoscaler  deployments/scale.apps "app-tier" not found](<kubectl describe hpa
Name:                                                  app-tier
Namespace:                                             default
Labels:                                                app=microservices
                                                       tier=app
Annotations:                                           %3Cnone%3E
CreationTimestamp:                                     Fri, 20 Jun 2025 10:34:38 +0000
Reference:                                             Deployment/app-tier
Metrics:                                               ( current / target )
  resource cpu on pods  (as a percentage of request):  <unknown> / 70%
Min replicas:                                          1
Max replicas:                                          5
Deployment pods:                                       0 current / 0 desired
Conditions:
  Type         Status  Reason          Message
  ----         ------  ------          -------
  AbleToScale  False   FailedGetScale  the HPA controller was unable to get the target's current scale: deployments/scale.apps "app-tier" not found
Events:
  Type     Reason          Age                From                       Message
  ----     ------          ----               ----                       -------
  Warning  FailedGetScale  14s (x6 over 89s)  horizontal-pod-autoscaler  deployments/scale.apps "app-tier" not found>)
```
Get current resources. 
```Shell
 kubectl get -n deployments hpa
NAME       REFERENCE             TARGETS        MINPODS   MAXPODS   REPLICAS   AGE
app-tier   Deployment/app-tier   cpu: 16%/70%   
```