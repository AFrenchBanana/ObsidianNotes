# Kubernetes REST API
* It is possible but not common to work directly with the API server
* You might need to if there is no Kubernetes client library for your programming language. 
# Client Libraries
* Handles Auth and managing REST API request/responses.
* Official libraries for:
	* Go
	* Python
	* .NET
	* Java
	* Javascript
* Many community maintained libraries. 
# Kubectl
*kubecontrol*
* Issue high level commands that are translate into REST API calls.
* With with local and remote clusters
* Manages all types of resources and provides debugging and introspection.


```Shell
kubectl create
```
Creates a new Kubernetes resources. 
* From a file or cmd line
	* Usually YAML
	* Manifests

```Shell
kubectl delete
```
* Deletes a resource
	* Can do with a file
```Shell
kubectl get
```
* Returns all resources of a given type
```Shell
kubectl get pods
```

```Shell
kubectl describe
```
```Shell
kubectl describe pod server
```
* Describes information about the resource
```Shell
kubectl log
```
* Prints logs for containers. 
# Web Dashboard
* Easy to navigate views of clusters. 
* Optional