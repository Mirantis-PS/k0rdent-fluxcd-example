# k0rdent FluxCD GitOps repo example

This repo is generated from the k0rdent GitOps repo template.

## Change log

1. Created the repo from the template
2. Bootstrapped FluxCD to the management cluster and connected it to sync configurations from the `management-clusters/management-cluster-1/flux` directory (commits [1b52037cad2c4c4dc545dae909bdc3beb4005e08](https://github.com/Mirantis-PS/k0rdent-fluxcd-example/commit/1b52037cad2c4c4dc545dae909bdc3beb4005e08) and [ff8fe3904cccd4b6a7967a00ef802731969e41ec](https://github.com/Mirantis-PS/k0rdent-fluxcd-example/commit/ff8fe3904cccd4b6a7967a00ef802731969e41ec))
3. [Generated base k0rdent configuration](https://github.com/Mirantis-PS/k0rdent-fluxcd-example/commit/239d6fd0319b14c19ddeaeaf6b079a21d3aef899) with the `task configure` command. Flux synced the state and installs kcm controller, GitRepository and Kustomization resources to sync the state of k0rdent objects from corresponding directories
4. For demo purposes some required [components are installed](https://github.com/Mirantis-PS/k0rdent-fluxcd-example/commit/df59207440fdaa7e18697e0c44b74093019eaeae) to the management cluster. We added configurations for them to the `base/components` directory, link in the kustomization file of the management cluster (e.g. `management-clusters/management-cluster-1/components/helm-repo/kustomization.yaml`) and added flux sync configurations (e.g. ``management-clusters/management-cluster-1/flux/helm-repo.yaml`) to demonstrate how the common configuration for any component can be shared across multiple management clusters. Components are:
  - Helm repo. This is the simple OCI helm registry, we will use it later to upload and store helm charts for custom cluster and service templates. In real environments you should use production-ready solutions, like Harbor, Artifactory, etc.
  - Bitnami sealed secrets. It will be used to encrypt provider credentials, so we can safely store them in git repositiry. In production environments use some common alternatives like cloud secret managers, Hashicorp Vault, etc.
