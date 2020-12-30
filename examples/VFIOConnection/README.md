# Test VFIO connection

This example shows that NSC and NSE can work with each other over the VFIO connection.

## Run

Create test namespace

```bash
NAMESPACE=($(kubectl create -f namespace.yaml)[0])
NAMESPACE=${NAMESPACE:10}
```

Register namespace in `spire` server:

```bash
kubectl exec -n spire spire-server-0 -- \
/opt/spire/bin/spire-server entry create \
-spiffeID spiffe://example.org/ns/${NAMESPACE}/sa/default \
-parentID spiffe://example.org/ns/spire/sa/spire-agent \
-selector k8s:ns:${NAMESPACE} \
-selector k8s:sa:default
```

Create customization file
```bash
cat > kustomization.yaml <<EOF
---
apiVersion: kustomize.config.k8s.io/v1beta1
kind: Kustomization

namespace: ${NAMESPACE}

bases:
- ../../apps/vfio-nsc
- ../../apps/vfio-nse
EOF
```

Deploy NSC and NSE:

```bash
kubectl apply -k .
```

Wait for applications ready:
```bash
kubectl -n ${NAMESPACE} wait --for=condition=ready --timeout=1m pod -l app=nsc
```
```bash
kubectl -n ${NAMESPACE} wait --for=condition=ready --timeout=1m pod -l app=nse
```

Get NSC pod:
```bash
NSC_POD=$(kubectl -n ${NAMESPACE} get pods -l app=nsc |
  grep -v "NAME" |
  sed -E "s/([.]*) .*/\1/g")
```

Check connection result:
```bash
kubectl -n ${NAMESPACE} logs ${NSC_POD} sidecar |
  grep "All client init operations are done."
```

Test connection:
```bash
RX_PACKETS=$(kubectl -n ${NAMESPACE} exec -it ${NSC_POD} --container pinger -- /bin/bash /root/scripts/ping.sh |
  grep "rx .* pong packets" |
  sed -E 's/rx ([0-9]*) pong packets/\1/g')
```
```bash
test ${RX_PACKETS} -ne 0
```

## Cleanup

Delete ns
```bash
kubectl delete ns ${NAMESPACE}
```