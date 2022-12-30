# Manage OpenShift Container Platform
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

> https://docs.openshift.com/container-platform/4.10/nodes/index.html

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



## Secret
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
> https://docs.openshift.com/container-platform/4.10/nodes/pods/nodes-pods-secrets.html




