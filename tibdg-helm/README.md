    Copyright (c) 2018 TIBCO
    Software Inc.  All Rights Reserved. Confidential & Proprietary.  For more
    information, please contact: TIBCO Software Inc., Palo Alto, California, USA

# TIBCO ActiveSpaces Helm Chart

## Introduction

TIBCO ActiveSpaces (abbreviated tibdg for 'TIBCO Data Grid') is a distributed, fault-tolerant, key-value store. [Helm](https://helm.sh) is a package manager for Kubernetes.

## Prerequisites

- Kubernetes 1.9+
- PV (Persistent Volume) provisioner support in the underlying infrastructure
- Make sure the ActiveSpaces Docker images are accessible from the cluster. Either the images must be loaded into a common private registry as outlined [here](https://kubernetes.io/docs/concepts/containers/images/), or the images can be loaded on each node in the cluster. Ensure all of the requisite Docker images from the product install package are loaded:
        
        docker load -i as-operations-3.5.0.dockerimage.xz
        docker load -i as-tibdg-3.5.0.dockerimage.xz
        docker load -i as-tibdgnode-3.5.0.dockerimage.xz
        docker load -i as-tibdgkeeper-3.5.0.dockerimage.xz
        docker load -i as-tibdgproxy-3.5.0.dockerimage.xz
        docker load -i as-tibdgadmind-3.5.0.dockerimage.xz

    Additionally, the FTL realmserver image is required. The minimum version required for this data grid is: 5.3.1

        docker load -i ftl-tibrealmserver-5.3.1.dockerimage.xz

    The images referenced by the chart can be configured, e.g. if a custom registry path is required.

## Installing the Chart

To install the chart with the release name `tibdg-helm`, run this command from the directory containing this README:

    helm install --name=tibdg-helm .

This command deploys ActiveSpaces on the Kubernetes cluster with the default configuration. The [configuration](#configuration) section lists the parameters that can be configured during installation.

To view the generated Kubernetes manifest without actually deploying into the cluster, the `--dry-run` and `--debug` options can be used:

    helm install --name=tibdg-helm --debug --dry-run .

## Uninstalling the Chart

To uninstall/delete the `tibdg-helm` deployment:

    helm delete --purge tibdg-helm

This command removes all the Kubernetes components associated with the chart and deletes the release.

## Configuration

The following table lists the configurable parameters of the ActiveSpaces chart
and their default values. More detailed documentation is available in the
comments of [values.yaml](values.yaml).

| Parameter                           | Description | Default      |
| ----------------------------------- | ----------- | ------------ |
| `asVersion`                         | The ActiveSpaces verion to use | `3.5.0` |
| `ftlVersion`                        | The FTL version to use | `5.3.1` |
| `gridCreationOptions`               | Custom grid creation options | `null` |
| `gridName`                          | Custom grid name. Default used if null | `null` |
| `tibdgConfig`                       | Additional tibdg commands to be run at grid initialization | `table create t1 key long`<br>`column create t1 value string` |
| `externalAccessMode`                | The mode of external access, see [Enabling Access Beyond the Cluster](#enabling-access-beyond-the-cluster) | `null` |
| `realmserver.image`                 | The Docker image for the realmserver container | `ftl-tibrealmserver` |
| `realmserver.imageTag`              | The realmserver image tag. By default `ftlVersion` is used | `null` |
| `realmserver.storageCapacity`       | The storage capacity of the realmserver persistent volume | `1Gi` |
| `realmserver.ports.http`            | The http api port of the realmserver | `30080` |
| `realmserver.ports.ftl`             | The FTL port of the realmserver | `30083` |
| `realmserver.ports.gui`             | The http gui port of the realmserver | `30085` |
| `realmserver.storageClass`          | The StorageClass for the realmserver PersistentValueClaim | `null` |
| `realmserver.loadBalancerIP`        | The realmserver LoadBalancer ip if external access mode is enabled. If null will use the value of tibdgproxy.advertisedIP | `null` |
| `realmserver.resources`             | realmserver CPU/Memory resource requests/limits | `{}` |
| `tibdgadmind.image`                 | The Docker image for the tibdgadmind container | `as-tibdgadmind` |
| `tibdgadmind.imageTag`              | The tibdgadmind image tag. By default `asVersion` is used | `null` |
| `tibdgadmind.ports.admind`          | The http api port of the tibdgadmind | `30081` |
| `tibdgadmind.resources`             | tibdgadmind CPU/Memory resource requests/limits | `{}` |
| `tibdg.image`                       | The Docker image for the tibdg container | `as-tibdg` |
| `tibdg.imageTag`                    | The tibdg image tag. By default `asVersion` is used | `null` |
| `tibdgnode.image`                   | The Docker image for the tibdgnode container | `as-tibdgnode` |
| `tibdgnode.imageTag`                | The tibdgnode image tag. By default `asVersion` is used | `null` |
| `tibdgnode.copysetCount`            | The number of copysets to partion data amongst | `1` |
| `tibdgnode.copysetSize`             | The number of replicas in each copyset | `2` |
| `tibdgnode.storageCapacity`         | The storage capacity of the tibdgnode persistent volume | `10Gi` |
| `tibdgnode.storageClass`            | The StorageClass for the tibdgnode PersistentValueClaim | `null` |
| `tibdgnode.resources`               | tibdgnode CPU/Memory resource requests/limits | `{}` |
| `tibdgkeeper.image`                 | The Docker image for the tibdgkeeper container | `as-tibdgkeeper` |
| `tibdgkeeper.imageTag`              | The tibdgkeeper image tag. By default `asVersion` is used | `null` |
| `tibdgkeeper.count`                 | The number of state keeper instances to run. Valid values are 1 or 3 | `3` |
| `tibdgkeeper.storageCapacity`       | The storage capacity of the tibdgkeeper persistent volume | `1Gi` |
| `tibdgkeeper.storageClass`          | The StorageClass for the tibdgkeeper PersistentValueClaim | `null` |
| `tibdgkeeper.resources`             | tibdgkeeper CPU/Memory resource requests/limits | `{}` |
| `tibdgproxy.image`                  | The Docker image for the tibdgproxy container | `as-tibdgproxy` |
| `tibdgproxy.imageTag`               | The tibdgproxy image tag. By default `asVersion` is used | `null` |
| `tibdgproxy.count`                  | The number of proxy instances to run | `2` |
| `tibdgproxy.externalBasePort`       | If external access is enabled, each proxy instance is assigned a unique port from externalBasePort up to externalBasePort + count - 1 | `30100` |
| `tibdgproxy.advertisedIPs`          | If external access is enabled, the ip address each proxy advertises. `$(NODE_IP)` is the Kubernetes node ip address | `$(NODE_IP)` |
| `tibdgproxy.listenPort`             | The internal proxy listen port | `8888` |
| `tibdgproxy.resources`              | tibdgproxy CPU/Memory resource requests/limits | `{}` |
| `pod.terminationGracePeriodSeconds` | The Kubernetes terminationGracePeriodSeconds to set for each pod. Leave `null` for default | `null` |

Specify each parameter using the `--set key=value[,key=value]` argument to `helm install`. For example,

    helm install --name tibdg-helm --set tibdgnode.copysetCount=5 .

The above command starts a data grid with 5 copysets.

Alternatively, a YAML file that specifies the values for the parameters can be provided while installing the chart. For example,

    helm install --name tibdg-helm -f values.yaml .

> **Tip**: You can use the default [values.yaml](values.yaml)

## Persistence

The realmserver, statekeeper, and node processes of the data grid use the `/data` path of the container to store data and configuration. The chart mounts a [Persistent Volume](https://kubernetes.io/docs/concepts/storage/persistent-volumes/) at this location. By default, the Persistent Volume is provisioned using the default dynamic provisioner of the cluster. You can configure alternative provisioner by specifying the `realmserver.storageClass` / `tibdgkeeper.storageClass` / `tibdgnode.storageClass` options.

The realmserver and statekeepers primarily store configuration data. The bulk of the storage is consumed by the node processes. Ensure that the `storageCapacity` options are sized appropriately.

## Enabling Access Beyond the Cluster

By default, the data grid is accessible from within the Kubernetes cluster only. To enable external access, the  `externalAccessMode` option must be used. 3 options are available:
- `null`: the default, no external access enabled.
- `NodePort`: This enables [NodePort](https://kubernetes.io/docs/concepts/services-networking/service/#type-nodeport) services for the realmserver and proxy pods. Under this mode, clients can access the cluster by directly communicating with the Kubernetes cluster nodes on the configured port values (`tibdgproxy.externalBasePort`, `realmserver.ports.*`). N.B. the configured ports must fall within the cluster's NodePort range (default: 30000-32767).
- `LoadBalancer`: This enables [LoadBalancer](https://kubernetes.io/docs/concepts/services-networking/service/#type-loadbalancer) services for the realmserver and proxy pods. The behavior of the LoadBalancer service is cluster dependent. Typically the cluster creates a new LoadBalancer instance native to its implementation, which is used to proxy the internal service. One caveat with this method is that the IP address of the LoadBalancer must be configured (`tibdgproxy.advertisedIPs`) prior to starting the data grid. This is because each tibdgproxy advertises the 'external' IP address to potential clients.

## Accessing the Data Grid

To verify the data grid is running the `tibdg` tool can be used to check status. The `tibdg` tool is part of the `as-tibdg:3.5.0` image and can be run in the cluster via the following command:

    kubectl run -it --rm --restart=Never --image=as-tibdg:3.5.0 tibdg -- \
        -r http://realmserver:30080 status

The `operations` sample can also be run to get and put data and run other operations:

    kubectl run -it --rm --restart=Never --image=as-operations:3.5.0 operations -- \
        -r http://realmserver:30080

The chart creates a DNS entry for the realmserver at domain `realmserver`. The default realmserver http port is 30080. Therefore any applications needing to access the data grid in the cluster can use the realmserver address http://realmserver:30080.

If external access is enabled, the realmserver address becomes http://&lt;node or loadbalancer ip address&gt;:30080
