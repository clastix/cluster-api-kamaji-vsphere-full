# Cluster API vSphere with Kamaji

This Helm umbrella chart deploys a complete Kubernetes cluster on vSphere using Cluster API and Kamaji.

## Components
This chart deploys a complete Kubernetes cluster on vSphere using Cluster API and Kamaji, leveraging the Kamaji etcd datastore for high availability. It includes the following components:

- [capi-kamaji-vsphere](https://github.com/clastix/charts/tree/main/charts/capi-kamaji-vsphere)
- [vsphere-csi](https://github.com/clastix/charts/tree/main/charts/vsphere-csi)
- [cluster-autoscaler](https://github.com/kubernetes/autoscaler/tree/master/charts/cluster-autoscaler)

**Note**: This chart requires a separate [kamaji-etcd](https://github.com/clastix/charts/tree/main/charts/kamaji-etcd) installation, see below.

## Installation

### Prepare etcd datastore
This chart requires a Kamaji etcd datastore to be installed before deploying the cluster. Follow these steps to install the etcd datastore:

Create a file `kamaji-etcd-values.yaml` with tenant-specific etcd values:

```yaml
# -----------------------------------------------------------------------------
# <CLUSTER_NAME>-etcd-values.yaml chart values
# -----------------------------------------------------------------------------
fullnameOverride: "<CLUSTER_NAME>-etcd" # Replace with your CLUSTER_NAME
datastore:
  enabled: true
  name: "<CLUSTER_NAME>" # Replace with your CLUSTER_NAME
affinity:
  podAntiAffinity:
    requiredDuringSchedulingIgnoredDuringExecution:
    - labelSelector:
        matchExpressions:
          - key: app.kubernetes.io/instance
            operator: In
            values:
              - "<CLUSTER_NAME>-etcd"  # Replace with your CLUSTER_NAME
      topologyKey: kubernetes.io/hostname
```

### Install etcd datastore

```bash
helm repo add clastix https://clastix.github.io/charts
helm repo update
helm install <CLUSTER_NAME>-etcd clastix/kamaji-etcd \
  --namespace <CLUSTER_NAMESPACE> --create-namespace \
  --values <CLUSTER_NAME>-etcd-values.yaml \
  --wait --timeout=600s
```

## Install cluster

Cluster API requires to authenticate against vSphere using a secret containing the vSphere credentials and a configuration file for the Cloud Controller Manager. You also need to create a secret for the CSI driver configuration.

### Shared vSphere Cluster Identity
vSphere credentials are provided via `VSphereClusterIdentity`, a cluster scoped resource that enables multiple `VSphereClusters` to share the same set of credentials. If you plan to use the same vSphere credentials across multiple clusters, you can create a `VSphereClusterIdentity`:

```yaml
# -----------------------------------------------------------------------------
# vsphere-cluster-identity.yaml
# -----------------------------------------------------------------------------
apiVersion: infrastructure.cluster.x-k8s.io/v1beta1
kind: VSphereClusterIdentity
metadata:
  name: vsphere-cluster-identity
spec:
  secretName: global-vsphere-secret # Name of the Secret containing vSphere credentials
  allowedNamespaces:
    selector:
      matchLabels: {} # allow all namespaces
```

Create the `VSphereClusterIdentity` cluster wide resource:

```bash
kubectl apply -f vsphere-cluster-identity.yaml
```

Also make sure the referenced `global-vsphere-secret` is created in the `capi-providers` namespace with the correct vSphere credentials. If not already created, you can use the following example to create it:

```yaml
# -----------------------------------------------------------------------------
# global-vsphere-secret.yaml 
# -----------------------------------------------------------------------------
apiVersion: v1
kind: Secret
metadata:
  name: global-vsphere-secret
  namespace: capi-providers
stringData:
  username: "administrator@vsphere.local"
  password: "password"
```

Create the global vSphere secret in the `capi-providers` namespace:

```bash
kubectl -n capi-providers apply -f global-vsphere-secret.yaml
```

**Note:** you have to create the global vSphere secret only once, it will be used by all clusters using the same `VSphereClusterIdentity`. More details on Cluster API vSphere provider official documentation.


### vSphere Cloud Provider Config Secret

```yaml
# -----------------------------------------------------------------------------
# vsphere-config-secret.yaml 
# -----------------------------------------------------------------------------
apiVersion: v1
kind: Secret
metadata:
  name: vsphere-config-secret
  labels:
    cluster.x-k8s.io/cluster-name: "<CLUSTER_NAME>" # Replace with your CLUSTER_NAME
stringData:
  vsphere.conf: |
    global:
      port: 443
      insecureFlag: true # use for selfsigned certificates
      password: "password"
      user: "administrator@vsphere.local"
    vcenter:
      vcenter.example.com:
        datacenters:
        - "datacenter-name"
        server: "vcenter.example.com"
```

### CSI Configuration Secret

```yaml
# -----------------------------------------------------------------------------
# csi-config-secret.yaml 
# -----------------------------------------------------------------------------
apiVersion: v1
kind: Secret
type: Opaque
metadata:
  name: csi-config-secret
stringData:
  csi-vsphere.conf: |
    [Global]
    cluster-id = "<CLUSTER_NAMESPACE>/<CLUSTER_NAME>"
    user = "administrator@vsphere.local"
    password = "password"
    port = 443
    [VirtualCenter "vcenter.local"]
    datacenters = "datacenter-name"
    insecure-flag = true # use for selfsigned certificates
```

### Create Secrets

```bash
kubectl -n <CLUSTER_NAMESPACE> apply -f vsphere-config-secret.yaml 
kubectl -n <CLUSTER_NAMESPACE> apply -f csi-config-secret.yaml
```

## Create Cluster Configuration
Create a copy of [`values.yaml`](charts/capi-kamaji-vsphere-full/values.yaml) file called `<CLUSTER_NAME>-values.yaml` and fill it with your custom configuration.

```yaml
cp values.yaml <CLUSTER_NAME>-values.yaml
vi <CLUSTER_NAME>-values.yaml
```

See the values you can override [here](charts/capi-kamaji-vsphere-full/README.md).


### Install the Cluster

```bash
helm install <CLUSTER_NAME> clastix/capi-kamaji-vsphere-full \
  --namespace <CLUSTER_NAMESPACE> \
  --values <CLUSTER_NAME>-values.yaml \
  --wait --timeout=600s
```

## Post-Installation

After installation, use the commands shown in the NOTES output to monitor your cluster deployment and extract the kubeconfig when ready.

## Uninstallation

Follow this specific order when uninstalling to ensure proper cleanup:

### Uninstall cluster
```bash
helm uninstall <CLUSTER_NAME> \
  --namespace <CLUSTER_NAMESPACE> \
  --wait --timeout=600s
```

**Note**: The uninstall process may take several minutes as it needs to properly drain nodes and delete VMs in vSphere. The two-step process ensures that control plane resources are cleaned up properly before the etcd datastore is removed, preventing stuck finalizers and reconciliation loops.

### Uninstall etcd datastore
```bash
helm uninstall <CLUSTER_NAME>-etcd \
  --namespace <CLUSTER_NAMESPACE> \
  --wait --timeout=600s
```

**Note**: As per Helm design, the uninstall process of `kamaji-etcd` helm release does not remove `PersistentVolumeClaims` resources. You have to do it manually:

```bash
kubectl -n <CLUSTER_NAMESPACE> delete pvc --all
```

Enjoy!
