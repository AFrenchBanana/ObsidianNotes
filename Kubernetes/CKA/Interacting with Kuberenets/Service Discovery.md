* Split containers into different services. 
![[Pasted image 20250620110026.png]]
* Supports Multi-pod Design
* Handles Pod IP changes
* Load Balancing
# Service Discovery Mechanisms
## Environment Variables 
* Services address automatically injected into containers
* Follow naming conventions 
## DNS
* Automatically created in clusters DNS
* Contains automatically configured to query cluster DNS.



# Example
Make a new name space. 
```YAML
apiVersion: v1
kind: Namespace
metadata:
  name: service-discovery
  labels:
    app: counter
```
```Shell
 kubectl create -f 4.1-namespace.yaml
```


```YAML
apiVersion: v1
kind: Service
metadata:
  name: data-tier
  labels:
    app: microservices
spec:
  ports:
  - port: 6379
    protocol: TCP # default
    name: redis # optional when only 1 port
  selector:
    tier: data # selector is the pods selector
  type: ClusterIP # internal access only
--- # 3 lines creates a new data type
apiVersion: v1
kind: Pod
metadata:
  name: data-tier
  labels:
    app: microservices
    tier: data 
spec:
  containers:
    - name: redis
      image: redis:latest
      imagePullPolicy: IfNotPresent
      ports:
        - containerPort: 6379
```
* Created in the order of the yaml file.
```YAML
kubectl describe service -n service-discovery data-tier
Name:              data-tier
Namespace:         service-discovery
Labels:            app=microservices
Annotations:       <none>
Selector:          tier=data
Type:              ClusterIP
IP Family Policy:  SingleStack
IP Families:       IPv4
IP:                10.100.191.28
IPs:               10.100.191.28
Port:              redis  6379/TCP
TargetPort:        6379/TCP
Endpoints:         192.168.203.135:6379
Session Affinity:  None
Events:            <none>
```
* Make sure there is a port on the end point.
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
apiVersion: v1
kind: Pod
metadata:
  name: app-tier
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
          # value: redis://localhost:6379 
```
* environment variables set here to get the service IP.  This allows for scaling across services. 
* Environment variables only get set up at container boot, and need to be in the same namespace
* DNS can work across name spaces.
```YAML
apiVersion: v1
kind: Pod
metadata:
  name: support-tier
  labels:
    app: microservices
    tier: support
spec:
  containers:

    - name: counter
      image: lrakai/microservices:counter-v1
      env:
        - name: API_URL
          # DNS for service discovery
          # Naming pattern:
          #   IP address: <service_name>.<service_namespace>
          #   Port: needs to be extracted from SRV DNS record
          value: http://app-tier.service-discovery:8080

    - name: poller
      image: lrakai/microservices:poller-v1
      env:
        - name: API_URL
          # omit namespace to only search in the same namespace
          value: http://app-tier:$(APP_TIER_SERVICE_PORT)
```
* DNS for service discover. 
```Shell
 kubectl get pods -n service-discovery
NAME           READY   STATUS    RESTARTS   AGE
app-tier       1/1     Running   0          110s
data-tier      1/1     Running   0          6m3s
support-tier   2/2     Running   0          66s
```

