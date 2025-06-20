* Production grade container orchestrator designed to automate, scale operating containerised applications.
* Made by google
* Distrusted system 
	* On cloud
	* On Prem
* Can move containers as machines are added or removed

# Alternatives
## DC/OS
* Disturbed cloud operating systems
* Pools compute resources into a uniform task pool
* Supports many different types of workloads. 
* Includes package manager
* Can run Kubernetes on it
## Amazon ECS
* Eleatic Container Service
* Create pools of compute resources
* API calls to orchestrate containers. 
* On available on AWS
## Docker Swarm Mode
* Official Docker solution for orchestrating contains across a cluster of machines. Builds a cluster from multiple docker hosts
* Works natively with the docker command
* Docker provides full support of Kubernetes. 
# Deploying Kubernetes
## Single Node Clusters
* Can be run on single machines for testing. 
	* Docker Desktop for mac and windows 
	* Minikube
	* Kubeadm
* Useful with CI
	* Ephemeral clusters that start quickly and are in a pristine state for testing. 
## Multi Mode
* Horizontal scaling
* Tolerate failures
* How much control vs effort.

* Fully managed solutions free you from routine maintenance
	* Often lag from the latest releases


| Fully Managed      | Full Control |
| ------------------ | ------------ |
| Amazon EKS         | Kubespray    |
| AKS (Azure)        | Kops         |
| GKE (google cloud) | kubeadm      |
* *Do you already have expertise with a particulate cloud provider?*
* *Do you need enterprise support?*
* *Are you concerned about vendor lock-in?*
* *On Prem, in the cloud, or both?*
	* Kubernetes provide an abstraction of a cluster of resources and the underlying nodes can be run in different platforms. 
* *Linux, Windows or both?*
	* Need to ensure you have windows/Linux nodes in your cluster.
