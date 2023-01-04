# Manage Resource Access
- Define role-based access controls
- Apply permissions to users
- Create and apply secrets to manage sensitive information
- Create service accounts and apply permissions using security context constraints

### Define role-based access controls
Default roles
-	**basic-user** Users with this role have read access to the project.
-	**cluster-admin** Users with this role have superuser access to the cluster resources. These users can perform any action on the cluster, and have full control of all projects.
-	**cluster-status** Users with this role can get cluster status information.
-	**self-provisioner** Users with this role can create new projects.

Default roles that can be added or removed from a project level:
-	**admin** Users with this role can manage all project resources, including granting access to other users to the project.
-	**edit** Users with this role can create, change, and delete common application resources from the project, such as services and deployment configurations. These users cannot act on management resources such as limit ranges and quotas, and cannot manage access permissions to the project.
-	**view** Users with this role can view project resources, but cannot modify project resources.

Creating local role:
```
# oc create role podview --verb=get --resource=pod -n blue

# oc adm policy add-role-to-user podview user2 --role-namespace=blue -n blue
```

Creating cluster role:
```
# oc create clusterrole podviewonly --verb=get --resource=pod

# oc adm policy add-role-to-group  podviewonly group1
```

### Apply permissions to users
```
oc adm policy add-cluster-role-to-user cluster-admin username
oc adm policy remove-cluster-role-from-user cluster-admin username
oc adm policy who-can delete user
oc adm policy add-role-to-user basic-user dev -n wordpress
```


### Create and apply secrets to manage sensitive information
Managing Sensitive Information with Secrets. 
> https://docs.openshift.com/container-platform/4.9/nodes/pods/nodes-pods-secrets.html
```diff
# oc create secret generic secret_name --from-literal key1=secret1 --from-literal key2=secret2

# oc secrets link --for mount serviceaccount/serviceaccount-name secret/secret_name


Add secret as a volume 
# oc set volume pod/example-pod --add --type=secret --secret-name=test-secret --mount-path=/etc/secret-volume
apiVersion: v1
kind: Pod
metadata:
  name: example-pod
spec:
  containers:
    - name: secret-test-container
      image: busybox
      command: [ "/bin/sh", "-c", "cat /etc/secret-volume/*" ]
      volumeMounts:
          # name must match the volume name below
          - name: secret-volume
            mountPath: /etc/secret-volume
            readOnly: true
  volumes:
    - name: secret-volume
      secret:
        secretName: test-secret
  restartPolicy: Never

Add secret as environment varible
# oc set env dc/demo --from=secret/demo-secret
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

### Create service accounts and apply permissions using security context constraints
Creating Service Account:
```diff
# oc create sa sample1
serviceaccount/sample1 created
# oc adm policy add-role-to-user view -z sample1
clusterrole.rbac.authorization.k8s.io/view added: "sample1"

```

Controlling Application Permissions with Security Context Constraints (SCCs) (anyuid, privileged etc)  
```
# oc adm policy add-scc-to-user anyuid -z default
```

