# capi-kamaji-vsphere-full

![Version: 1.0.1](https://img.shields.io/badge/Version-1.0.1-informational?style=flat-square) ![Type: application](https://img.shields.io/badge/Type-application-informational?style=flat-square) ![AppVersion: 1.32.0](https://img.shields.io/badge/AppVersion-1.32.0-informational?style=flat-square)

Helm umbrella chart for deploying Kamaji Clusters on vSphere using Cluster API. Including CSI driver and cluster autoscaler. Requires separate kamaji-etcd installation.

**Homepage:** <https://github.com/clastix/capi-kamaji-vsphere-full>

## Maintainers

| Name | Email | Url |
| ---- | ------ | --- |
| Clastix Labs | <authors@clastix.labs> | <https://clastix.io> |

## Source Code

* <https://github.com/clastix/cluster-api-kamaji-vsphere>
* <https://github.com/clastix/kamaji-etcd>
* <https://github.com/clastix/vsphere-csi>
* <https://github.com/kubernetes/autoscaler>

## Requirements

| Repository | Name | Version |
|------------|------|---------|
| https://clastix.github.io/charts | capi-kamaji-vsphere | ~0.2.6 |
| https://clastix.github.io/charts | vsphere-csi | ~0.1.2 |
| https://kubernetes.github.io/autoscaler | cluster-autoscaler | ~9.48.0 |

## Values

| Key | Type | Default | Description |
|-----|------|---------|-------------|
| capi-kamaji-vsphere | object | `{"cluster":{"controlPlane":{"dataStoreName":"<CLUSTER_NAME>","kubelet":{"preferredAddressTypes":["InternalIP"]},"network":{"serviceAddress":"<CLUSTER_ADDRESS>","serviceAnnotations":{"metallb.io/loadBalancerIPs":"<CLUSTER_ADDRESS>"}},"version":"v1.32.0"},"metrics":{"enabled":true},"name":"<CLUSTER_NAME>"},"enabled":true,"ipamProvider":{"gateway":"<PLACEHOLDER>","prefix":"<PLACEHOLDER>","ranges":["<PLACEHOLDER>"]},"nodePools":[{"addressesFromPools":{"enabled":true},"autoscaling":{"enabled":true,"maxSize":"6","minSize":"2"},"dataStore":"<DATASTORE_NAME>","diskGiB":40,"folder":"<FOLDER_NAME>","memoryMiB":4096,"name":"default","nameServers":["8.8.8.8"],"network":"<NETWORK_NAME>","numCPUs":2,"replicas":3,"resourcePool":"<RESOURCE_POOL_NAME>","staticRoutes":[],"storagePolicyName":"<STORAGE_POLICY_NAME>","template":"ubuntu-2404-kube-v1.32.0","users":[{"lockPassword":false,"name":"<USER_NAME>","passwd":"<MKPASSWD>","sshAuthorizedKeys":["<SSH_PUBLIC_KEY>"]}]}],"vSphere":{"dataCenter":"<DATACENTER_NAME>","identityRef":{"name":"vsphere-cluster-identity","type":"VSphereClusterIdentity"},"insecure":true,"server":"<VCENTER_NAME>"},"vSphereCloudControllerManager":{"image":{"tag":"v1.32.0"}}}` | This section configures the CAPI Kamaji vSphere chart |
| capi-kamaji-vsphere.cluster | object | `{"controlPlane":{"dataStoreName":"<CLUSTER_NAME>","kubelet":{"preferredAddressTypes":["InternalIP"]},"network":{"serviceAddress":"<CLUSTER_ADDRESS>","serviceAnnotations":{"metallb.io/loadBalancerIPs":"<CLUSTER_ADDRESS>"}},"version":"v1.32.0"},"metrics":{"enabled":true},"name":"<CLUSTER_NAME>"}` | Cluster configuration |
| capi-kamaji-vsphere.ipamProvider | object | `{"gateway":"<PLACEHOLDER>","prefix":"<PLACEHOLDER>","ranges":["<PLACEHOLDER>"]}` | IPAM provider configuration |
| capi-kamaji-vsphere.nodePools | list | `[{"addressesFromPools":{"enabled":true},"autoscaling":{"enabled":true,"maxSize":"6","minSize":"2"},"dataStore":"<DATASTORE_NAME>","diskGiB":40,"folder":"<FOLDER_NAME>","memoryMiB":4096,"name":"default","nameServers":["8.8.8.8"],"network":"<NETWORK_NAME>","numCPUs":2,"replicas":3,"resourcePool":"<RESOURCE_POOL_NAME>","staticRoutes":[],"storagePolicyName":"<STORAGE_POLICY_NAME>","template":"ubuntu-2404-kube-v1.32.0","users":[{"lockPassword":false,"name":"<USER_NAME>","passwd":"<MKPASSWD>","sshAuthorizedKeys":["<SSH_PUBLIC_KEY>"]}]}]` | Node Pools configuration |
| capi-kamaji-vsphere.vSphere | object | `{"dataCenter":"<DATACENTER_NAME>","identityRef":{"name":"vsphere-cluster-identity","type":"VSphereClusterIdentity"},"insecure":true,"server":"<VCENTER_NAME>"}` | vSphere configuration |
| capi-kamaji-vsphere.vSphere.identityRef | object | `{"name":"vsphere-cluster-identity","type":"VSphereClusterIdentity"}` | Identity reference for vSphere Cluster Identity |
| capi-kamaji-vsphere.vSphereCloudControllerManager | object | `{"image":{"tag":"v1.32.0"}}` | vSphere Cloud Controller Manager configuration |
| cluster-autoscaler | object | `{"autoDiscovery":{"clusterName":"<CLUSTER_NAME>","namespace":"<CLUSTER_NAMESPACE>"},"cloudProvider":"clusterapi","clusterAPIKubeconfigSecret":"<CLUSTER_NAME>-kubeconfig","clusterAPIMode":"kubeconfig-incluster","enabled":true}` | This section configures the Cluster Autoscaler |
| cluster-autoscaler.autoDiscovery | object | `{"clusterName":"<CLUSTER_NAME>","namespace":"<CLUSTER_NAMESPACE>"}` | Auto Discovery Options |
| cluster-autoscaler.autoDiscovery.clusterName | string | `"<CLUSTER_NAME>"` | clusterName: <CLUSTER_NAME> |
| cluster-autoscaler.autoDiscovery.namespace | string | `"<CLUSTER_NAMESPACE>"` | namespace: <CLUSTER_NAMESPACE> |
| cluster-autoscaler.clusterAPIKubeconfigSecret | string | `"<CLUSTER_NAME>-kubeconfig"` | Cluster API Kubeconfig Secret |
| cluster-autoscaler.clusterAPIMode | string | `"kubeconfig-incluster"` | Cluster API Mode Configuration: must be set to kubeconfig-incluster |
| vsphere-csi | object | `{"cluster":{"name":"<CLUSTER_NAME>","targetNamespace":"kube-system"},"enabled":true,"storageClass":{"allowVolumeExpansion":true,"default":true,"enabled":true,"name":"vsphere-csi","parameters":{"storagepolicyname":"<CLUSTER_NAME>"},"reclaimPolicy":"Delete","volumeBindingMode":"WaitForFirstConsumer"}}` | This section configures the vSphere CSI driver |
| vsphere-csi.cluster | object | `{"name":"<CLUSTER_NAME>","targetNamespace":"kube-system"}` | Cluster configuration |
| vsphere-csi.storageClass | object | `{"allowVolumeExpansion":true,"default":true,"enabled":true,"name":"vsphere-csi","parameters":{"storagepolicyname":"<CLUSTER_NAME>"},"reclaimPolicy":"Delete","volumeBindingMode":"WaitForFirstConsumer"}` | Storage Class configuration |

----------------------------------------------
Autogenerated from chart metadata using [helm-docs v1.11.0](https://github.com/norwoodj/helm-docs/releases/v1.11.0)
