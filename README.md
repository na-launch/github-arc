# github-arc
GitHub Action Runner Controller (ARC) for OpenShift

Set values for reusability between helm charts

Note: 
`GITHUB_ARC_RUNNER_INSTALLATION_NAME` is the value that is surfaced in GitHub Actions UI and used within actions to trigger jobs on those specific runners.
```
export GITHUB_ARC_SYSTEM_NAMESPACE="github-arc-system"
export GITHUB_ARC_SYSTEM_INSTALLATION_NAME="github-arc-system"
export GITHUB_ARC_RUNNER_NAMESPACE="github-arc-runners"
export GITHUB_ARC_RUNNER_INSTALLATION_NAME="github-arc-runners"
```

Install GitHub ARC system controller
```
helm install ${GITHUB_ARC_SYSTEM_INSTALLATION_NAME} \
    --namespace "${GITHUB_ARC_SYSTEM_NAMESPACE}" \
    --create-namespace \
    --set serviceAccount.name="${GITHUB_ARC_SYSTEM_INSTALLATION_NAME}-gha-rs-controller" \
    oci://ghcr.io/actions/actions-runner-controller-charts/gha-runner-scale-set-controller
```

Install GitHub ARC runner set
```
GITHUB_CONFIG_URL="https://github.com/<your_enterprise/org/repo>"
GITHUB_PAT="<PAT>"
helm install "${GITHUB_ARC_RUNNER_INSTALLATION_NAME}" \
    --namespace "${GITHUB_ARC_RUNNER_NAMESPACE}" \
    --create-namespace \
    --set githubConfigUrl="${GITHUB_CONFIG_URL}" \
    --set githubConfigSecret.github_token="${GITHUB_PAT}" \
    --set controllerServiceAccount.name="${GITHUB_ARC_SYSTEM_INSTALLATION_NAME}-gha-rs-controller" \
    --set controllerServiceAccount.namespace="${GITHUB_ARC_SYSTEM_NAMESPACE}" \
    -f values.yaml \
    oci://ghcr.io/actions/actions-runner-controller-charts/gha-runner-scale-set
```

Create custom SecurityContextConstraint and ClusterRole required for executing runners on OpenShift. Bind this role to the RunnerSet ServiceAccount
```
oc apply -f manifests/scc.yaml
oc apply -f manifests/cluster-role.yaml
oc policy add-role-to-user system:openshift:scc:github-arc -z ${GITHUB_ARC_RUNNER_INSTALLATION_NAME}-gha-rs-no-permission -n ${GITHUB_ARC_RUNNER_NAMESPACE}
```

TODO: Update documentation for configuration of multiple runner groups
