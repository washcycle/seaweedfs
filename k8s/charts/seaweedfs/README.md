# SeaweedFS Chart

## Getting Started

### Add the helm repo

`helm repo add seaweedfs https://seaweedfs.github.io/seaweedfs/helm`

### Install the helm chart

`helm install seaweedfs seaweedfs/seaweedfs`

### (Recommended) Provide `values.yaml`

`helm install --values=values.yaml seaweedfs seaweedfs/seaweedfs`

## Info:
* By default filer is using leveldb - if HA is desired for the filer a database backend is recommended.
* master/filer/volume are stateful sets with anti-affinity on the hostname so your deployment will be HA.
* mysql user/password are created in a k8s secret (secret-seaweedfs-db.yaml) and injected to the filer with ENV.
* secent envs can be referenced in the helm chart for the filer
* cert config exists and can be enabled, but not been tested, requires cert-manager to be installed.

## Prerequisites
### Database

leveldb is the default database this only supports one filer replica.

To have multiple filers a external datastore is recommened.

Such as MySQL-compatible database, as specified in the `values.yaml` at `filer.extraEnvironmentVars`. 
This database should be pre-configured and initialized by running:
```sql
CREATE TABLE IF NOT EXISTS `filemeta` (
  `dirhash`   BIGINT NOT NULL       COMMENT 'first 64 bits of MD5 hash value of directory field',
  `name`      VARCHAR(766) NOT NULL COMMENT 'directory or file name',
  `directory` TEXT NOT NULL         COMMENT 'full path to parent directory',
  `meta`      LONGBLOB,
  PRIMARY KEY (`dirhash`, `name`)
) DEFAULT CHARSET=utf8mb4 COLLATE=utf8mb4_bin;
```

Alternative database can also be configured (e.g. leveldb, postgres) following the instructions at `filer.extraEnvironmentVars`.

### Node Labels
Kubernetes nodes can have labels which help to define which node(Host) will run which pod:

Here is an example:
* s3/filer/master needs the label **sw-backend=true**
* volume need the label **sw-volume=true**

to label a node to be able to run all pod types in k8s:
```
kubectl label node YOUR_NODE_NAME sw-volume=true,sw-backend=true
```

on production k8s deployment you will want each pod to have a different host,
especially the volume server and the masters, all pods (master/volume/filer)
should have anti-affinity rules to disallow running multiple component pods  on the same host.

If you still want to run multiple pods of the same component (master/volume/filer) on the same host please set/update the corresponding affinity rule in values.yaml to an empty one:

```affinity: ""```

## PVC - storage class ###

On the volume stateful set added support for k8s PVC, currently example
with the simple local-path-provisioner from Rancher (comes included with k3d / k3s)
https://github.com/rancher/local-path-provisioner

you can use ANY storage class you like, just update the correct storage-class
for your deployment.

## current instances config (AIO):

1 instance for each type (master/filer+s3/volume)

You can update the replicas count for each node type in values.yaml,
need to add more nodes with the corresponding labels if applicable.

Most of the configuration are available through values.yaml any pull requests to expand functionality or usability are greatly appreciated. Any pull request must pass [chart-testing](https://github.com/helm/chart-testing).

## Parameters

### Global Parameters

| Key | Type | Description |
| --- | --- | --- |
| global.registry | string | Docker registry to use for the SeaweedFS image. |
| global.repository | string | Repository within the registry to use for the SeaweedFS image. |
| global.imageName | string | Name of the SeaweedFS image to use. |
| global.imagePullPolicy | string | Image pull policy for the SeaweedFS image. |
| global.imagePullSecrets | string | Image pull secrets for the SeaweedFS image. |
| global.restartPolicy | string | Restart policy for the SeaweedFS pods. |
| global.loggingLevel | int | Logging level for the SeaweedFS pods. |
| global.enableSecurity | bool | Whether to enable security for the SeaweedFS cluster. |
| global.monitoring.enabled | bool | Whether to enable monitoring for the SeaweedFS cluster. |
| global.monitoring.gatewayHost | string | Hostname of the monitoring gateway for the SeaweedFS cluster. |
| global.monitoring.gatewayPort | string | Port of the monitoring gateway for the SeaweedFS cluster. |
| global.enableReplication | bool | Whether to enable replication for the SeaweedFS cluster. |
| global.replicationPlacment | string | Replica placement configuration for the SeaweedFS cluster. |
| global.extraEnvironmentVars.WEED_CLUSTER_DEFAULT | string | Default value for WEED_CLUSTER_DEFAULT environment variable. |
| global.extraEnvironmentVars.WEED_CLUSTER_SW_MASTER | string | Value for WEED_CLUSTER_SW_MASTER environment variable. |
| global.extraEnvironmentVars.WEED_CLUSTER_SW_FILER | string | Value for WEED_CLUSTER_SW_FILER environment variable. |
| image.registry | string | Docker registry to use for the SeaweedFS image. |
| image.repository | string | Repository within the registry to use for the SeaweedFS image. |

### Master Parameters

| Key | Type | Description |
| --- | --- | --- |
| master.enabled | boolean | Determines whether the master is enabled. |
| master.repository | string | The repository of the master's image. |
| master.imageName | string | The name of the master's image. |
| master.imageTag | string | The tag of the master's image. |
| master.imageOverride | string | Overrides the default Docker image if specified. |
| master.restartPolicy | string | The restart policy for the master pod. |
| master.replicas | integer | The number of replicas to run. |
| master.port | integer | The port on which to serve the HTTP API. |
| master.grpcPort | integer | The port on which to serve the gRPC API. |
| master.metricsPort | integer | The port on which to serve the Prometheus metrics. |
| master.ipBind | string | The IP address to bind the HTTP API to. |
| master.volumePreallocate | boolean | Determines whether to preallocate disk space. |
| master.volumeSizeLimitMB | integer | The limit of disk space to use. |
| master.loggingOverrideLevel | string | Overrides the default logging level. |
| master.pulseSeconds | string | The number of seconds between heartbeats. |
| master.garbageThreshold | string | The threshold to vacuum and reclaim spaces. |
| master.metricsIntervalSec | integer | The Prometheus push interval in seconds. |
| master.defaultReplication | string | The default replication type for new volumes. |
| master.disableHttp | boolean | Determines whether to disable HTTP requests. |
| master.data.type | string | The type of data storage. |
| master.data.size | string | The size of the data storage. |
| master.data.storageClass | string | The storage class for the data storage. |
| master.data.hostPathPrefix | string | The host path prefix for the data storage. |
| master.logs.type | string | The type of log storage. |
| master.logs.size | string | The size of the log storage. |
| master.logs.storageClass | string | The storage class for the log storage. |
| master.logs.hostPathPrefix | string | The host path prefix for the log storage. |
| master.initContainers | array | Additional init containers for the master. |
| master.extraVolumes | array | Additional volumes for the master. |
| master.extraVolumeMounts | array | Additional volume mounts for the master. |
| master.resources | string | Resource requests, limits, etc. for the master cluster placement. |
| master.updatePartition | integer | Used to control a careful rolling update of SeaweedFS masters. |
| master.affinity | object | Affinity settings for the master pods. |
| master.tolerations | string | Toleration settings for master pods. |
| master.nodeSelector | string | Node selector labels for master pod assignment. |
| master.priorityClassName | string | Used to assign priority to master pods. |
| master.ingress.enabled | boolean | Determines whether the ingress is enabled. |
| master.ingress.className | string | The class name of the ingress controller. |
| master.ingress.annotations | object | Annotations for the ingress. |
| master.extraEnvironmentVars | object | Additional environment variables for the master. |

### Volume Parameters

| Key | Type | Description |
| --- | ---- | ----------- |
| volume.enabled | boolean | Enable/disable the volume component |
| volume.repository | string | Docker registry for volume component |
| volume.imageName | string | Docker image name for volume component |
| volume.imageTag | string | Docker image tag for volume component |
| volume.imageOverride | string | Override the Docker image used for volume component |
| volume.restartPolicy | string | The restart policy for volume component |
| volume.port | int | Port number for volume component |
| volume.grpcPort | int | Port number for gRPC server |
| volume.metricsPort | int | Port number for metrics server |
| volume.ipBind | string | IP address to bind volume component |
| volume.replicas | int | Number of replicas for volume component |
| volume.loggingOverrideLevel | string | Override logging level for volume component |
| volume.pulseSeconds | string | Number of seconds between heartbeats |
| volume.index | string | Choose mode for memory~performance balance |
| volume.fileSizeLimitMB | string | Limit file size to avoid out of memory |
| volume.minFreeSpacePercent | int | Minimum free disk space in percents |
| volume.data.type | string | Type of data volume |
| volume.data.size | string | Size of data volume |
| volume.data.storageClass | string | Storage class for data volume |
| volume.data.hostPathPrefix | string | Prefix for data volume host path |
| volume.idx.type | string | Type of index volume |
| volume.idx.size | string | Size of index volume |
| volume.idx.storageClass | string | Storage class for index volume |
| volume.idx.hostPathPrefix | string | Prefix for index volume host path |
| volume.logs.type | string | Type of logs volume |
| volume.logs.size | string | Size of logs volume |
| volume.logs.storageClass | string | Storage class for logs volume |
| volume.logs.hostPathPrefix | string | Prefix for logs volume host path |
| volume.compactionMBps | string | Limit background compaction or copying speed in MB/s |
| volume.dir | string | Directories to store data files |
| volume.dir_idx | string | Directories to store index files |
| volume.maxVolumes | string | Maximum numbers of volumes |
| volume.rack | string | Rack name for volume server |
| volume.dataCenter | string | Data center name for volume server |
| volume.readMode | string | Redirect moved or non-local volumes |
| volume.whiteList | string | Comma separated IP addresses having write permission |
| volume.imagesFixOrientation | boolean | Adjust jpg orientation when uploading |
| volume.initContainers | list | List of init containers to run before volume server container |
| volume.extraVolumes | list | Extra volumes to be mounted in volume server container |
| volume.extraVolumeMounts | list | Extra volume mounts for volume server container |
| volume.affinity | object | Affinity settings for volume server pod |
| volume.resources | object | Resource requests and limits for volume server pod |
| volume.tolerations | string | Toleration settings for volume server pod |
| volume.nodeSelector | string | Node selector labels for volume server pod assignment |
| volume.priorityClassName | string | Priority class name for volume server pod |


### Filer Parameters

| Key                       | Type    | Description                                                                                                            |
|---------------------------|---------|------------------------------------------------------------------------------------------------------------------------|
| filer.enabled             | boolean | Whether to enable the filer                                                                                            |
| filer.repository          | string  | The Docker repository for the filer image                                                                              |
| filer.imageName           | string  | The Docker image name for the filer                                                                                     |
| filer.imageTag            | string  | The Docker image tag for the filer                                                                                      |
| filer.imageOverride       | string  | Override the Docker image with a different one                                                                          |
| filer.restartPolicy       | string  | Restart policy for the filer container                                                                                  |
| filer.replicas            | integer | Number of filer replicas to run                                                                                         |
| filer.port                | integer | Port number to use for the filer HTTP service                                                                           |
| filer.grpcPort            | integer | Port number to use for the filer gRPC service                                                                           |
| filer.metricsPort         | integer | Port number to use for the filer metrics service                                                                        |
| filer.loggingOverrideLevel| string  | Override log levels for certain components of SeaweedFS, such as Filers or Masters                                    |
| filer.defaultReplicaPlacement | string | Replication type for newly created collections |
| filer.disableDirListing   | boolean | Turn off directory listing                                                                                              |
| filer.maxMB               | string  | Split files larger than the limit (default 32)                                                                           |
| filer.encryptVolumeData   | boolean | Encrypt data on volume servers                                                                                          |
| filer.redirectOnRead      | boolean | Whether to proxy or redirect to volume server during file GET request                                                   |
| filer.dirListLimit        | integer | Limit size of subdirectory listing                                                                                      |
| filer.disableHttp         | boolean | Whether to disable HTTP requests, only allowing gRPC operations                                                          |
| filer.enablePVC           | boolean | Whether to enable PVC for filer for data persistence                                                                     |
| filer.storage             | string  | Disk size of the attached volume                                                                                        |
| filer.storageClass        | string  | Storage class for PVC, which defaults to the Kube cluster's default                                                       |
| filer.data.type           | string  | Type of data volume to use: hostPath or persistentVolumeClaim                                                            |
| filer.data.size           | string  | Size of the data volume                                                                                                 |
| filer.data.storageClass   | string  | Storage class to use for data volume                                                                                    |
| filer.data.hostPathPrefix | string  | Prefix for the hostPath when using hostPath type for data volume                                                          |
| filer.logs.type           | string  | Type of logs volume to use: hostPath or persistentVolumeClaim                                                            |
| filer.logs.size           | string  | Size of the logs volume                                                                                                 |
| filer.logs.storageClass   | string  | Storage class to use for logs volume                                                                                    |
| filer.logs.hostPathPrefix | string  | Prefix for the hostPath when using hostPath type for logs volume                                                          |
| filer.initContainers      | list    | List of initContainers to run in the filer Pod                                                                          |
| filer.extraVolumes        | list    | List of additional volumes to mount in the filer Pod                                                                     |
| filer.extraVolumeMounts   | list    | List of additional mounts to attach to the volumes in the filer Pod

### S3 Parameters

| Key                     | Type     | Description                                                                                                         |
| -----------------------| --------| ------------------------------------------------------------------------------------------------------------------- |
| s3.enabled             | boolean  | Enable/disable S3 service                                                                                           |
| s3.repository          | string   | Docker repository for S3 image                                                                                      |
| s3.imageName           | string   | Name of S3 Docker image to use                                                                                      |
| s3.imageTag            | string   | Tag of S3 Docker image to use                                                                                       |
| s3.restartPolicy       | string   | Kubernetes pod restart policy                                                                                       |
| s3.replicas            | integer  | Number of S3 replicas to deploy                                                                                     |
| s3.bindAddress         | string   | Bind address for S3 service                                                                                         |
| s3.port                | integer  | Port for S3 service                                                                                                |
| s3.metricsPort         | integer  | Port for S3 service metrics                                                                                         |
| s3.loggingOverrideLevel| string   | Override logging level for S3 service                                                                               |
| s3.allowEmptyFolder    | boolean  | Allow empty folders in S3 service                                                                                   |
| s3.enableAuth          | boolean  | Enable user authentication and permission control for S3 service                                                    |
| s3.skipAuthSecretCreation | boolean | Skip creating authentication secret                                                                                 |
| s3.auditLogConfig      | object   | Configuration for audit logging (empty object)                                                                      |
| s3.domainName          | string   | Suffix of the host name for S3 service (bucket.domainName)                                                           |
| s3.initContainers      | string   | Kubernetes init containers for S3 service (empty string)                                                             |
| s3.extraVolumes        | string   | Additional Kubernetes volumes to mount for S3 service (empty string)                                                |
| s3.extraVolumeMounts   | string   | Additional Kubernetes volume mounts for S3 service (empty string)                                                   |
| s3.resources           | object   | Kubernetes resources requests, limits, etc. for S3 pods (empty object)                                              |
| s3.tolerations         | string   | Kubernetes toleration settings for S3 pods (empty string)                                                            |
| s3.nodeSelector        | object   | Kubernetes nodeSelector labels for S3 pods (empty object)                                                            |
| s3.priorityClassName   | string   | Kubernetes priorityClassName for S3 pods                                                                            |
| s3.logs.type           | string   | Type of logging for S3 service (hostPath)                                                                            |
| s3.logs.size           | string   | Maximum size of logs for S3 service (empty string)                                                                   |
| s3.logs.storageClass   | string   | Kubernetes storageClass for S3 logs                                                                                  |
| s3.logs.hostPathPrefix | string   | Host path prefix for S3 logs                                                                                        |

### Certificate

| Key | Type | Description |
| --- | --- | --- |
| certificates.commonName | string | The common name (CN) to use for the SeaweedFS CA. |
| certificates.ipAddresses | list | A list of IP addresses to include in the certificate. |
| certificates.keyAlgorithm | string | The algorithm to use for the private key (e.g. rsa, ecdsa). |
| certificates.keySize | int | The size (in bits) of the private key. |
| certificates.duration | string | The duration for which the certificate will be valid. |
| certificates.renewBefore | string | The amount of time before the certificate expiration to start the renewal process. |
