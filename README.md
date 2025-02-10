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
6. k0rdent provdes a set of built-in cluster and service templates. But to demonstrate how to extend it we [added a set of custom templates](https://github.com/Mirantis-PS/k0rdent-fluxcd-example/commit/17dfe2643e5cb93d6cbfef5bac97e5bebb3a4094) under the `management-clusters/management-cluster-1/k0rdent/templates` directory. Helm charts that used for this example are also added to the `helm-chart-examples` directory. When flux syncs the state from the repo, we have new valid ClusterTemplate and ServiceTemplate objects in the `kcm-system` namespace:
    ```
    > kubectl -n kcm-system get clustertemplate
    NAME                                   VALID
    custom-aws-standalone-cp-0-0-1         true
    custom-aws-standalone-cp-0-0-2         true
    custom-azure-standalone-cp-0-0-1       true
    custom-azure-standalone-cp-0-0-2       true
    custom-openstack-standalone-cp-0-0-1   true
    custom-openstack-standalone-cp-0-0-2   true
    ```

    ```
    kubectl -n kcm-system get servicetemplate
    NAME                          VALID
    custom-ingress-nginx-4-11-0   true
    custom-ingress-nginx-4-11-3   true
    custom-kyverno-3-2-6          true
    ```

    Additionaly, each cluster template directory has the `cluster-deployment-patch.yaml` file that can be used later in the main [kustomization.yaml](./management-clusters/management-cluster-1/k0rdent/kustomization.yaml) file to set the correspinding cluster template reference in any `ClusterDeployment` object. And in each service template directory the `service-template-patch.yaml` file that can be used in the main `kustomization.yaml` file to set the name of `ServiceTemplate` in `ClusterDeployment` or `MultiClusterService` objects.

7. When credentials are configured and some cluster and service templates are ready, we can deploy managed clusters. As an example [we added 2 AWS and 1 Azure clusters](https://github.com/Mirantis-PS/k0rdent-fluxcd-example/commit/0192d8faa21013af2c2b223ea34b6ea790ed55b7). Also, to show how configurations can be managed we prepared the following structure:
    ```
    /management-clusters/management-cluster-1/k0rdent/cluster-deployments
    ├── base
    |   ├── aws
    |   |   └── t3-small-us-west-2
    |   └── azure
    |       └── standard-a4-westus
    └── kcm-system
        ├── aws
        |   ├── aws-managed-cluster-1
        |   └── aws-managed-cluster-2
        └── azure
            └── azure-managed-cluster-1
    ```

    - `base` directory contains some base cluster configurations that can be reused by any managed clusters
    - `kcm-system` is the directory for the `kcm-system` namespace. Currently, all credential and template objects are deployed to that namespace and can be used by `ClusterDeployment` objects that deployed to the same namespace. As an example, we use `kcm-system` as the namespace where k0rdent platform admins develop and test templates and cluster deployments. We deployed several managed clusters:
      1. `aws-managed-cluster-1`:
        - it uses the `t3-small-us-west-2` ClusterDeployment base
        - in the main [`kustomization.yaml`](./management-clusters/management-cluster-1/k0rdent/kustomization.yaml) file it's specified that it should use the `custom-aws-standalone-cp-0-0-1` ClusterTemplate
        - in the main [`kustomization.yaml`](./management-clusters/management-cluster-1/k0rdent/kustomization.yaml) file it's specified that it should use the `aws-cloud-1` Credential
        - in the main [`kustomization.yaml`](./management-clusters/management-cluster-1/k0rdent/kustomization.yaml) file it's specified that `custom-ingress-nginx-4-11-0` service must be deployed to the managed cluster 
      2. `aws-managed-cluster-2`:
        - it uses the `t3-small-us-west-2` ClusterDeployment base
        - in the main [`kustomization.yaml`](./management-clusters/management-cluster-1/k0rdent/kustomization.yaml) file it's specified that it should use the `custom-aws-standalone-cp-0-0-1` ClusterTemplate
        - in the main [`kustomization.yaml`](./management-clusters/management-cluster-1/k0rdent/kustomization.yaml) file it's specified that it should use the `aws-cloud-2` Credential
        - in the main [`kustomization.yaml`](./management-clusters/management-cluster-1/k0rdent/kustomization.yaml) file it's specified that `custom-ingress-nginx-4-11-0` service must be deployed to the managed cluster 
      2. `azure-managed-cluster-1`:
        - it uses the `t3-small-us-west-2` ClusterDeployment base
        - in the main [`kustomization.yaml`](./management-clusters/management-cluster-1/k0rdent/kustomization.yaml) file it's specified that it should use the `custom-azure-standalone-cp-0-0-1` ClusterTemplate
        - in the main [`kustomization.yaml`](./management-clusters/management-cluster-1/k0rdent/kustomization.yaml) file it's specified that it should use the `azure-cloud-1` Credential

    After some time we can check that all 3 clusters are fully deployed:
    ```
    > kubectl -n kcm-system get clusterdeployments
    NAME                      READY   STATUS
    aws-managed-cluster-1     True    ClusterDeployment is ready
    aws-managed-cluster-2     True    ClusterDeployment is ready
    azure-managed-cluster-1   True    ClusterDeployment is ready
    ```

    And we can access all of them to check that they are operational and required services are deployed:
    1. `aws-managed-cluster-1`:
        ```
        > kubectl -n kcm-system get secret aws-managed-cluster-1-kubeconfig -o jsonpath='{.data.value}' | base64 -d > bin/aws-managed-cluster-1.kubeconfig

        > KUBECONFIG=bin/aws-managed-cluster-1.kubeconfig kubectl get no
        NAME                                   STATUS   ROLES           AGE   VERSION
        aws-managed-cluster-1-cp-0             Ready    control-plane   21m   v1.31.2+k0s
        aws-managed-cluster-1-md-cr855-gcwz8   Ready    <none>          19m   v1.31.2+k0s
        aws-managed-cluster-1-md-cr855-kv6jd   Ready    <none>          19m   v1.31.2+k0s

        > KUBECONFIG=bin/aws-managed-cluster-1.kubeconfig kubectl -n ingress-nginx get po
        NAME                                        READY   STATUS    RESTARTS   AGE
        ingress-nginx-controller-86bd747cf9-n5zfj   1/1     Running   0          18m
        ```
    2. `aws-managed-cluster-1`:
        ```
        > kubectl -n kcm-system get secret aws-managed-cluster-2-kubeconfig -o jsonpath='{.data.value}' | base64 -d > bin/aws-managed-cluster-2.kubeconfig
        
        > KUBECONFIG=bin/aws-managed-cluster-2.kubeconfig kubectl get no
        NAME                                   STATUS   ROLES           AGE   VERSION
        aws-managed-cluster-2-cp-0             Ready    control-plane   24m   v1.31.2+k0s
        aws-managed-cluster-2-md-zcw8b-5gnm7   Ready    <none>          21m   v1.31.2+k0s
        aws-managed-cluster-2-md-zcw8b-t97tv   Ready    <none>          22m   v1.31.2+k0s

        > KUBECONFIG=bin/aws-managed-cluster-2.kubeconfig kubectl -n ingress-nginx get po
        NAME                                        READY   STATUS    RESTARTS   AGE
        ingress-nginx-controller-86bd747cf9-gw8xr   1/1     Running   0          23s
        ```
    3. `azure-managed-cluster-1`:
        ```
        > kubectl -n kcm-system get secret azure-managed-cluster-1-kubeconfig -o jsonpath='{.data.value}' | base64 -d > bin/azure-managed-cluster-1.kubeconfig
        
        > KUBECONFIG=bin/azure-managed-cluster-1.kubeconfig kubectl get no
        NAME                                     STATUS   ROLES           AGE   VERSION
        azure-managed-cluster-1-cp-0             Ready    control-plane   23m   v1.31.1+k0s
        azure-managed-cluster-1-md-94sct-vps4l   Ready    <none>          22m   v1.31.1+k0s
        azure-managed-cluster-1-md-94sct-zvbwq   Ready    <none>          21m   v1.31.1+k0s
        ```
8. One of the k0rdent features is `MultiClusterService`, which allows to deploy any service or set of services to multiple `ClusterDeployment`s that are selected by labels or expressions. [As an example](https://github.com/Mirantis-PS/k0rdent-fluxcd-example/commit/b6fe6e2bdcca8c7d6338d98bc2550ea7a2c36b92), we added [`protected-kyverno`](./management-clusters/management-cluster-1/k0rdent/multiclusterservices/protected-kyverno) global service. The version of kyverno for the `MultiClusterService`, required labels for `ClusterDeployment`s are set in the main [kustomization.yaml](./management-clusters/management-cluster-1/k0rdent/kustomization.yaml) file. This configuration creates the `MultiClusterService` that picks all cluster deployments with `kyverno-version=3.2.6` and `protected=kyverno` labels and deploys the kyverno service specified in the `custom-kyverno-3-2-6` ServiceTemplate. Due to we added these labels to the all existing ClusterDeployments, we can check any of them and make sure that the service is deployed:
    ```
    > KUBECONFIG=bin/aws-managed-cluster-1.kubeconfig kubectl -n kyverno get po  
    NAME                                                       READY   STATUS      RESTARTS   AGE
    kyverno-admission-controller-96c5d48b4-w8b44               1/1     Running     0          30m
    kyverno-background-controller-65f9fd5859-6gm25             1/1     Running     0          30m
    kyverno-cleanup-admission-reports-28986490-2kvz9           0/1     Completed   0          7m36s
    kyverno-cleanup-cluster-admission-reports-28986490-c6nb5   0/1     Completed   0          7m36s
    kyverno-cleanup-cluster-ephemeral-reports-28986490-2k6g4   0/1     Completed   0          7m36s
    kyverno-cleanup-controller-848b4c579d-5nbwz                1/1     Running     0          30m
    kyverno-cleanup-ephemeral-reports-28986490-l5kgs           0/1     Completed   0          7m36s
    kyverno-cleanup-update-requests-28986490-sn84m             0/1     Completed   0          7m36s
    kyverno-reports-controller-6f59fb8cd6-l7gbv                1/1     Running     0          30m
    ```
