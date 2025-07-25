# ceph-csi-cephfs

The ceph-csi-cephfs chart adds cephFS volume support to your cluster.

## Install from release repo

Add chart repository to install helm charts from it

```console
helm repo add ceph-csi https://ceph.github.io/csi-charts
```

## Install from local Chart

we need to enter into the directory where all charts are present

```console
cd charts
```

**Note:** charts directory is present in root of the ceph-csi project

### Install Chart

To install the Chart into your Kubernetes cluster(For helm 3.x):

Create the namespace where Helm should install the components with

```bash
kubectl create namespace ceph-csi-cephfs
```

Run the installation

```bash
helm install --namespace "ceph-csi-cephfs" "ceph-csi-cephfs" ceph-csi/ceph-csi-cephfs
```

After installation succeeds, you can get a status of Chart

```bash
helm status --namespace "ceph-csi-cephfs" "ceph-csi-cephfs"
```

### Upgrade Chart

If you want to upgrade your Chart, use the following commands.

```bash
helm repo update ceph-csi
helm upgrade --namespace ceph-csi-cephfs ceph-csi-cephfs ceph-csi/ceph-csi-cephfs
```

For upgrading to a specific version, provide the flag `--version` and the
version.

**Do not forget to include your values**, if they differ from the default values.
We recommend not to use `--reuse-values` in case there are new defaults AND
compare your currently used values with the new default values.

### Enabling encryption support

To enable FSCrypt support, you will need to include the KMS configuration in
`encryptionKMSConfig`.

Here is a `values.yaml` example using a Kubernetes secret (`kubernetes` KMS)

```yaml
encryptionKMSConfig:
    encryptionKMSType: "metadata"
    secretName: "cephfs-encryption-passphrase" # This secret needs to contain the passphrase as the key `encryptionPassphrase`
    secretNamespace: "my-namespace"
storageClass:
    encrypted: true
    encryptionKMSID: kubernetes
```

#### Least privilege secret access

If you use the `metadata` and let RBAC created by the chart, permissions
will be given to access **only** the secret referenced in the
`encryptionKMSConfig`. This is something important to keep in mind, as a
manual change to the config to point to another secret or add further KMS
config will not be authorized. If you wish to give CephCSI a global secret
access to the cluster, you may set `rbac.leastPrivileges` to `false`, and
permissions will be granted globally via a *ClusterRole*.

#### Known Issues Upgrading

- When upgrading to version >=3.7.0, you might encounter an error that the
  CephFS CSI Driver cannot be updated. Please refer to
  [issue](https://github.com/ceph/ceph-csi/issues/3397) for more details.
  This is due to the CSIDriver resource not being updatable. To work around this
  you can delete the CSIDriver object by running:

  ```bash
  kubectl delete csidriver cephfs.csi.ceph.com
  ```

  Then rerun your `helm upgrade` command.

### Delete Chart

If you want to delete your Chart, use this command

```bash
helm uninstall "ceph-csi-cephfs" --namespace "ceph-csi-cephfs"
```

If you want to delete the namespace, use this command

```bash
kubectl delete namespace ceph-csi-cephfs
```

### Configuration

The following table lists the configurable parameters of the ceph-csi-cephfs
charts and their default values.

| Parameter                                      | Description                                                                                                                                          | Default                                            |
| ---------------------------------------------- | ---------------------------------------------------------------------------------------------------------------------------------------------------- | -------------------------------------------------- |
| `rbac.create`                                  | Specifies whether RBAC resources should be created                                                                                                   | `true`                                             |
| `rbac.leastPrivileges`                         | Specifies whether RBAC resources should be created with a restricted scope when supported (only secrets supported currently)                         | `true`                                             |
| `serviceAccounts.nodeplugin.create`            | Specifies whether a nodeplugin ServiceAccount should be created                                                                                      | `true`                                             |
| `serviceAccounts.nodeplugin.name`              | The name of the nodeplugin ServiceAccount to use. If not set and create is true, a name is generated using the fullname                              | ""                                                 |
| `serviceAccounts.provisioner.create`           | Specifies whether a provisioner ServiceAccount should be created                                                                                     | `true`                                             |
| `serviceAccounts.provisioner.name`             | The name of the provisioner ServiceAccount of provisioner to use. If not set and create is true, a name is generated using the fullname              | ""                                                 |
| `csiConfig`                                    | Configuration for the CSI to connect to the cluster                                                                                                  | []                                                 |
| `encryptionKMSConfig`                          | Configuration for the encryption KMS                                                                                                                 | `{}`                                               |
| `commonLabels`                                 | Labels to apply to all resources                                                               | `{}`                                                  |
| `logLevel`                                     | Set logging level for csi containers. Supported values from 0 to 5. 0 for general useful logs, 5 for trace level verbosity.                          | `5`                                                |
| `sidecarLogLevel`                              | Set logging level for csi sidecar containers. Supported values from 0 to 5. 0 for general useful logs, 5 for trace level verbosity.                  | `1`                                                |
| `logSlowOperationInterval`                     | Log slow operations at the specified rate. Operation is considered slow if it outlives its deadline.                                                 | `30s`                                              |
| `nodeplugin.name`                              | Specifies the nodeplugin name                                                                                                                        | `nodeplugin`                                       |
| `nodeplugin.updateStrategy`                    | Specifies the update Strategy. If you are using ceph-fuse client set this value to OnDelete                                                          | `RollingUpdate`                                    |
| `nodeplugin.priorityClassName`                 | Set user created priorityClassName for csi plugin pods. default is system-node-critical which is highest priority                                    | `system-node-critical`                             |
| `nodeplugin.imagePullSecrets`                | Specifies imagePullSecrets for containers                                                                                                        | `[]`                                            |
| `nodeplugin.profiling.enabled`                 | Specifies whether profiling should be enabled                                                                                                        | `false`                                            |
| `nodeplugin.registrar.image.repository`        | Node-Registrar image repository URL                                                                                                                  | `registry.k8s.io/sig-storage/csi-node-driver-registrar` |
| `nodeplugin.registrar.image.tag`               | Image tag                                                                                                                                            | `v2.13.0`                                           |
| `nodeplugin.registrar.image.pullPolicy`        | Image pull policy                                                                                                                                    | `IfNotPresent`                                     |
| `nodeplugin.plugin.image.repository`           | Nodeplugin image repository URL                                                                                                                      | `quay.io/cephcsi/cephcsi`                          |
| `nodeplugin.plugin.image.tag`                  | Image tag                                                                                                                                            | `canary`                                           |
| `nodeplugin.plugin.image.pullPolicy`           | Image pull policy                                                                                                                                    | `IfNotPresent`                                     |
| `nodeplugin.podSecurityContext`                | Specifies pod-level security context.                                                                                                                | `{}`                                               |
| `nodeplugin.annotations`                       | Specifies DaemonSet level annotations.                                                                                                               | `{}`                                               |
| `nodeplugin.podAnnotations`                    | Specifies pod-level annotations.                                                                                                                     | `{}`                                               |
| `nodeplugin.nodeSelector`                      | Kubernetes `nodeSelector` to add to the Daemonset                                                                                                    | `{}`                                               |
| `nodeplugin.tolerations`                       | List of Kubernetes `tolerations` to add to the Daemonset                                                                                             | `{}`                                               |
| `nodeplugin.forcecephkernelclient`             | Set to true to enable Ceph Kernel clients on kernel < 4.17 which support quotas                                                                      | `true`                                             |
| `nodeplugin.kernelmountoptions`                | Comma separated string of mount options accepted by cephfs kernel mounter quotas                                                                      | `""`                                               |
| `nodeplugin.fusemountoptions`                  | Comma separated string of mount options accepted by ceph-fuse mounter quotas                                                                      | `""`                                               |
| `provisioner.name`                             | Specifies the name of provisioner                                                                                                                    | `provisioner`                                      |
| `provisioner.replicaCount`                     | Specifies the replicaCount                                                                                                                           | `3`                                                |
| `provisioner.timeout`                          | GRPC timeout for waiting for creation or deletion of a volume                                                                                        | `60s`                                              |
| `provisioner.clustername`                      | Cluster name to set on the subvolume                                                                                                                 | ""                                                 |
| `provisioner.setmetadata`                      | Set metadata on volume                                                                                                                               | `true`                                             |
| `provisioner.priorityClassName`                | Set user created priorityClassName for csi provisioner pods. Default is `system-cluster-critical` which is less priority than `system-node-critical` | `system-cluster-critical`                          |
| `provisioner.enableHostNetwork`                | Specifies whether hostNetwork is enabled for provisioner pod.                                                                                        | `false`                                            |
| `provisioner.imagePullSecrets`                | Specifies imagePullSecrets for containers                                                                                                        | `[]`                                            |
| `provisioner.profiling.enabled`                | Specifies whether profiling should be enabled                                                                                                        | `false`                                            |
| `provisioner.provisioner.image.repository`     | Specifies the csi-provisioner image repository URL                                                                                                   | `registry.k8s.io/sig-storage/csi-provisioner`      |
| `provisioner.provisioner.image.tag`            | Specifies image tag                                                                                                                                  | `v5.1.0`                                           |
| `provisioner.provisioner.image.pullPolicy`     | Specifies pull policy                                                                                                                                | `IfNotPresent`                                     |
| `provisioner.provisioner.args.httpEndpointPort`    | Specifies http server port for diagnostics, health checks and metrics                                                                                    | `""`                                               |
| `provisioner.provisioner.extraArgs`            | Specifies extra arguments for the provisioner sidecar                                                                                                | `[]`                                               |
| `provisioner.attacher.name`                    | Specifies the name of csi-attacher sidecar                                                                                                           | `attacher`                                         |
| `provisioner.attacher.enabled`                 | Specifies whether attacher sidecar is enabled                                                                                                        | `true`                                             |
| `provisioner.attacher.image.repository`        | Specifies the csi-attacher image repository URL                                                                                                      | `registry.k8s.io/sig-storage/csi-attacher`              |
| `provisioner.attacher.image.tag`               | Specifies image tag                                                                                                                                  | `v4.8.0`                                           |
| `provisioner.attacher.image.pullPolicy`        | Specifies pull policy                                                                                                                                | `IfNotPresent`                                     |
| `provisioner.attacher.args.httpEndpointPort`       | Specifies http server port for diagnostics, health checks and metrics                                                                                    | `""`                                               |
| `provisioner.attacher.extraArgs`               | Specifies extra arguments for the attacher sidecar                                                                                                   | `[]`                                               |
| `provisioner.resizer.name`                     | Specifies the name of csi-resizer sidecar                                                                                                            | `resizer`                                          |
| `provisioner.resizer.enabled`                  | Specifies whether resizer sidecar is enabled                                                                                                         | `true`                                             |
| `provisioner.resizer.image.repository`         | Specifies the csi-resizer image repository URL                                                                                                       | `registry.k8s.io/sig-storage/csi-resizer`          |
| `provisioner.resizer.image.tag`                | Specifies image tag                                                                                                                                  | `v1.13.1`                                           |
| `provisioner.resizer.image.pullPolicy`         | Specifies pull policy                                                                                                                                | `IfNotPresent`                                     |
| `provisioner.resizer.args.httpEndpointPort`        | Specifies http server port for diagnostics, health checks and metrics                                                                                    | `""`                                               |
| `provisioner.resizer.extraArgs`                | Specifies extra arguments for the resizer sidecar                                                                                                    | `[]`                                               |
| `provisioner.snapshotter.image.repository`     | Specifies the csi-snapshotter image repository URL                                                                                                   | `registry.k8s.io/sig-storage/csi-snapshotter`      |
| `provisioner.snapshotter.image.tag`            | Specifies image tag                                                                                                                                  | `v8.2.0`                                           |
| `provisioner.snapshotter.image.pullPolicy`     | Specifies pull policy                                                                                                                                | `IfNotPresent`                                     |
| `provisioner.snapshotter.args.enableVolumeGroupSnapshots`  | enables the creation of volume group snapshots                                                                                           | `false`                                            |
| `provisioner.snapshotter.args.httpEndpointPort`    | Specifies http server port for diagnostics, health checks and metrics                                                                                    | `""`                                               |
| `provisioner.snapshotter.extraArgs`            | Specifies extra arguments for the snapshotter sidecar                                                                                                | `[]`                                               |
| `provisioner.nodeSelector`                     | Specifies the node selector for provisioner deployment                                                                                               | `{}`                                               |
| `provisioner.tolerations`                      | Specifies the tolerations for provisioner deployment                                                                                                 | `{}`                                               |
| `provisioner.affinity`                         | Specifies the affinity for provisioner deployment                                                                                                    | `{}`                                               |
| `provisioner.podSecurityContext`               | Specifies pod-level security context.                                                                                                                | `{}`                                               |
| `provisioner.annotations`                      | Specifies Deployment level annotations.                                                                                                              | `{}`                                               |
| `provisioner.podAnnotations`                   | Specifies pod-level annotations.                                                                                                                     | `{}`                                               |
| `provisionerSocketFile`                        | The filename of the provisioner socket                                                                                                               | `csi-provisioner.sock`                             |
| `pluginSocketFile`                             | The filename of the plugin socket                                                                                                                    | `csi.sock`                                         |
| `readAffinity.enabled` | Enable read affinity for CephFS subvolumes. Recommended to set to true if running kernel 5.8 or newer. | `false` |
| `readAffinity.crushLocationLabels` | Define which node labels to use as CRUSH location. This should correspond to the values set in the CRUSH map. For more information, click [here](https://github.com/ceph/ceph-csi/blob/devel/docs/cephfs/deploy.md#read-affinity-using-crush-locations-for-cephfs-subvolumes)| `[]` |
| `kubeletDir`                                   | Kubelet working directory                                                                                                                            | `/var/lib/kubelet`                                 |
| `driverName`                                   | Name of the csi-driver                                                                                                                               | `cephfs.csi.ceph.com`                              |
| `configMapName`                                | Name of the configmap which contains cluster configuration                                                                                           | `ceph-csi-config`                                  |
| `externallyManagedConfigmap`                   | Specifies the use of an externally provided configmap                                                                                                | `false`                                            |
| `cephConfConfigMapName`                        | Name of the configmap which contains ceph.conf configuration                                                                                           | `ceph-config`                                  |
| `storageClass.create`                          | Specifies whether the StorageClass should be created                                                                                                 | `false`                                            |
| `storageClass.name`                            | Specifies the cephFS StorageClass name                                                                                                               | `csi-cephfs-sc`                                    |
| `storageClass.annotations`                     | Specifies the annotations for the cephFS storageClass                                                                                                | `[]`                                               |
| `storageClass.clusterID`                       | String representing a Ceph cluster to provision storage from                                                                                         | `<cluster-ID>`                                     |
| `storageClass.encrypted`                       | Specifies whether volume should be encrypted. Set it to true if you want to enable encryption                                                        | `""`                                               |
| `storageClass.encryptionKMSID`                 | Specifies the encryption kms id                                                                                                                      | `""`                                               |
| `storageClass.fsName`                          | CephFS filesystem name into which the volume shall be created                                                                                        | `myfs`                                             |
| `storageClass.pool`                            | Ceph pool into which volume data shall be stored                                                                                                     | `""`                                               |
| `storageClass.fuseMountOptions`                | Comma separated string of Ceph-fuse mount options                                                                                                    | `""`                                               |
| `storageclass.kernelMountOptions`              | Comma separated string of CephFS kernel mount options                                                                                                | `""`                                               |
| `storageClass.mounter`                         | The driver can use either ceph-fuse (fuse) or ceph kernelclient (kernel)                                                                             | `""`                                               |
| `storageClass.volumeNamePrefix`                | Prefix to use for naming subvolumes                                                                                                                  | `""`                                               |
| `storageClass.provisionerSecret`               | The secrets have to contain user and/or Ceph admin credentials.                                                                                      | `csi-cephfs-secret`                                |
| `storageClass.provisionerSecretNamespace`      | Specifies the provisioner secret namespace                                                                                                           | `""`                                               |
| `storageClass.controllerExpandSecret`          | Specifies the controller expand secret name                                                                                                          | `csi-cephfs-secret`                                |
| `storageClass.controllerExpandSecretNamespace` | Specifies the controller expand secret namespace                                                                                                     | `""`                                               |
| `storageClass.nodeStageSecret`                 | Specifies the node stage secret name                                                                                                                 | `csi-cephfs-secret`                                |
| `storageClass.nodeStageSecretNamespace`        | Specifies the node stage secret namespace                                                                                                            | `""`                                               |
| `storageClass.reclaimPolicy`                   | Specifies the reclaim policy of the StorageClass                                                                                                     | `Delete`                                           |
| `storageClass.allowVolumeExpansion`            | Specifies whether volume expansion should be allowed                                                                                                 | `true`                                             |
| `storageClass.mountOptions`                    | Specifies the mount options                                                                                                                          | `[]`                                               |
| `volumeSnapshotClass.create`                               | Specifies whether the VolumeSnapshotClass should be created                                                                                                                                                                                                                   | `false`                                                 |
| `volumeSnapshotClass.name`                                 | Specifies the cephFS VolumeSnapshotClass name                                                                                                                                                                                                                                 | `csi-cephfsplugin-snapclass`                            |
| `volumeSnapshotClass.annotations`                          | Specifies the annotations for the cephFS volumeSnapshotClass                                                                                                                                                                                                                  | `[]`                                                    |
| `volumeSnapshotClass.clusterID`                            | String representing a Ceph cluster to provision storage snapshot from                                                                                                                                                                                                         | `<cluster-ID>`                                          |
| `volumeSnapshotClass.snapshotNamePrefix`                   | Prefix to use for naming CephFS snapshots                                                                                                                                                                                                                                     | `""`                                                    |
| `volumeSnapshotClass.snapshotterSecret`                    | The secrets have to contain user and/or Ceph admin credentials.                                                                                                                                                                                                               | `csi-cephfs-secret`                                     |
| `volumeSnapshotClass.snapshotterSecretNamespace`           | Specifies the snapshotter secret namespace                                                                                                                                                                                                                                    | `""`                                                    |
| `volumeSnapshotClass.deletionPolicy`                       | Specifies the deletion policy of the VolumeSnapshotClass                                                                                                                                                                                                                      | `Delete`                                                |
| `volumeGroupSnapshotClass.name`                            | Specifies the cephFS VolumeGroupSnapshotClass name                                                                                                                                                                                                                            | `csi-cephfsplugin-groupsnapclass`                       |
| `volumeGroupSnapshotClass.annotations`                     | Specifies the annotations for the cephFS volumeGroupSnapshotClass                                                                                                                                                                                                             | `[]`                                                    |
| `volumeGroupSnapshotClass.clusterID`                       | String representing a Ceph cluster to provision storage group snapshot from                                                                                                                                                                                                   | `<cluster-ID>`                                          |
| `volumeGroupSnapshotClass.fsName`                          | CephFS filesystem name into which the volume shall be created                                                                                                                                                                                                                 | `myfs`                                                  |
| `volumeGroupSnapshotClass.volumeGroupNamePrefix`           | Prefix to use for naming CephFS volumeGroups                                                                                                                                                                                                                                  | `""`                                                    |
| `volumeGroupSnapshotClass.groupSnapshotterSecret`          | The secrets have to contain user and/or Ceph admin credentials.                                                                                                                                                                                                               | `csi-cephfs-secret`                                     |
| `volumeGroupSnapshotClass.groupSnapshotterSecretNamespace` | Specifies the groupSnapshotter secret namespace                                                                                                                                                                                                                               | `""`                                                    |
| `volumeGroupSnapshotClass.deletionPolicy`                  | Specifies the deletion policy of the VolumeGroupSnapshotClass                                                                                                                                                                                                                 | `Delete`                                                |
| `secret.create`                                | Specifies whether the secret should be created                                                                                                       | `false`                                            |
| `secret.name`                                  | Specifies the cephFS secret name                                                                                                                     | `csi-cephfs-secret`                                |
| `secret.userID`                                | Specifies the user ID of the cephFS secret.                                                                                                          | `""`                                               |
| `secret.userKey`                               | Specifies the key that corresponds to the userID.                                                                                                    | `<Ceph auth key corresponding to ID above>`        |
| `selinuxMount`                                | Mount the host /etc/selinux inside pods to support selinux-enabled filesystems                                                                                                      | `true`                                            |
| `CSIDriver.fsGroupPolicy` | Specifies the fsGroupPolicy for the CSI driver object | `File` |
| `CSIDriver.seLinuxMount` | Specify for efficient SELinux volume relabeling | `true` |
| `instanceID`                                   | Unique ID distinguishing this instance of Ceph CSI among other instances, when sharing Ceph clusters across CSI instances for provisioning. | ` ` |
| `radosNamespaceCephFS`                         | CephFS RadosNamespace used to store CSI specific objects and keys. | ` ` |

### Command Line

You can pass the settings with helm command line parameters.
Specify each parameter using the --set key=value argument to helm install.
For Example:

```bash
helm install --set configMapName=ceph-csi-config
```
