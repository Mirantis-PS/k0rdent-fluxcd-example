# k0rdent FluxCD GitOps repo example

This repo is generated from the k0rdent GitOps repo template.

## Change log

1. Created the repo from the template
2. Bootstrapped FluxCD to the management cluster and connected it to sync configurations from the `management-clusters/management-cluster-1/flux` directory (commits [1b52037cad2c4c4dc545dae909bdc3beb4005e08](https://github.com/Mirantis-PS/k0rdent-fluxcd-example/commit/1b52037cad2c4c4dc545dae909bdc3beb4005e08) and [ff8fe3904cccd4b6a7967a00ef802731969e41ec](https://github.com/Mirantis-PS/k0rdent-fluxcd-example/commit/ff8fe3904cccd4b6a7967a00ef802731969e41ec))
3. [Generated base k0rdent configuration](https://github.com/Mirantis-PS/k0rdent-fluxcd-example/commit/239d6fd0319b14c19ddeaeaf6b079a21d3aef899) with the `task configure` command. Flux synced the state and installs kcm controller, GitRepository and Kustomization resources to sync the state of k0rdent objects from corresponding directories
4. For demo purposes some required [components are installed](https://github.com/Mirantis-PS/k0rdent-fluxcd-example/commit/df59207440fdaa7e18697e0c44b74093019eaeae) to the management cluster. We added configurations for them to the `base/components` directory, link in the kustomization file of the management cluster (e.g. `management-clusters/management-cluster-1/components/helm-repo/kustomization.yaml`) and added flux sync configurations (e.g. `management-clusters/management-cluster-1/flux/helm-repo.yaml`) to demonstrate how the common configuration for any component can be shared across multiple management clusters. Components are:
    - Helm repo. This is the simple OCI helm registry, we will use it later to upload and store helm charts for custom cluster and service templates. In real environments you should use production-ready solutions, like Harbor, Artifactory, etc.
    - Bitnami sealed secrets. It will be used to encrypt provider credentials, so we can safely store them in git repositiry. In production environments use some common alternatives like cloud secret managers, Hashicorp Vault, etc.
5. [Added AWS, Azure and Openstack credential](https://github.com/Mirantis-PS/k0rdent-fluxcd-example/commit/119b776aa718a0933b040e9e9fe8162ad43ee993) examples to the `management-clusters/management-cluster-1/k0rdent/credentials` folder. All credentials are deployed to the `kcm-system` namespace. For each credential we also added the cluster-deployment-patch.yaml file. This patch will help us to add the reference to the corresponding credential from any ClusterDeployments later. As the result we have ready Credentials in the `kcm-system` namespace:
    ```
    > kubectl -n kcm-system get credentials
    NAME                           READY   DESCRIPTION
    aws-credential-cloud-1         true    AWS credentials
    aws-credential-cloud-2         true    AWS credentials
    azure-credential-cloud-1       true    Azure credentials
    azure-credential-cloud-2       true    Azure credentials
    openstack-credential-cloud-1   true    OpenStack credentials
    openstack-credential-cloud-2   true    OpenStack credentials
    ```

