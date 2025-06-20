* Basic building block
* Basic building block
* One or more containers in a pod
* Pod containers all share a container network
* One IP address per Pod
# Pod Declaration
* Container Image
* Container Ports
* Container restart policy
* Resource limits.

# Manifest File
* Example manifest
```YAML
apiVersion: v1
kind: Pod
metadata:
	name: mypod
spec:
	containers:
	- name: mycontainer
	  image: nginx:latest
```
* Sent to the API server
```
<kubectl create -f file.yaml
```
* API server does the following:
		1. Select a node with sufficent resources
		2. Schedule Pod onto node
		3. Node pulls Pods container images
		4. Starts Pod's container
* Specify Ports for Access (Can only access from with Kubernetes network)
```YAML
apiVersion: v1
kind: Pod
metadata:
  name: mypod
spec:
  containers:
  - name: mycontainer
    image: nginx:latest
    ports:
      - containerPort: 80
```
* With a label for description:
```YAML
apiVersion: v1
kind: Pod
metadata:
  name: mypod
  labels:
    app: webserver
spec:
  containers:
  - name: mycontainer
    image: nginx:latest
    ports:
      - containerPort: 80
```
* With min and max resources:
* Will be guaranteed these resources in requests else it wont be created.
	* May need to benchmark to find optimum resources.
```YAML
apiVersion: v1
kind: Pod
metadata:
  name: mypod
  labels:
    app: webserver
spec:
  containers:
  - name: mycontainer
    image: nginx:latest
    resources:
      requests:
        memory: "128Mi" # 128Mi = 128 mebibytes
        cpu: "500m"     # 500m = 500 milliCPUs (1/2 CPU)
      limits:
        memory: "128Mi"
        cpu: "500m"
    ports:
      - containerPort: 80
```
