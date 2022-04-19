## Configure Networking Components ###
- Configure networking components
	- Troubleshoot software defined networking
	- Create and edit external routes
	- Control cluster network ingress
	- Create a self signed certificate
	- Secure routes using TLS certificates


Service types: ClusterIP, NodePort, LoadBalancer, ExternalService
API Resources: Ingress, Route

```
# oc get dnses.operator.openshift.io default -o yaml
apiVersion: operator.openshift.io/v1
kind: DNS
metadata:
  creationTimestamp: "2022-01-28T03:50:24Z"
  finalizers:
  - dns.operator.openshift.io/dns-controller
  generation: 1
  name: default
  resourceVersion: "22387871"
  uid: 65618011-6e8c-4037-a96d-07cfb108d0c1
spec:
  nodePlacement: {}

```
CoreDNS entries:
- A `svcname.namespace.svc.cluster.local`
- SRV `_port-name._port-protocol.svc.namespace.svc.cluster.local`

```diff
# oc get networks.operator.openshift.io cluster -o yaml
apiVersion: operator.openshift.io/v1
kind: Network
metadata:
  annotations:
    networkoperator.openshift.io/ovn-cluster-initiator: 192.168.206.10
  creationTimestamp: "2022-01-28T03:47:51Z"
  generation: 43
  name: cluster
  resourceVersion: "19305535"
  uid: d13afbae-6364-4856-ba41-38278dc0d151
spec:
  clusterNetwork:
  - cidr: 10.132.0.0/14
    hostPrefix: 23
  defaultNetwork:
    ovnKubernetesConfig:
      genevePort: 6081
      mtu: 1400
      policyAuditConfig:
        destination: "null"
        maxFileSize: 50
        rateLimit: 20
        syslogFacility: local0
    type: OVNKubernetes
  deployKubeProxy: false
  disableMultiNetwork: false
  disableNetworkDiagnostics: false
  logLevel: Normal
  managementState: Managed
  observedConfig: null
  operatorLogLevel: Normal
  serviceNetwork:
  - 172.30.0.0/16
  unsupportedConfigOverrides: null
  useMultiNetworkPolicy: false

```

Ingress rule:
```
oc get ingress
```

Certificate generation:
```diff
openssl genrsa -out file.key
openssl req -new -subj <subject> -out file.req -key file.key
openssl x509 -req -in file.req -out file.crt -signkey file.key
```

Secure route (edge/passthru):
```
oc create route edge \
> --service <svcname> --hostname <host>.apps.acme.com \
> --key file.key --cert file.crt
```
