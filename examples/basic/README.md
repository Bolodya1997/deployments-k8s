## Requires

- [spire](../spire)

## Includes

- [Local Connection](../LocalConnection)
- [Remote Connection](../RemoteConnection)

## Run

Create ns for deployments:
```bash
kubectl create ns nsm-system
```

Register `nsm-system` namespace in spire:

```bash
kubectl exec -n spire spire-server-0 -- \
/opt/spire/bin/spire-server entry create \
-spiffeID spiffe://example.org/ns/nsm-system/sa/default \
-parentID spiffe://example.org/ns/spire/sa/spire-agent \
-selector k8s:ns:nsm-system \
-selector k8s:sa:default
```
Apply NSM resources for basic tests:

```bash
kubectl apply -k .
```

## Cleanup

```bash
kubectl delete ns nsm-system
```