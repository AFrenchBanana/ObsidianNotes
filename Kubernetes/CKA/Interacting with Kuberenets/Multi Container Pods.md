![[Pasted image 20250620103623.png]]
# Namespace
* Separate resources to users environments or applications. 
* Role-Based access control to secure access per Namespace
* Best practice
```YAML
apiVersion: v1
kind: Namespace
metadata:
  name: microservice
  labels:
    app: counter
```
**Once a namespace is created you need to specify the namespace on all create**

```Shell
kubectl create -f file.yaml -n microservice
```

```YAML
apiVersion: v1
kind: Pod
metadata:
  name: app
spec:
  containers:
    - name: redis
      image: redis:latest
      imagePullPolicy: IfNotPresent # only pull if no image is available
      ports:
        - containerPort: 6379

    - name: server
      image: lrakai/microservices:server-v1
      ports:
        - containerPort: 8080
      env:
        - name: REDIS_URL
          value: redis://localhost:6379

    - name: counter
      image: lrakai/microservices:counter-v1
      env:
        - name: API_URL
          value: http://localhost:8080

    - name: poller
      image: lrakai/microservices:poller-v1
      env:
```
* All share same IP so can reach other containers on the port using localhost.
```shell
 kubectl get -n microservice pod app
 NAME   READY   STATUS    RESTARTS   AGE
app    4/4     Running   0          69s
```
Shows the number of containers running/ available.
```Shell
kubectl describe pod -n microservice
```
```
kubectl logs -n microservice app counter
```