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

## Create and edit external routes

Certificate generation:
```diff
Self-signed certs:
# openssl genrsa -out tls.key
# openssl req -new -out tls.csr -key tls.key
# openssl x509 -req -in tls.csr -out tls.crt -signkey tls.key

Signed by CA:
# openssl x509 -req -days 365 -in tls.csr -CA ca.crt -CAkey ca.key -set_serial 01 -out tls.crt
```

#### Reencrypt Route
The tls.key, tls.crt and ca.crt are used to create TLS connection with client. The destca.crt is used to trust service's certificate.

```yaml
# oc create route reencrypt --service=frontend --cert=tls.crt --key=tls.key --dest-ca-cert=destca.crt --ca-cert=ca.crt --hostname=www.example.com

or
# cat <<EOF | oc create -f -
apiVersion: route.openshift.io/v1
kind: Route
metadata:
  name: frontend
spec:
  host: www.example.com
  to:
    kind: Service
    name: frontend
  tls:
    termination: reencrypt
    key: |-
      -----BEGIN PRIVATE KEY-----
      [...]
      -----END PRIVATE KEY-----
    certificate: |-
      -----BEGIN CERTIFICATE-----
      [...]
      -----END CERTIFICATE-----
    caCertificate: |-
      -----BEGIN CERTIFICATE-----
      [...]
      -----END CERTIFICATE-----
    destinationCACertificate: |-
      -----BEGIN CERTIFICATE-----
      [...]
      -----END CERTIFICATE-----
EOF
```

#### Edge Route
```yaml
# oc create route edge --service=frontend --cert=tls.crt --key=tls.key --ca-cert=ca.crt --hostname=www.example.com

# cat <<EOF | oc create -f -
apiVersion: route.openshift.io/v1
kind: Route
metadata:
  name: frontend
spec:
  host: www.example.com
  to:
    kind: Service
    name: frontend
  tls:
    termination: edge
    key: |-
      -----BEGIN PRIVATE KEY-----
      [...]
      -----END PRIVATE KEY-----
    certificate: |-
      -----BEGIN CERTIFICATE-----
      [...]
      -----END CERTIFICATE-----
    caCertificate: |-
      -----BEGIN CERTIFICATE-----
      [...]
      -----END CERTIFICATE-----
EOF
```

#### Passthrough Route
```yaml
# oc create route passthrough route-passthrough-secured --service=frontend --port=8080

# cat <<EOF | oc create -f -
apiVersion: route.openshift.io/v1
kind: Route
metadata:
  name: route-passthrough-secured 
spec:
  host: www.example.com
  port:
    targetPort: 8080
  tls:
    termination: passthrough 
    insecureEdgeTerminationPolicy: None 
  to:
    kind: Service
    name: frontend
 EOF
 ```
