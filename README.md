# k0rdent FluxCD GitOps repo example

This repo is generated from the k0rdent GitOps repo template.

## Change log

1. Created the repo from the template
2. Bootstrapped FluxCD to the management cluster and connected it to sync configurations from the `management-clusters/management-cluster-1/flux` directory (commits [1b52037cad2c4c4dc545dae909bdc3beb4005e08](https://github.com/Mirantis-PS/k0rdent-fluxcd-example/commit/1b52037cad2c4c4dc545dae909bdc3beb4005e08) and [ff8fe3904cccd4b6a7967a00ef802731969e41ec](https://github.com/Mirantis-PS/k0rdent-fluxcd-example/commit/ff8fe3904cccd4b6a7967a00ef802731969e41ec))
3. [Generated base k0rdent configuration](https://github.com/Mirantis-PS/k0rdent-fluxcd-example/commit/239d6fd0319b14c19ddeaeaf6b079a21d3aef899) with the `task configure` command. Flux synced the state and installs kcm controller, GitRepository and Kustomization resources to sync the state of k0rdent objects from corresponding directories
