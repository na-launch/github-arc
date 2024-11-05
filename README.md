# github-arc
GitHub Action Runner Controller (ARC) for OpenShift

```
NAMESPACE="github-arc-systems"
INSTALLATION_NAME="github-arc-system"
helm install "${INSTALLATION_NAME}"\
    --namespace "${NAMESPACE}" \
    --create-namespace \
    oci://ghcr.io/actions/actions-runner-controller-charts/gha-runner-scale-set-controller
```

```
NAMESPACE="github-arc-runners"
INSTALLATION_NAME="github-arc-runner-set"
GITHUB_CONFIG_URL="https://github.com/<your_enterprise/org/repo>"
GITHUB_PAT="<PAT>"
helm install "${INSTALLATION_NAME}" \
    --namespace "${NAMESPACE}" \
    --create-namespace \
    --set githubConfigUrl="${GITHUB_CONFIG_URL}" \
    --set githubConfigSecret.github_token="${GITHUB_PAT}" \
    --set controllerServiceAccount.name="${INSTALLATION_NAME}-gha-rs-no-permission" \
    --set controllerServiceAccount.namespace="${NAMESPACE}" \
    -f values.yaml \
    oci://ghcr.io/actions/actions-runner-controller-charts/gha-runner-scale-set
```

```
oc apply -f manifests/scc.yaml
oc apply -f manifests/cluster-role-binding.yaml
```
