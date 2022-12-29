# Configure Pod Scheduling

This topic has the most weight over all the topics in this exam.



Limiting resource usage (factors that can affect the resources that a pod is allowed use or run)  
```
oc adm top nodes -l node-role.kubernetes.io/worker
oc set resources deployment hello-world-nginx \
> --requests cpu=10m,memory=20Mi --limits cpu=80m,memory=100Mi
oc create quota dev-quota --hard services=10,cpu=1300,memory=1.5Gi -n <ns>
oc get resourcequota -n <ns>
oc describe limitrange dev-limits
```

## Taint and Toleration
A taint allows a node to refuse a pod to be scheduled unless that pod has a matching toleration. You apply taints to a node through the Node specification (NodeSpec) and apply tolerations to a pod through the Pod specification (PodSpec). When you apply a taint a node, the scheduler cannot place a pod on that node unless the pod can tolerate the taint.
- **NoSchedule**: New pods that do not match the taint are not scheduled onto that node. Existing pods on the node remain.
- **PreferNoSchedule**: New pods that do not match the taint might be scheduled onto that node, but the scheduler tries not to. Existing pods on the node remain.
- **NoExecute**: New pods that do not match the taint cannot be scheduled onto that node. Existing pods on the node that do not have a matching toleration are removed.

```diff
Add taint to node:
# oc adm taint node hub-node2 app=bookinfo:NoExecute
node/hub-node2 tainted
# oc get node hub-node2 -o yaml
......
spec:
  taints:
  - effect: NoExecute
    key: app
    value: bookinfo
    
Remove taint from node:
# oc adm taint node hub-node2 app-
node/hub-node2 untainted

Add/remove tolertation to pod in spec:
    spec:
      tolerations:
      - key: "app"
        operator: "Equal"
        value: "bookinfo"
        effect: "NoExecute"
        tolerationSeconds: 3600
```
> https://docs.openshift.com/container-platform/4.10/nodes/scheduling/nodes-scheduler-taints-tolerations.html


## Node Selector
A node selector specifies a map of key/value pairs that are defined using custom labels on nodes and selectors specified in pods.
For the pod to be eligible to run on a node, the pod must have the same key/value node selector as the label on the node.
```yaml
# Label node
apiVersion: v1
kind: Node
metadata:
  name: hub-node1
  labels:
    type: user-node
    region: east

# Add nodeSelector in pod spec:
spec:
  nodeSelector: 
    region: east
    type: user-node
    
# Add nodeSelector in Namespace annotation:
apiVersion: v1
kind: Namespace
metadata:
  name: east-region
  annotations:
    openshift.io/node-selector: "region=east"
```

```diff
Label node:
# oc label node node1.us-west-1.compute.internal env[-|=dev] [--overwrite]
# oc get node node1.us-west-1.compute.internal --show-labels
# oc get node -L failure-domain.beta.kubernetes.io/region

Add node-selector to existing deployment:
# oc patch deployment/myapp --patch \
> '{"spec":{"template":{"spec":{"nodeSelector":{"env":"dev"}}}}}'

Create new project with node-selector: 
# oc adm new-project demo --node-selector "tier=1"

Add node-selector to existing project:
# oc annotate namespace demo \
> openshift.io/node-selector="tier=2" --overwrite
```

## LimitRange
A violation of LimitRange constraints prevents pod creation, and resulting error messages are displayed. A limit range, defined by a LimitRange object, restricts resource consumption in a project. In the project you can set specific resource limits for a pod, container, image, image stream, or persistent volume claim (PVC).

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

> https://docs.openshift.com/container-platform/4.6/nodes/clusters/nodes-cluster-limit-ranges.html


## ResourceQuota
A violation of ResourceQuota constraints prevents a pod from being scheduled to any node. The pod might be created but remain in the pending state. A resource quota, defined by a ResourceQuota object, provides constraints that limit aggregate resource consumption per project. It can limit the quantity of objects that can be created in a project by type, as well as the total amount of compute resources and storage that might be consumed by resources in that project.

#### ResourceQuota in single project
```yaml
$ oc get quota -n bookinfo  compute-resources -o yaml
apiVersion: v1
kind: ResourceQuota
metadata:
  creationTimestamp: "2022-10-24T22:31:10Z"
  name: compute-resources
  namespace: bookinfo
  resourceVersion: "267099800"
  uid: d71de0e3-7efe-48af-9834-08ae429a1651
spec:
  hard:
    limits.cpu: "4"
    limits.memory: 2Gi
    pods: "10"
    requests.cpu: "2"
    requests.memory: 1Gi
status:
  hard:
    limits.cpu: "4"
    limits.memory: 2Gi
    pods: "10"
    requests.cpu: "2"
    requests.memory: 1Gi
  used:
    limits.cpu: 2100m
    limits.memory: 1400Mi
    pods: "7"
    requests.cpu: 1400m
    requests.memory: 700Mi
 
    
# oc create quota test \
    --hard=count/deployments.extensions=2,count/replicasets.extensions=4,count/daemonsets.apps=1, \
      count/deployments.apps=1,count/pods=3,count/secrets=4
```

If the quota has a value specified for requests.cpu or requests.memory, then it requires that every incoming container make an explicit request for those resources. If the quota has a value specified for limits.cpu or limits.memory, then it requires that every incoming container specify an explicit limit for those resources.

#### ResourceQuota in multiple projects
```diff
# oc create clusterquota user-qa \
> --project-annotation-selector openshift.io/requester=qa \
> --hard pods=12,secrets=20

# oc create clusterquota env-qa \
> --project-label-selector environment=qa \
> --hard pods=10,services=5

# oc get quota -o wide
```

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

> https://docs.openshift.com/container-platform/4.6/applications/quotas/quotas-setting-per-project.html


## Scaling an Application
Using a horizontal pod autoscaler (HPA) to specify how OpenShift Container Platform should automatically increase or decrease the scale of a replication controller or deployment configuration, based on metrics collected from the pods that belong to that replication controller or deployment configuration.

```diff
> Scale application manually
# oc scale --replicas=2 deployment/details-v1

# oc get all
NAME                                  READY   STATUS    RESTARTS   AGE
pod/details-v1-54c75c996d-4gd4k       1/1     Running   0          15m
pod/details-v1-54c75c996d-f7cgd       1/1     Running   0          8s

NAME                             READY   UP-TO-DATE   AVAILABLE   AGE
deployment.apps/details-v1       2/2     2            2           15m

NAME                                        DESIRED   CURRENT   READY   AGE
replicaset.apps/details-v1-54c75c996d       2         2         2       15m

> Scale application automatically
# oc autoscale deployment/kibana --min=2 --max=7 --cpu-percent=75

apiVersion: autoscaling/v1
kind: HorizontalPodAutoscaler
metadata:
  name: kibana
  namespace: openshift-logging
spec:
  maxReplicas: 7
  minReplicas: 2
  scaleTargetRef:
    apiVersion: apps/v1
    kind: Deployment
    name: kibana
  targetCPUUtilizationPercentage: 75
status:
  currentReplicas: 5
  desiredReplicas: 0
  
# oc autoscale deployment/example --min=1 --max=10 ???

apiVersion: autoscaling/v2 
kind: HorizontalPodAutoscaler
metadata:
  name: hpa-resource-metrics-memory 
  namespace: default
spec:
  scaleTargetRef:
    apiVersion: apps/v1 
    kind: Deployment 
    name: example 
  minReplicas: 1 
  maxReplicas: 10 
  metrics: 
  - type: Resource
    resource:
      name: memory 
      target:
        type: AverageValue 
        averageValue: 500Mi 
  behavior: 
    scaleDown:
      stabilizationWindowSeconds: 300
      policies:
      - type: Pods
        value: 4
        periodSeconds: 60
      - type: Percent
        value: 10
        periodSeconds: 60
      selectPolicy: Max

```

> https://docs.openshift.com/container-platform/4.6/nodes/pods/nodes-pods-autoscaling.html
> https://docs.openshift.com/container-platform/4.10/nodes/pods/nodes-pods-autoscaling-custom.html
