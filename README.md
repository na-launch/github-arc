# GitHub Action Runner Controller (ARC) for OpenShift

Deploy GitHub ARC on OpenShift

This repo contains the steps necessary to deploy GitHub ARC on OpenShift. Once deployed additional runners will appear within the Actions UI as "Self-hosted runners" which correspond to the names used in the steps below (github-arc-runners, github-arc-runners-additional). These names are examples and can be modified to fit your needs.

## Installation

### Set values for reusability between helm charts

Note:
`GITHUB_ARC_RUNNER_INSTALLATION_NAME` is the value that is surfaced in GitHub Actions UI and used within actions to trigger jobs on those specific runners.

```bash
export GITHUB_ARC_SYSTEM_NAMESPACE="github-arc-system"
export GITHUB_ARC_SYSTEM_INSTALLATION_NAME="github-arc-system"
export GITHUB_ARC_RUNNER_NAMESPACE="github-arc-runners"
export GITHUB_ARC_RUNNER_INSTALLATION_NAME="github-arc-runners"
```

### Install GitHub ARC system controller

```bash
helm update ${GITHUB_ARC_SYSTEM_INSTALLATION_NAME} \
    --install \
    --namespace "${GITHUB_ARC_SYSTEM_NAMESPACE}" \
    --create-namespace \
    --set serviceAccount.name="${GITHUB_ARC_SYSTEM_INSTALLATION_NAME}-gha-rs-controller" \
    oci://ghcr.io/actions/actions-runner-controller-charts/gha-runner-scale-set-controller
```

### Install GitHub ARC runner set

```bash
GITHUB_CONFIG_URL="https://github.com/<your_enterprise/org/repo>"
GITHUB_PAT="<PAT>"
helm update "${GITHUB_ARC_RUNNER_INSTALLATION_NAME}" \
    --install \
    --namespace "${GITHUB_ARC_RUNNER_NAMESPACE}" \
    --create-namespace \
    --set githubConfigUrl="${GITHUB_CONFIG_URL}" \
    --set githubConfigSecret.github_token="${GITHUB_PAT}" \
    --set controllerServiceAccount.name="${GITHUB_ARC_SYSTEM_INSTALLATION_NAME}-gha-rs-controller" \
    --set controllerServiceAccount.namespace="${GITHUB_ARC_SYSTEM_NAMESPACE}" \
    -f values.yaml \
    oci://ghcr.io/actions/actions-runner-controller-charts/gha-runner-scale-set
```

### Create custom `SecurityContextConstraint` and `ClusterRole` required for executing runners on OpenShift. Bind this role to the RunnerSet `ServiceAccount`

```bash
oc apply -f manifests/scc.yaml
oc apply -f manifests/cluster-role.yaml
oc policy add-role-to-user system:openshift:scc:github-arc -z ${GITHUB_ARC_RUNNER_INSTALLATION_NAME}-gha-rs-no-permission -n ${GITHUB_ARC_RUNNER_NAMESPACE}
```

## To create additional RunnerSets apply the following steps

* Maintain the original ARC system namespace and system installation name. You do not need to create additional ARC Systems. One ARC system can handle multiple runner sets.
* You do not need to create the `SecurityContextConstraint` and `ClusterRole` again.

DO NOT CHANGE THESE

```bash
export GITHUB_ARC_SYSTEM_NAMESPACE="github-arc-system"
export GITHUB_ARC_SYSTEM_INSTALLATION_NAME="github-arc-system"
```

Update these values for your desired runner namespace and installation name (referenced in Actions jobs)

```bash
export GITHUB_ARC_RUNNER_NAMESPACE_ADDITIONAL="github-arc-runners-additional"
export GITHUB_ARC_RUNNER_INSTALLATION_NAME_ADDITIONAL="github-arc-runners-additional"
```

```bash
GITHUB_CONFIG_URL="https://github.com/<your_enterprise/org/repo>"
GITHUB_PAT="<PAT>"
helm update "${GITHUB_ARC_RUNNER_INSTALLATION_NAME_ADDITIONAL}" \
    --install \
    --namespace "${GITHUB_ARC_RUNNER_NAMESPACE_ADDITIONAL}" \
    --create-namespace \
    --set githubConfigUrl="${GITHUB_CONFIG_URL}" \
    --set githubConfigSecret.github_token="${GITHUB_PAT}" \
    --set controllerServiceAccount.name="${GITHUB_ARC_SYSTEM_INSTALLATION_NAME}-gha-rs-controller" \
    --set controllerServiceAccount.namespace="${GITHUB_ARC_SYSTEM_NAMESPACE}" \
    -f values.yaml \
    oci://ghcr.io/actions/actions-runner-controller-charts/gha-runner-scale-set
```

### Apply the policy to the newly created RunnerSet

```bash
oc policy add-role-to-user system:openshift:scc:github-arc -z ${GITHUB_ARC_RUNNER_INSTALLATION_NAME_ADDITIONAL}-gha-rs-no-permission -n ${GITHUB_ARC_RUNNER_NAMESPACE_ADDITIONAL}
```
