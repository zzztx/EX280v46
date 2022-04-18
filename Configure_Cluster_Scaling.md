## Configure Cluster Scaling

 - Manually control the number of cluster workers
 - Automatically scale the number of cluster workers

The Machine API Operator provisions the following resources:
- MachineSet
- Machine
- Cluster Autoscaler
- Machine Autoscaler
- Machine Health Checks

Manually Scaling an OpenShift Cluster by adding or removing an instance of a machine in a machine set.
```
# oc scale --replicas=2 machineset MACHINE-SET -n openshift-machine-api
```

Automatically Scaling an OpenShift Cluster. Applying autoscaling to an OCP cluster involves deploying a cluster autoscaler and then deploying machine autoscalers for each machine type in your cluster.
```diff
# oc get clusterautoscaler
apiVersion: "autoscaling.openshift.io/v1"
kind: "ClusterAutoscaler"
metadata:
  name: "default"
spec:
  podPriorityThreshold: -10 
  resourceLimits:
    maxNodesTotal: 24 
    cores:
      min: 8 
      max: 128 
    memory:
      min: 4 
      max: 256 
    gpus:
      - type: nvidia.com/gpu 
        min: 0 
        max: 16 
      - type: amd.com/gpu 
        min: 0 
        max: 4 
  scaleDown: 
    enabled: true 
    delayAfterAdd: 10m 
    delayAfterDelete: 5m 
    delayAfterFailure: 30s 
    unneededTime: 5m
    
    
# oc get machineautoscaler -n openshift-machine-api
apiVersion: "autoscaling.openshift.io/v1beta1"
kind: "MachineAutoscaler"
metadata:
  name: "worker-us-east-1a" 
  namespace: "openshift-machine-api"
spec:
  minReplicas: 1 
  maxReplicas: 12 
  scaleTargetRef: 
    apiVersion: machine.openshift.io/v1beta1
    kind: MachineSet 
    name: worker-us-east-1a 
```
