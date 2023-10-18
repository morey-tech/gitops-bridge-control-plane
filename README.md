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
The `bootstrap` folder is used as the source for the initial `Application` created by Terraform after standing up an instance of Argo CD. It bootstraps the instance with an `Application` that deploys `ApplicationSets` for managing cluster addons, and an `ApplicationSet` for deploying cluster-specific workloads (e.g. from the `clusters/<name>` folders).

## Clusters
Each cluster will need a folder with the same name (as what's defined in Argo CD) under `clusters/`. The `clusters` `ApplicationSet` (deployed by the Argo CD bootstrapping) will create an `Application` to deploy any manifests in that folder (excluding the `addons` folder). A cluster can have any number of arbitrary manifests deployed to it by simply adding them to it's corriesponding folder.

## Addons
Each cluster can opt into addons deployed into it.

The `global/addons`, `clusters/<name>/addons`, and `environments/<name>/addons` folders are used to provide `values.yaml` to the addons charts.

For example, the `argo-rollouts` addon chart used in the `dev` environment and `gke` cluster, is supplied the following values files (in order of precidence):
- `global/addons/argo-rollouts/values.yaml`
- `environments/dev/addons/argo-rollouts/values.yaml`
- `clusters/gke/addons/argo-rollouts/values.yaml`