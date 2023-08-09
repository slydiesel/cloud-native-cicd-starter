# CICD Starter Repository

This repository contains an example methodology of deployment following the ArgoCD platform's app-of-apps pattern.  This provides the capability of managing configurations for multiple applications across multiple environments via a GitOps flow.  ArgoCD will poll against your defined sources (eg. chart definitions and values configurations), automatically render any manifest changes, and apply them accordingly all the while visualizing the progression of resources and their health.

# Getting Started

1. Clone this repository

    ```bash
    git clone https://github.com/cloud-native-cicd-starter.git
    cd cloud-native-cicd-starter
    ```

2. Refer to `app-of-apps-values.yaml`

    This file is where we define our applications and environments, aka our sources and our targets.  The configurations here will default to an auto-syncing strategy for our applications however environments may individually override this behavior as desired via `syncOverrides`.  Applications may also be gated from these environments by supplying an `applicationFilter`.

    Notable structure explained:
    - `syncPolicy` defines our default syncing strategy for each environment
        - `automated` configures auto-sync for ArgoCD to poll our chart definitions and value configurations for changes and deploy them automatically
        - `syncOptions` provides additional sync options as displayed [here](https://github.com/argoproj/argo-cd/blob/master/docs/operator-manual/application.yaml#L186)
            - We use this to create namespaces automatically for any new environments
    - `gitops` defines our designated repository where our GitOps occurs or more notably where our individual environments' value configurations are held and managed
        - `repoURL` specifies the repository URL
    - `spec`
        - `applications` define our list of sources aka our chart definitions, their repositories, file pathing within those repositories, and potentially any branching needs (for instance, test an application deployment that has not been pushed to your main branch)
        - `environments` define our list of targets aka the destination cluster, its namespace, and the file path of its individual app ocnfigurations within your GitOps repository (which would be this one by default).

    Together, applications and environments will generate a collection of ArgoCD Application manifests that are deployed to the desired clusters and namespaces.  These applications will be configured individually by our configurations located in the folder structure detailed below.

3.  Make note of the `environments` folder in this repository

    In this directory, you will find folders for each environment (e.g. `development`, `staging`, and `production`) which contain individual `values.yaml` files for each application.  These value configuration files follow the naming format `<application.name>-values.yaml`.  Our rendered ArgoCD Application manifests are configured to monitor their respective files for syncing purposes within the relevant environment folder.  Your environments are all laid out in front of you.

# Install `app-of-apps` chart

1. Modify your `app-of-apps-values.yaml` according to the structures defined above per your needs.

2. Install the chart via **Helm**

    ```bash
    helm upgrade --install app-of-apps ./argocd-app-of-apps -f app-of-apps-values.yaml -n argocd
    ```

    NOTE: It is important to install the app-of-apps chart into the `argocd` namespace as this is the namespace ArgoCD will monitor.  From there, it will dish out individual kubernetes resources for the actuall deployments to their respective target namespaces.

# Add an application

0. Ensure a container image exists and you have an image pull secret in your target namespace

1. Create a chart in your organization's chart repository

    You can kickstart a chart using `helm create`
    
    ```bash
    helm create example-app
    ```

    Ensure this is pushed to a branch in the remote repository

2. Update `app-of-apps-values.yaml`

    Add an application following the format demonstrated and described, for example:

    ```yaml
    - name: example-app
      path: helm/example-app
      repoURL: https://github.com/yourOrganization/your-chart-repo.git
      targetRevision: HEAD # or branch ref
    ```

3. Add `example-app-values.yaml` to your desired environments' folders and configure accordingly

4. Upgrade your `app-of-apps` installation

    ```bash
    helm upgrade --install app-of-apps ./argocd-app-of-apps -f app-of-apps-values.yaml -n argocd
    ```

    This application will now be deployed and auto-synced as expected.

5. Tailor to your application's needs

    Perhaps your chart has environmental variables and potentially some that are sensitive.  You would likely be pulling these in from a `ConfigMap` or a `Secret`.  These references would either be supplied via the chart definitions themselves or fed into the chart by value configurations in our individual app values per environment.  An example of this would be found in `environments/development/baseline-laravel-values.yaml`:

    ```yaml
    env:
    - name: APP_NAME
      valueFrom:
        secretKeyRef:
          name: laravel-app-secret
          key: APP_NAME
    - name: APP_ENV
      valueFrom:
        secretKeyRef:
          name: laravel-app-secret
          key: APP_ENV
    - name: APP_KEY
      valueFrom:
        secretKeyRef:
          name: laravel-app-secret
          key: APP_KEY
    ```

    Perhaps your docker repo is private and you need an image pull secret.  You can create a docker registry secret in your environment's namespace and configure an image pull secret in your values like so:

    ```yaml
    imagePullSecrets:
    - name: image-pull-secret
    ```

# Add an environment

1. Create a folder in the `environments` directory

2. Create and configure a values file for each respective application to be deployed within that environment

3. Update `app-of-apps-values.yaml` to include your new environment

    ```yaml
    - name: arbitrary
      path: environments/arbitrary
      targetRevision: HEAD
      destination:
        server: https://kubernetes.default.svc
        namespace: arbitrary
    ```

4. Upgrade your `app-of-apps` installation

    ```bash
    helm upgrade --install app-of-apps ./argocd-app-of-apps -f app-of-apps-values.yaml -n argocd
    ```

5.  Add any required secrets or config maps to your environment's namespace

# Automate your Builds

Refer to the [README](./argo-workflow-ci/README.md) of our example `argo-workflow-ci` chart.  This chart will install a github webhook event source and buildkit workflow that will build for a root level Dockerfile.  Your mileage may vary but the chart may be customized to your needs.