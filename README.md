# Container Storage Interface (CSI) driver for VMware Cloud Director Named Independent Disks
This repository contains the source code and build methods to build a Kubernetes CSI driver that helps provision [VMware Cloud Director Named Independent Disks](https://docs.vmware.com/en/VMware-Cloud-Director/10.3/VMware-Cloud-Director-Tenant-Portal-Guide/GUID-8F8BFCD3-071A-4E45-BAC0-A9B78F2C19CE.html) as a storage solution for Kubernetes Applications. This uses VMware Cloud Director API for functionality and hence needs an appropriate VMware Cloud Director Installation. This CSI driver will help enable common scenarios with persistent volumes and stateful-sets using VMware Cloud Director Shareable Named Disks.


This extension is intended to be installed into a Kubernetes cluster installed with [VMware Cloud Director](https://www.vmware.com/products/cloud-director.html) as a Cloud Provider, by a user that has the rights as described in the sections below.

**cloud-director-named-disk-csi-driver** is distributed as a container image hosted at [Distribution Harbor](https://projects.registry.vmware.com) as `projects.registry.vmware.com/vmware-cloud-director/cloud-director-named-disk-csi-driver:<CSI version>`

This driver is in a GA state and will be supported in production.

Note: This driver is not impacted by the Apache Log4j open source component vulnerability.

## CSI Feature matrix
|         Feature          | Support Scope                                                                                                                                                                                  |
|:------------------------:|:-----------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------|
|       Storage Type       | <ul>Independent Shareable Named Disks of VCD</ul>                                                                                                                                              |
|       Provisioning       | <ul><li>Static Provisioning</li><li>Dynamic Provisioning</li></ul>                                                                                                                             |
|       Access Modes       | <ul><li>ReadOnlyMany</li><li>ReadWriteOnce</li></ul>                                                                                                                                           |
|          Volume          | <ul>Block</ul>                                                                                                                                                                                 |
|        VolumeMode        | <ul><li>FileSystem</li><li>Block</li></ul>                                                                                                                                                     |
| Volume Expansion Support | <ul><li>OFFLINE</li><li>ONLINE</li></ul>                                                                                                                                                       |
|         Topology         | <ul><li>Static Provisioning: reuses VCD topology capabilities</li><li>Dynamic Provisioning: places disk in the OVDC of the `ClusterAdminUser` based on the StorageProfile specified.</li></ul> |

## Terminology
1. VCD: VMware Cloud Director
2. ClusterAdminRole: This is the role that has enough rights to create and administer a Kubernetes Cluster in VCD. This role can be created by cloning the [vApp Author Role](https://docs.vmware.com/en/VMware-Cloud-Director/10.3/VMware-Cloud-Director-Tenant-Portal-Guide/GUID-BC504F6B-3D38-4F25-AACF-ED584063754F.html) and then adding the following rights (details on adding the rights below can be found in the [CSE docs](https://github.com/rocknes/container-service-extension/blob/cse_3_1_docs/docs/cse3_1/RBAC.md#additional-required-rights)):
   1. Full Control: VMWARE:CAPVCDCLUSTER
   2. Edit: VMWARE:CAPVCDCLUSTER
   3. View: VMWARE:CAPVCDCLUSTER
3. ClusterAdminUser: For CSI functionality, there needs to be a set of additional rights added to the `ClusterAdminRole` as described in the "Additional Rights for CSI" section below. The Kubernetes Cluster needs to be **created** by a user belonging to this **enhanced** `ClusterAdminRole`. For convenience, let us term this user as the `ClusterAdminUser`.

## VMware Cloud Director Configuration
In this section, we assume that the Kubernetes cluster is created using the Container Service Extension 4.0. However, that is not a mandatory requirement.

### Additional Rights for CSI
The `ClusterAdminUser` should have view access to the vApp containing the Kubernetes cluster. Since the `ClusterAdminUser` itself creates the cluster, it will have this access by default.
This `ClusterAdminUser` needs to be created from a `ClusterAdminRole` with the following additional rights:
1. Access Control =>
   1. User => Manage user's own API TOKEN
2. Organization VDC => Create a Shared Disk

## Troubleshooting
### Log VCD requests and responses
Execute the following command to log HTTP requests to VCD and HTTP responses from VCD -
```shell
kubectl set env -n kube-system StatefulSet/csi-vcd-controllerplugin -c vcd-csi-plugin GOVCD_LOG_ON_SCREEN=true -oyaml
kubectl set env -n kube-system DaemonSet/csi-vcd-nodeplugin -c vcd-csi-plugin GOVCD_LOG_ON_SCREEN=true -oyaml
```
Once the above command is executed, CSI containers will start logging the HTTP requests and HTTP responses made via go-vcloud-director SDK.
The container logs can be obtained using the command `kubectl logs -n kube-system <CSI pod name>`

To stop logging the HTTP requests and responses from VCD, the following command can be executed -
```shell
kubectl set env -n kube-system Deployment/csi-vcd-controllerplugin -c vcd-csi-plugin GOVCD_LOG_ON_SCREEN-
kubectl set env -n kube-system DaemonSet/csi-vcd-nodeplugin -c vcd-csi-plugin GOVCD_LOG_ON_SCREEN-
```

**NOTE: Please make sure to collect the logs before and after enabling the wire log. The above commands update the CSI controller Deployment and CSI node-plugin DaemonSet, which creates a new CSI pods. The logs present in the old pods will be lost.**

### Upgrade CSI

To perform an upgrade of the Container Storage Interface (CSI) from versions v1.2.0, v1.2.1, v1.3.0, v1.3.1, and v1.3.2, it is recommended to follow the following steps:
1. Remove the current StatefulSet:
```shell
kubectl delete statefulset -n kube-system csi-vcd-controllerplugin
```
2. Apply the CSI 1.4 Controller CRS:
```shell
kubectl apply -f https://github.com/vmware/cloud-director-named-disk-csi-driver/blob/1.4.z/manifests/csi-controller-crs.yaml
```
**NOTE:**

1. These steps ensure a successful upgrade of CSI to the latest version (v1.4.0) and guarantee that the new CSI Deployment is properly installed within the Kubernetes environment.
2. it is recommended not to manually delete any Persistent Volumes (PVs) or Persistent Volume Claims (PVCs) associated with a StatefulSet.

## Contributing
Please see [CONTRIBUTING.md](CONTRIBUTING.md) for instructions on how to contribute.


## License
[Apache-2.0](LICENSE.txt)
