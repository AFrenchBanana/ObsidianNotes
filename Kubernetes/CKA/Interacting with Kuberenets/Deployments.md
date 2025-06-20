* Represents multiple replicas of a Pod. 
* Describe a desired state that Kubernetes needs to achieve. 
* Deployment Controller master component converges actual state to desired state.

# Example
* New Namespace
```YAML
apiVersion: v1
kind: Namespace
metadata:
  name: deployments
  labels:
    app: counteru
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
    tier: data
  type: ClusterIP # default
---
apiVersion: apps/v1 # apps API group rather than core api
kind: Deployment # Deployment Kind
metadata:
  name: data-tier
  labels:
    app: microservices
    tier: data
spec:
  replicas: 1 # how many pods to create for this deployment
  selector:
    matchLabels:
      tier: data # this should match the labels below
  template:
    metadata:
      labels: # unique names for each pod in deployment
        app: microservices 
        tier: data
    spec: # Pod spec
      containers:
      - name: redis
        image: redis:latest
        imagePullPolicy: IfNotPresent
        ports:
          - containerPort: 6379
```
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
            # value: redis://localhost:6379
```
```YAML
apiVersion: apps/v1
kind: Deployment
metadata:
  name: support-tier
  labels:
    app: microservices
    tier: support
spec:
  replicas: 1
  selector:
    matchLabels:
      tier: support
  template:
    metadata:
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
              value: http://app-tier.deployments:8080

        - name: poller
          image: lrakai/microservices:poller-v1
          env:
            - name: API_URL
              # omit namespace to only search in the same namespace
              value: http://app-tier:$(APP_TIER_SERVICE_PORT)
```
```Shell
kubectl create -n deployments -f 5.2-data_tier.yaml -f 5.3-app_tier.yaml -f 5.4-support_tier.yaml
```
```
 kubectl get -n deployments deployments
NAME           READY   UP-TO-DATE   AVAILABLE   AGE
app-tier       1/1     1            1           22s
data-tier      1/1     1            1           22s
support-tier   1/1     1            1           22s
```
```Shell
kubectl -n deployments get pods
NAME                            READY   STATUS    RESTARTS      AGE
app-tier-7cb79c78-rvcbt         1/1     Running   1 (53s ago)   55s
data-tier-76c598cd55-qctnf      1/1     Running   0             55s
support-tier-6df7464869-9mkn9   2/2     Running   0             55s
```
* Change support tier to 5 counters
```Shell
 kubectl scale -n deployments deployment support-tier --replicas=5
```
```Shell
 kubectl -n deployments get pods
NAME                            READY   STATUS    RESTARTS       AGE
app-tier-7cb79c78-rvcbt         1/1     Running   1 (112s ago)   114s
data-tier-76c598cd55-qctnf      1/1     Running   0              114s
support-tier-6df7464869-48wjq   2/2     Running   0              27s
support-tier-6df7464869-4nclv   2/2     Running   0              27s
support-tier-6df7464869-9mkn9   2/2     Running   0              114s
support-tier-6df7464869-lngg2   2/2     Running   0              27s
support-tier-6df7464869-ptjmt   2/2     Running   0              27s
```
```
 kubectl delete -n deployments pods support-tier-6df7464869-48wjq support-tier-6df7464869-4nclv
```
* Kubernetes will rescerect these pods back to 5.
```
kubectl scale -n deployments deployment app-tier --replicas=5
```
```
kubectl -n deployments get pods
NAME                            READY   STATUS    RESTARTS       AGE
app-tier-7cb79c78-4bq7b         1/1     Running   0              11s
app-tier-7cb79c78-7f8d7         1/1     Running   0              11s
app-tier-7cb79c78-ddz4f         1/1     Running   0              11s
app-tier-7cb79c78-kbdp2         1/1     Running   0              11s
app-tier-7cb79c78-rvcbt         1/1     Running   1 (4m5s ago)   4m7s
data-tier-76c598cd55-qctnf      1/1     Running   0              4m7s
support-tier-6df7464869-24mv7   2/2     Running   0              69s
support-tier-6df7464869-6hb6f   2/2     Running   0              69s
support-tier-6df7464869-9mkn9   2/2     Running   0              4m7s
support-tier-6df7464869-lngg2   2/2     Running   0              2m40s
support-tier-6df7464869-ptjmt   2/2     Running   0              2m40s
```
```
 kubectl describe -n deployments service app-tier
Name:              app-tier
Namespace:         deployments
Labels:            app=microservices
Annotations:       <none>
Selector:          tier=app
Type:              ClusterIP
IP Family Policy:  SingleStack
IP Families:       IPv4
IP:                10.109.151.246
IPs:               10.109.151.246
Port:              <unset>  8080/TCP
TargetPort:        8080/TCP
Endpoints:         192.168.203.138:8080,192.168.203.143:8080,192.168.203.144:8080 + 2 more...
Session Affinity:  None
Events:            <none>
```
5 IPs in the end points. 

* Make sure the pods your working with support horizontal scaling
	* Make sure they are stateless.

