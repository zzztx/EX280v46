# ResourceQuota and LimitRange

## ResourceQuota
A resource quota, defined by a ResourceQuota object, provides constraints that limit aggregate resource consumption per project. It can limit the quantity of objects that can be created in a project by type, as well as the total amount of compute resources and storage that might be consumed by resources in that project.

```yaml
apiVersion: v1
kind: ResourceQuota
metadata:
  name: core-object-counts
  namespace: project1
spec:
  hard:
    configmaps: "10" 
    persistentvolumeclaims: "4" 
    replicationcontrollers: "20" 
    secrets: "10" 
    services: "10" 
    services.loadbalancers: "2" 
```

If the quota has a value specified for requests.cpu or requests.memory, then it requires that every incoming container make an explicit request for those resources. If the quota has a value specified for limits.cpu or limits.memory, then it requires that every incoming container specify an explicit limit for those resources.



#### Quota scopes
Each quota can have an associated set of scopes. A quota only measures usage for a resource if it matches the intersection of enumerated scopes.
- Terminating: Match pods where spec.activeDeadlineSeconds >= 0.
- NotTerminating: Match pods where spec.activeDeadlineSeconds is nil.
- BestEffort: Match pods that have best effort quality of service for either cpu or memory.
- NotBestEffort: Match pods that do not have best effort quality of service for cpu and memory.


```yaml
apiVersion: v1
kind: ResourceQuota
metadata:
  name: compute-resources
spec:
  hard:
    pods: "4" 
    requests.cpu: "1" 
    requests.memory: 1Gi 
    limits.cpu: "2" 
    limits.memory: 2Gi 
  scopes:
  - BestEffort
```

## LimitRange
A limit range, defined by a LimitRange object, restricts resource consumption in a project. In the project you can set specific resource limits for a pod, container, image, image stream, or persistent volume claim (PVC).

#### For Container, Pod, Image, ImageStream and PVC
```yaml
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
    - type: openshift.io/Image 
      max:
        storage: 1Gi
    - type: openshift.io/ImageStream 
      max:
        openshift.io/image-tags: 20
        openshift.io/images: 30
    - type: "PersistentVolumeClaim" 
      min:
        storage: "2Gi"
      max:
        storage: "50Gi"
```

View limits defined in project:
```
# oc get limits -n project1

# oc describe limits resource-limits -n project1
```
