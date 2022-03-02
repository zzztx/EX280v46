## Manage Users and Policies ##


Working with htpasswd

- Create: `htpasswd -c -B -b /tmp/htpasswd student redhat123`
- Update: `htpasswd -b /tmp/htpasswd student redhat1234`

Create a secret:
```
oc create secret generic htpasswd-secret \
> --from-file htpasswd=/tmp/htpasswd -n openshift-config
```

Adding to OAuth:
```
apiVersion: config.openshift.io/v1
kind: OAuth
metadata:
  name: cluster
spec:
  identityProviders:
  - name: my_htpasswd_provider
    mappingMethod: claim
    type: HTPasswd
    htpasswd:
      fileData:
        name: htpasswd-secret
```

Getting users and identities
```
oc get users
oc get identity
```

Removing the default kubeadmin:
```
oc delete secret kubeadmin -n kube-system
```
