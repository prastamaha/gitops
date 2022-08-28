# Argocd Gitops

This is an example repository for managing argocd gitops multiple environments with helm.

## Directory Structure

```
.
├── charts
│   ├── applications
│   │   ├── Chart.yaml
│   │   ├── templates
│   │   │   ├── applications.yaml
│   │   │   └── projects.yaml
│   │   └── values.yaml
│   └── <your chart>
│       ├── Chart.yaml
│       └── <define your chart here>
├── values
│   ├── dev
│   │   ├── applications
│   │   │   └── values.yaml
│   │   └── <your chart>
│   │       └── values.yaml
│   ├── stg
│   │   ├── applications
│   │   │   └── values.yaml
│   │   └── <your chart>
│   │       └── values.yaml
│   └── prd
│       ├── applications
│       │   └── values.yaml
│       └── <your chart>
│           └── values.yaml
└── README.md
```

## Initial Setup

execute following command in each environment manually to setup applications chart

```
ARGOCD_NAMESPACE=$(kubectl get pods --selector app.kubernetes.io/instance=argocd -A | tail -1 | awk '{ print $1 }')
ARGOCD_ENV=dev
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
      - ../../values/$ARGOCD_ENV/applications/values.yaml
    path: charts/applications
    repoURL: https://github.com/prastamaha/gitops
    targetRevision: HEAD
EOF
```

## FAQ

<details>
<summary>How to add new chart?</summary>
<br>
If you have the chart already, just copy it into `charts/` directory. and create the values for each environments.

e.g:
```
$ cp -r /path/to/your_chart charts/
$ mkdir \
  values/dev/your_chart \
  values/stg/your_chart \
  values/prd/your_chart
$ touch \
  values/dev/your_chart/values.yaml \
  values/stg/your_chart/values.yaml \
  values/prd/your_chart/values.yaml
```
</details>

<br>
<details>
<summary>How to diff PR before merged</summary>
<br>
For example, we change values/echo-server/dev/values.yaml and create a pull request.

execute following command to compare the changes with the existing one.
```
argocd app diff echo-server --revision <PR_BRANCH>
```
</details>
