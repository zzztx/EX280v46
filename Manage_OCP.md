## Manage OpenShift Container Platform ##
- Manage OpenShift Container Platform
	- Use the command-line interface to manage and configure an OpenShift cluster
	- Use the web console to manage and configure an OpenShift cluster
	- Create and delete projects
	- Import, export, and configure Kubernetes resources
	- Examine resources and cluster status
	- View logs
	- Monitor cluster events and alerts
	- Troubleshoot common cluster events and alerts
	- Use product documentation


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

### Secret ###
Creating secret:
```yaml

apiVersion: v1
kind: Secret
metadata:
  name: test-secret
type: Opaque 
data: 
  username: dXNlcjEK
  password: cGFzc3dvcmQK
stringData: 
  hostname: myapp.mydomain.com
  secret.properties: |
    property1=valueA
    property2=valueB

```

Consuming secret in Pod:
```yaml
# Using as file
apiVersion: v1
kind: Pod
metadata:
  name: secret-example-pod
spec:
  containers:
    - name: secret-test-container
      image: busybox
      command: [ "/bin/sh", "-c", "cat /etc/secret-volume/*" ]
      volumeMounts: 
          - name: secret-volume
            mountPath: /etc/secret-volume 
            readOnly: true 
  volumes:
    - name: secret-volume
      secret:
        secretName: test-secret 
  restartPolicy: Never
  
# Using as environment variable  
apiVersion: v1
kind: Pod
metadata:
  name: secret-example-pod
spec:
  containers:
    - name: secret-test-container
      image: busybox
      command: [ "/bin/sh", "-c", "export" ]
      env:
        - name: TEST_SECRET_USERNAME_ENV_VAR
          valueFrom:
            secretKeyRef: 
              name: test-secret
              key: username
  restartPolicy: Never

```

Using secret in Service Account:
```yaml
apiVersion: v1
kind: ServiceAccount
 ...
secrets:
- name: test-secret
```





