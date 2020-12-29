#  Check Point Cloudguard CSPM Agents for Openshift

## Introduction

This chart creates a single resource management Pod that scans the cluster's resources (Nodes, Images, Pods, Namespaces, Services, PSP, Network Policy, and Ingress) and uploads them to [Check Point ClougGuard CSPM](https://secure.dome9.com/).
Check Point ClougGuard CSPM provides Compliance, Vulnerability Assessment, Visibility, Monitoring and Threat Hunting capabilities.

## Prerequisites

- Openshift 4.x+
- Helm 3.0+
- A Check Point ClougGuard CSPM account and API key

## Configure CSPM

Go to [Check Point ClougGuard CSPM](https://secure.dome9.com/). <br>

Select Assets > Environment > <b>Add New</b> <br>

Walk through the wizard. When given the helm string, <b>note the cluster ID.</b> We will use this later. Finish the wizard.

## Configuring the Openshift Cluster

Clone the repository
```
git clone https://github.com/metalstormbass/CSPM-Openshift-Helm.git
```

Create a new project/namespace for Check Point

```
oc new-project <project_name>
```

Create a service account
```
oc create sa <service_account_name>
```

Create the SCC for the service account
```
oc create -f cspm-scc.yaml
```

Apply the SCC to the newly created service account
```
oc adm policy add-scc-to-user cspm-scc system:serviceaccount:<project_name>:<service_account_name>
```


## Installing the Chart

Navigate to the cp-resource-management directory. Package the Helm chart:

```
helm package .
```

Then, install the Helm chart using the API keys, the Cluster ID noted from earlier, and the newly created service account.

```
helm install <release-name> cp-resource-management-1.08.3.tgz --set-string credentials.user=<api_key> --set-string credentials.secret=<api_secret> --set-string clusterID=<cluster_id> --set ocServiceAccount=<service_account_name>  --namespace <project_name>
```

> **Tip**: List all releases using `helm list`

## Uninstalling the Chart

To uninstall/delete the `<release-name>` deployment:

```bash
$ helm delete <release-name> -n <namespace/project>
```

This command removes all the Openshift components associated with the chart and deletes the release.

## Configuration

Refer to [values.yaml](values.yaml) for the full run-down on defaults. These are a mixture of Kubernetes and CSPM directives that map to environment variables.

Specify each parameter using the `--set key=value[,key=value]` argument to `helm install`. For example,

```bash
$ helm install my-release --set varname=true checkpoint/cp-resource-management
```

Alternatively, a YAML file that specifies the values for the parameters can be provided while installing the chart. For example,

```bash
$ helm install my-release -f values.yaml checkpoint/cp-resource-management
```

> **Tip**: You can use the default [values.yaml](values.yaml)

The following tables list the configurable parameters of this chart and their default values.

| Parameter                                                  | Description                                                     | Default                                          |
| ---------------------------------------------------------- | --------------------------------------------------------------- | ------------------------------------------------ |
| `replicaCount`                                             | Number of provisioner instances to deployed                     | `1`                                              |
| `RBAC.create`                                              | Specifies whether RBAC resources should be created              | `true`                                           |
| `RBAC.pspEnable`                                           | Specifies whether PSP resources should be created               | `false`                                          |
| `image.repository`                                         | Provisioner image                                               | `quay.io/checkpoint/cp-resource-management`      |
| `image.tag`                                                | Version of provisioner image                                    | `{TAG_NAME}`                                     |
| `image.pullPolicy`                                         | Image pull policy                                               | `IfNotPresent`                                   |
| `env`                                                      | Additional environmental variables                              | `{}`                                             |
| `credentials.secret`                                       | CloudGuard CSPM APISecret                                       | `CHANGEME`                                       |
| `credentials.user`                                         | CloudGuard CSPM APIID                                           | `CHANGEME`                                       |
| `clusterID`                                                | Cluster Unique identifier in CloudGuard system                  | `CHANGEME`                                       |
| `resources`                                                | Resources required (e.g. CPU, memory)                           | `{}`                                             |
| `podAnnotations`                                           | Arbitrary non-identifying metadata                              | `{}`                                             |
| `nodeSelector`                                             | Node labels for pod assignment                                  | `{}`                                             |
| `tolerations`                                              | List of node taints to tolerate                                 | `[]`                                             |
| `affinity`                                                 | Affinity settings                                               | `{}`                                             |
| `addons.imageUploader.enabled`                             | Specifies whether the Image Uploader addon should be installed  | `false`                                          |
| `addons.imageUploader.enabled.daemonset.image.repository`  | Provisioner image                                               | `quay.io/checkpoint/images-uploader`             |
| `addons.imageUploader.enableddaemonset.image.tag`          | Version of provisioner image                                    | `{TAG_NAME}`                                     |
| `addons.imageUploader.enableddaemonset.image.pullPolicy`   | Image pull policy                                               | `IfNotPresent`                                   |
| `addons.imageUploader.enableddaemonset.resources`          | Version of provisioner image                                    | `{}`                                             |
| `addons.imageUploader.enableddaemonset.nodeSelector`       | Node labels for pod assignment                                  | `{}`                                             |
| `addons.imageUploader.enableddaemonset.tolerations`        | List of node taints to tolerate                                 | `key: node-role.kubernetes.io/master`            |
|                                                            |                                                                 | `effect: NoSchedule`                             |
| `addons.imageUploader.enableddaemonset.affinity`           | Affinity setting                                                | `{}`                                             |