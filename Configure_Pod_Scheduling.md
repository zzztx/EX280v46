## Configure Pod Scheduling ##

This topic has the most weight over all the topics in this exam.

Controlling pod scheduling behavior (factors that can affect on which nodes a pod can or cannot be run)  
```diff
oc label node node1.us-west-1.compute.internal env[-|=dev] [--overwrite]
oc get node node2.us-west-1.compute.internal --show-labels
oc get node -L failure-domain.beta.kubernetes.io/region
oc patch deployment/myapp --patch \
> '{"spec":{"template":{"spec":{"nodeSelector":{"env":"dev"}}}}}'
oc adm new-project demo --node-selector "tier=1"
oc annotate namespace demo \
> openshift.io/node-selector="tier=2" --overwrite
```

Limiting resource usage (factors that can affect the resources that a pod is allowed use or run)  
```
oc adm top nodes -l node-role.kubernetes.io/worker
oc set resources deployment hello-world-nginx \
> --requests cpu=10m,memory=20Mi --limits cpu=80m,memory=100Mi
oc create quota dev-quota --hard services=10,cpu=1300,memory=1.5Gi -n <ns>
oc get resourcequota -n <ns>
oc describe limitrange dev-limits
```

### LimitRange ###
A violation of LimitRange constraints prevents pod creation, and resulting error messages are displayed. A limit range, defined by a LimitRange object, restricts resource consumption in a project. In the project you can set specific resource limits for a pod, container, image, image stream, or persistent volume claim (PVC).
https://docs.openshift.com/container-platform/4.6/nodes/clusters/nodes-cluster-limit-ranges.html
```diff
> Container limits
apiVersion: "v1"
kind: "LimitRange"
metadata:
  name: "resource-limits"
spec:
  limits:
    - type: "Container"
      max:
        cpu: "2"
        memory: "1Gi"
      min:
        cpu: "100m"
        memory: "4Mi"
      default:
        cpu: "300m"
        memory: "200Mi"
      defaultRequest:
        cpu: "200m"
        memory: "100Mi"
      maxLimitRequestRatio:
        cpu: "10"

> Pod limits
apiVersion: "v1"
kind: "LimitRange"
metadata:
  name: "resource-limits" 
spec:
  limits:
    - type: "Pod"
      max:
        cpu: "2" 
        memory: "1Gi" 
      min:
        cpu: "200m" 
        memory: "6Mi" 
      maxLimitRequestRatio:
        cpu: "10" 

```

### ResourceQuota ###
A violation of ResourceQuota constraints prevents a pod from being scheduled to any node. The pod might be created but remain in the pending state. A resource quota, defined by a ResourceQuota object, provides constraints that limit aggregate resource consumption per project. It can limit the quantity of objects that can be created in a project by type, as well as the total amount of compute resources and storage that might be consumed by resources in that project.
https://docs.openshift.com/container-platform/4.6/applications/quotas/quotas-setting-per-project.html

```
> ResourceQuota in multiple projects

oc create clusterquota user-qa \
> --project-annotation-selector openshift.io/requester=qa \
> --hard pods=12,secrets=20

oc create clusterquota env-qa \
> --project-label-selector environment=qa \
> --hard pods=10,services=5

> ResourceQuota in one peoject
oc create quota test \
    --hard=count/deployments.extensions=2,count/replicasets.extensions=4,count/daemonsets.apps=1, \
      count/deployments.apps=1,count/pods=3,count/secrets=4


apiVersion: v1
kind: ResourceQuota
metadata:
  name: core-object-counts
spec:
  hard:
    configmaps: "10" 
    persistentvolumeclaims: "4" 
    replicationcontrollers: "20" 
    secrets: "10" 
    services: "10" 
    services.loadbalancers: "2" 

    pods: "4" 
    requests.cpu: "1" 
    requests.memory: 1Gi 
    requests.ephemeral-storage: 2Gi 
    limits.cpu: "2" 
    limits.memory: 2Gi 
    limits.ephemeral-storage: 4Gi

  scopes:
  - BestEffort  // NotTerminating or Terminating
```

Scaling an Application  
```
oc scale --replicas 3 deployment/myapp
oc autoscale dc/hello --min 1 --max 10 --cpu-percent 80
oc get hpa
```
