* Rolling updates by default
* Updates in group rather than all at once
* Both old and new version running for some time
* Alternate is the recreate strategy
* *Scaling is not a rollout*

```YAML
 strategy:
    rollingUpdate:
      maxSurge: 25%
      maxUnavailable: 25%
    type: RollingUpdate
```
Number of pods to change at once. 

```Shell
kubectl edit -n deployments deployments app-tier
```
Make a change to the yaml

```
kubectl rollout -n deployments status deployment app-tier
```
* Pause a rollout
```
kubectrl rollout -n deployments pause deployment app-ter
```
* Will finish current but not new ones
```
kubectrl rollout -n deployments resume deployment app-ter
```
* Undo a rollout
```
 kubectl rollout -n deployments undo deployment app-tier
```
* history 
```
 kubectl rollout history  -n deployments deployment app-tier
```