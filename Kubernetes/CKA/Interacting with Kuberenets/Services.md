* Allow access outside of the container network.
* Allow other Pods to keep track if IP changes due to pod failure etc.

* *Define networking rules for accessing pods in the cluster from the internet*
* *Use Labels to select a group of Pods*
* *Service has a fixed IP address*
```YAML
apiVersion: v1
kind: Service
metadata:
  labels:
    app: webserver
  name: webserver
spec:
  ports:
  - port: 80 # Must also target ports
  selector: # Selector defines the label to match pods against 
    app: webserver # label to assign against
  type: NodePort # allocates a port for this service on each node, chosen from available ports unless you specify
```

```Shell
 kubectl get service
NAME         TYPE        CLUSTER-IP     EXTERNAL-IP   PORT(S)        AGE
kubernetes   ClusterIP   10.96.0.1      <none>        443/TCP        358d
webserver    NodePort    10.97.41.234   <none>        80:32587/TCP   12s
```
```Shell
 kubectl describe service webserver
Name:                     webserver
Namespace:                default
Labels:                   app=webserver
Annotations:              <none>
Selector:                 app=webserver
Type:                     NodePort
IP Family Policy:         SingleStack
IP Families:              IPv4
IP:                       10.97.41.234
IPs:                      10.97.41.234
Port:                     <unset>  80/TCP
TargetPort:               80/TCP
NodePort:                 <unset>  32587/TCP
Endpoints:                192.168.23.133:80
Session Affinity:         None
External Traffic Policy:  Cluster
Events:                   <none>
```
Node port are the pods you can access, need to grep for the node IP 
```Shell
kubectl describe nodes | grep -i address -A 1

Addresses:
  InternalIP:   10.0.0.11

 curl http://10.0.0.11:32587
```
Need to use the port Kubernetes is forwarding. 
