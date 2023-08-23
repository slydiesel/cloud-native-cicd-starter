# ArgoCD app-of-apps Chart

This chart will allow you to define a configurable number of sources (aka Helm Charts) and deploy them to a configurable number of targets packaged together.  To accomplish this, users will configure their `values.yaml` which will drive the population of custom `Application` resources which is a custom resource definition provided by Argo.

# Prerequisites

1. Helm Chart repository of choice
2. Github repo (you may fork this one to start your journey)
3. Container images (we host some public images on Docker Hub for the purpose of this deployment)
4. ArgoCD installed to your cluster

# Deploy Workflows

1. Populate `values.yaml`

    Overview of updating our values file:
    - Define the syncing behavior of your deployments
        - `syncPolicy`
            - `automated`: enable automatic syncing in ArgoCD for a deployment as well as configure additional automation behaviors
            - `syncOptions`: additional flags for syncing behaviors such as automatic namespace creation
    - Specify the repository that ArgoCD will monitor for value configurations (in addition to your chart sources)
        - `gitops.repoURL`
    - Define your applications and environments
        - `spec.applications`: your collection of chart sources
            - `name`: name of your chart deployment
            - `path`: path to the chart in the chart repository/registry
            - `repoURL`: repository/registry
            - `targetRevision`: revision of the chart repository
        - `spec.environments`: your collection of environments, or rather, where you are deploying your apps to
            - `name`: name of your environment
            - `path`: folder path in the GitOps repo that contains the individual app configurations for the environment
            - `targetRevision`: branch of the GitOps repo
            - `destination`: the cluster and namespace to deploy your app
                - `server`: the cluster server URL (local or external)
                - `namespace`: the namespace to create and/or deploy your app to

2. Install the chart

    ```bash
    git clone https://github.com/krumIO/cloud-native-cicd-starter.git
    cd cloud-native-cicd-starter
    helm upgrade --install app-of-apps ./argocd-app-of-apps -n argocd
    ```
