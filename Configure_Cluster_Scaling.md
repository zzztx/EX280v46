## Configure Cluster Scaling

Manually Scaling an OpenShift Cluster  
```
oc scale --replicas=2 \
> machineset MACHINE-SET -n openshift-machine-api
```

Automatically Scaling an OpenShift Cluster
```
oc get clusterautoscaler
oc get machineautoscaler -n openshift-machine-api
```
