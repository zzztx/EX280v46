## Manage OpenShift Container Platform ##

Executing troubleshooting commands:

- Getting node informations:
```
oc get node
oc describe node <nodename>
```

- Getting busiest nodes
```
oc adm top nodes
```

- Getting `journalctl` logs from a node
```
oc adm node-logs -u kubelet my-node-name
```

- Running a remote shell for a node
```
oc debug node/<nodename>
```

- Work with cluster installers
```
oc get clusteroperators
oc get clusterversion -o yaml
```

- Getting a pod logs
```
oc logs <pod> [-c <container>] [-f]
```

- Debugging a deployment or a pod
```
oc debug deployment/<deplname> [--as-root]
oc rsh <podname>
oc port-forward <pod> <localport>:<remoteport>
```

- Getting a file from a pod
```
oc cp file <pod>:/file
oc cp <pod>:/file file
```

### Taint and Toleration ###
A taint allows a node to refuse a pod to be scheduled unless that pod has a matching toleration. You apply taints to a node through the Node specification (NodeSpec) and apply tolerations to a pod through the Pod specification (PodSpec). When you apply a taint a node, the scheduler cannot place a pod on that node unless the pod can tolerate the taint.
- **NoSchedule**: New pods that do not match the taint are not scheduled onto that node. Existing pods on the node remain.
- **PreferNoSchedule**: New pods that do not match the taint might be scheduled onto that node, but the scheduler tries not to. Existing pods on the node remain.
- **NoExecute**: New pods that do not match the taint cannot be scheduled onto that node. Existing pods on the node that do not have a matching toleration are removed.

```diff
Add taint to node:
# oc adm taint node cnfdf06.ran.dfwt5g.lab key1=value1:NoSchedule
node/cnfdf06.ran.dfwt5g.lab tainted

Remove taint from node:
# oc adm taint node cnfdf06.ran.dfwt5g.lab key1-
node/cnfdf06.ran.dfwt5g.lab untainted

Add/remove tolertation to pod in spec:
    spec:
      tolerations:
      - key: "key1"
        operator: "Equal"
        value: "value1"
        effect: "NoExecute"
        tolerationSeconds: 3600
```

### Node Selector ##
A node selector specifies a map of key/value pairs that are defined using custom labels on nodes and selectors specified in pods.
For the pod to be eligible to run on a node, the pod must have the same key/value node selector as the label on the node.
```diff

Add nodeSelector in pod spec:
spec:
  nodeSelector: 
    region: east
    type: user-node
    
Add nodeSelector in Namespace annotation:
apiVersion: v1
kind: Namespace
metadata:
  name: east-region
  annotations:
    openshift.io/node-selector: "region=east"

```


