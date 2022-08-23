# GitOps Repository

## Setup

execute following command manually to setup applications chart

```
ARGOCD_NAMESPACE=$(kubectl get pods --selector app.kubernetes.io/instance=argocd -A | tail -1 | awk '{ print $1 }')
```

```
cat <<EOF | kubectl apply -f -
apiVersion: argoproj.io/v1alpha1
kind: Application
metadata:
  name: applications
  namespace: $ARGOCD_NAMESPACE
spec:
  destination:
    namespace: $ARGOCD_NAMESPACE
    server: https://kubernetes.default.svc
  project: default
  source:
    helm:
      valueFiles:
      - ../../values/applications/values.yaml
    path: charts/applications
    repoURL: https://github.com/prastamaha/gitops
    targetRevision: HEAD
EOF
```
