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

## Control cluster network ingress
Using External IP:
```yaml
apiVersion: v1
kind: Service
metadata:
  name: http-service
spec:
  clusterIP: 172.30.163.110
  externalIPs:
  - 192.168.132.253
  externalTrafficPolicy: Cluster
  ports:
  - name: highport
    nodePort: 31903
    port: 30102
    protocol: TCP
    targetPort: 30102
  selector:
    app: web
  sessionAffinity: None
  type: LoadBalancer
status:
  loadBalancer:
    ingress:
    - ip: 192.168.132.253
```

Using Load Balancer:
```yaml
apiVersion: v1
kind: Service
metadata:
  name: egress-2 
spec:
  ports:
  - name: db
    port: 3306 
  loadBalancerIP:
  loadBalancerSourceRanges: 
  - 10.0.0.0/8
  - 192.168.0.0/16
  type: LoadBalancer 
  selector:
    name: mysql 
```

Using NodePort:
```yaml
apiVersion: v1
kind: Service
metadata:
  name: details-ex-nodeport
  namespace: bookinfo
spec:
  clusterIP: 172.30.108.2
  clusterIPs:
  - 172.30.108.2
  externalTrafficPolicy: Cluster
  internalTrafficPolicy: Cluster
  ipFamilies:
  - IPv4
  ipFamilyPolicy: SingleStack
  ports:
  - nodePort: 32359
    port: 9080
    protocol: TCP
    targetPort: 9080
  selector:
    app: details
  sessionAffinity: None
  type: NodePort
```


## Secure routes using TLS certificates

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
