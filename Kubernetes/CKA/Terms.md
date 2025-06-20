* **Clusters**
	* Refers to all the machines collectively
	* Can be thought of the as the whole system
* **Nodes**
	* The machines in the cluster
	* Categorised into 2 types:
	* **Worker Nodes**
		* Include software to run containers managed by the Kubernetes control plane.
		* Worker Nodes include software to run containers managed by the Kubernetes control plane.
	* **Master Nodes**
		* Run the **control plane**
			* The control plane is a set of APIS and software that Kubernetes interacts with. 
			* The APIs are referred to as *master components*
* **Scheduling**
	* Control plane schedules containers onto nodes
	* Scheduling decisions consider required CPU and other factors. 
	* The decision process of placing containers onto the node.
* **Pods**
	* Pods may contain 1 or more containers
	* Smallest building blocks in Kubernetes
	* More complex and useful abstractions are built on top of pods. 
* **Services**
	* Define networking rules for exposing a group of pods
		* To other pods
		* The the internet. 
* **Deployments**
	* Manage the deploying configuration of changes to running pods
	* Horizontal scaling. 