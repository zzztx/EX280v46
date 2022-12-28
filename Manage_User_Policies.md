# Manage Users and Identities
- Configure the HTPasswd identity provider for authentication
- Create and delete users
- Modify user passwords
- Modify user and group permissions
- Create and manage groups

Working with htpasswd

- Create: `htpasswd -c -B -b /tmp/htpasswd user1 redhat123`
- Add: `htpasswd -B -b /tmp/htpasswd user2 redhat123`
- Update: `htpasswd -b /tmp/htpasswd user1 redhat1234`

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
```diff
# oc get users
NAME    UID                                    FULL NAME   IDENTITIES
user1   50573a23-126e-456f-92e9-3abd78c4496f               htpasswd_provider:user1
user2   a6f12ce5-bec8-4dfa-b303-5234e48576b0               htpasswd_provider:user2
user3   f49e19fa-7e3b-4ec5-8763-f53e025606e8               htpasswd_provider:user3

# oc get identity
NAME                      IDP NAME            IDP USER NAME   USER NAME   USER UID
htpasswd_provider:user1   htpasswd_provider   user1           user1       50573a23-126e-456f-92e9-3abd78c4496f
htpasswd_provider:user2   htpasswd_provider   user2           user2       a6f12ce5-bec8-4dfa-b303-5234e48576b0
htpasswd_provider:user3   htpasswd_provider   user3           user3       f49e19fa-7e3b-4ec5-8763-f53e025606e8

```

Removing the default kubeadmin:
```diff
# oc adm policy add-cluster-role-to-user cluster-admin user1
clusterrole.rbac.authorization.k8s.io/cluster-admin added: "user1"

# oc login -u user1 -p redhat123 https://api.cnfdf06.ran.dfwt5g.lab:6443
Login successful.

# oc delete secret kubeadmin -n kube-system
```

Create groups:
```diff
# oc adm groups new group1 user1
group.user.openshift.io/group1 created
# oc adm groups new group2 user2 user3
group.user.openshift.io/group2 created

# oc get groups group2 -o yaml
apiVersion: user.openshift.io/v1
kind: Group
metadata:
  creationTimestamp: "2022-03-02T23:30:03Z"
  name: group2
  resourceVersion: "13551633"
  uid: 9d8cfbdf-cf2d-4104-a95e-3f75e599829a
users:
- user2
- user3

```

Add permission to groups:
```diff
# oc adm policy add-role-to-group view group2
clusterrole.rbac.authorization.k8s.io/view added: "group2"

```
