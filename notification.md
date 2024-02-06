cat <<EOF | kubectl apply -f -
apiVersion: notification.toolkit.fluxcd.io/v1beta1
kind: Alert
metadata:
  name: gitops-connector
  namespace: flux-system
spec:
  eventSeverity: info
  eventSources:
  - kind: GitRepository
    name: cluster-config
  - kind: Kustomization
    name: cluster-config-cluster-config
  providerRef:
    name: gitops-connector
---
apiVersion: notification.toolkit.fluxcd.io/v1beta1
kind: Provider
metadata:
  name: gitops-connector
  namespace: flux-system
spec:
  type: generic
  address: http://gitops-connector:8080/gitopsphase
EOF