# GitOps Control Plane

Control Plane repository defines the desired state of shared infrastructure components and enables self-service onboarding process for the application developer teams.

This git repository is part of the project [GitOps Bridge](https://github.com/gitops-bridge-dev/gitops-bridge)

Repository contains the following directories:

```yaml
- bootstrap      # The Applications and ApplicationSets used for Argo CD bootstrapping.
- charts         # Local Helm charts.
- clusters       # Cluster specific configurations, including values for addons.
- environments   # Environment specific configurations, including values for addons.
- global         # The default `values.yaml` for charts used by addons.
```

## Argo CD Bootstrapping
The `bootstrap` folder is used as the source for the initial `Application` created by Terraform after standing up an instance of Argo CD. It bootstraps the instance with an `Application` that deploys `ApplicationSets` for managing cluster addons, and an `ApplicationSet` for deploying cluster-specific `Applications` (e.g. from the `clusters/<name>` folders).

## Addons
Each cluster can opt into addons deployed into it.

The `global/addons`, `clusters/<name>/addons`, and `environments/<name>/addons` folders are used to provide `values.yaml` to the addons charts.

For example, the `argo-rollouts` addon chart used in the `dev` environment and `gke` cluster, is supplied the following values files (in order of precidence):
- `global/addons/argo-rollouts/values.yaml`
- `environments/dev/addons/argo-rollouts/values.yaml`
- `clusters/gke/addons/argo-rollouts/values.yaml`

### Creating a New Addon
1. Create a folder with the same name as the chart under `global/addons`, `environments/*/addons`, and `clusters/*/addons`.
2. Add an empty `values.yaml` to each of those folders.
3. Add the chart information in `bootstrap/addons-app.yaml` under the `spec.source.helm.valuesObject.addons` list.
    ```yaml
    spec:
      source:
        helm:
        valuesObject:
          addons:
          - chartName: argo-rollouts
            chartRepo: https://argoproj.github.io/argo-helm
            chartVersionDefault: 2.32.0
    ```
4. In the Terraform configuration, add the cooresponding `enable_<chart_name_snakecase>` to the annotations of the clusters that should recieve the addon.

## Cluster Apps
Each cluster will need a folder with the same name (as what's defined in Argo CD) under `clusters/`. The `clusters` `ApplicationSet` (deployed by the Argo CD bootstrapping) will create an `Application` to deploy any `Application` manifests in the `apps` folder. A cluster can have any number of arbitrary `Applications` deployed to it by simply adding them to it's corriesponding folder. Including ones that point to another GitOps repo that contains the application-specific configurations.

For example, to create an `Application` for the `gke-dev` cluster, simply add the manifest to the `clusters/gke-dev/apps` folder. From there the `cluster-gke-dev-apps` `Applicaiton` (created by the `clusters-apps` `ApplicaitonSet`) will deploy it so that Argo CD will pick it up and start deploying the resources to the cluster.