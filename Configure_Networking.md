## Configure Networking Components ###
- Configure networking components
	- Troubleshoot software defined networking
	- Create and edit external routes
	- Control cluster network ingress
	- Create a self signed certificate
	- Secure routes using TLS certificates


Service types: ClusterIP, NodePort, LoadBalancer, ExternalService
```
oc describe dns.operator/default
```
CoreDNS entries:
•	A `svcname.namespace.svc.cluster.local`
•	SRV `_port-name._port-protocol.svc.namespace.svc.cluster.local`

```
oc get Network.config.openshift.io cluster -o yaml
```

Ingress rule:
```
oc get ingress
```

Certificate generation:
```
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
