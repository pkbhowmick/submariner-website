---
title: "subctl"
weight: 10
---

The `subctl` command-line utility simplifies the deployment and maintenance of Submariner by automating interactions with the [Submariner
Operator](https://github.com/submariner-io/submariner-operator).

## Synopsis

`subctl [command] [--flags] ...`

## Installation

{{% subctl-install %}}

### Installing specific versions

By default, [https://get.submariner.io](https://get.submariner.io) will provide the latest release for `subctl`, and hence Submariner.
Specific versions can be requested by using the VERSION environment variable.

Avalailable options are:

* `latest`: the latest stable release (default)
* `devel`: the devel branch code.
* `rc`: the latest release candidate.
* `x.x.x` (like 0.6.1, 0.5.0, etc)

For example

```bash
curl https://get.submariner.io | VERSION=devel bash
```

## Common options

`subctl` commands which need access to a Kubernetes cluster handle the same access mechanisms as `kubectl` (see `kubectl options`).
One or more `kubeconfig` files can be listed in the `KUBECONFIG` environment variable, or using the `--kubeconfig` option.
A specific context can be chosen using the `--context` option;
by default, `subctl` commands use the chosen `kubeconfig`’s current context.

Where appropriate, a namespace can be chosen using the `--namespace` (`-n`) option;
by default, `subctl` commands use either the appropriate Submariner namespace, or the chosen context’s current namespace.

Some commands support multiple contexts.
The “reference” context is the context specified by `--context`;
the “other” context is identified by a prefix, _e.g._ `--tocontext` or `--remotecontext`.
All the options providing access to a cluster are available with the corresponding prefix:
`--toconfig` (the `kubeconfig`), `--tousername` etc.
It is possible to use mutually conflicting `kubeconfig` files (_e.g._ files using the same names for different values)
by specifiying them using `--kubeconfig` and the prefixed `--…config`;
corresponding settings will only use the information from the matching `kubeconfig`.

References to the “selected context” and “selected namespace” in the documentation below refer to the context and namespace
specified using the options described above.

## Commands

### `deploy-broker`

`subctl deploy-broker [flags]`

The `deploy-broker` command configures the cluster specified by the selected context as the Broker.
It installs the necessary CRDs and the `submariner-k8s-broker` namespace.

In addition, it generates a `broker-info.subm` file which can be used with the `join` command to connect clusters to the Broker. This file
contains the following details:

* Encryption PSK key
* Broker access details for subsequent `subctl` runs
* Service Discovery settings

#### `deploy-broker` flags

<!-- markdownlint-disable line-length -->
| Flag                                  | Description
|:--------------------------------------|:---------------------------------------------------------------------------------------------------|
| `--kubeconfig` `<string>`             | Absolute path(s) to the kubeconfig file(s) (default `$HOME/.kube/config`)
| `--context` `<string>`                | kubeconfig context to use
| `--repository` `<string>`             | The repository from where the various Submariner images will be sourced (default `quay.io/submariner`)
| `--version` `<string>`                | Image version (default image tag "devel")
| `--components <strings>`              | Comma-separated list of components to be installed - any of `service-discovery`,`connectivity`. The default is: `service-discovery`,`connectivity`
| `--globalnet`                         | Enable support for overlapping Cluster/Service CIDRs in connecting clusters (default disabled)
| `--globalnet-cidr-range` `<string>`   | Global CIDR supernet range for allocating GlobalCIDRs to each cluster (default "242.0.0.0/8")
| `--globalnet-cluster-size` `<value>`  | Default cluster size for GlobalCIDR allocated to each cluster (amount of global IPs) (default 65536)
| `--ipsec-psk-from` `<string>`         | Import IPsec PSK from existing Submariner broker file, like broker-info.subm (default `broker-info.subm`)
| `--broker-namespace` `<string>`       | Namespace on the Broker used for synchronizing resources between clusters (default `submariner-k8s-broker`)
<!-- markdownlint-enable line-length -->

### `export`

#### `export service`

`subctl export service [flags] <name>` creates a `ServiceExport` resource for the given Service name. This makes the corresponding Service
discoverable from other clusters in the Submariner deployment.

#### `export service` flags

| Flag                         | Description
|:-----------------------------|:----------------------------------------------------------------------------|
| `--kubeconfig` `<string>`    | Absolute path(s) to the kubeconfig file(s) (default `$HOME/.kube/config`)
| `--context` `<string>`       | Kubeconfig context to use
| `--namespace` `<string>`     | Namespace to use

If no `namespace` flag is specified, it uses the default namespace from the current context, if present, otherwise it uses `default`.

### `unexport`

#### `unexport service`

`subctl unexport service [flags] <name>` removes the `ServiceExport` resource with the given name which in turn stops the Service of the
same name from being exported to other clusters.

#### `unexport service` flags

| Flag                         | Description
|:-----------------------------|:----------------------------------------------------------------------------|
| `--kubeconfig` `<string>`    | Absolute path(s) to the kubeconfig file(s) (default `$HOME/.kube/config`)
| `--context` `<string>`       | Kubeconfig context to use
| `--namespace` `<string>`     | Namespace to use

If no `namespace` flag is specified, it uses the default namespace from the current context, if present, otherwise it uses `default`.

### `join`

`subctl join broker-info.subm [flags]`

The `join` command deploys the Submariner Operator in a cluster using the settings provided in the `broker-info.subm` file. The service
account credentials needed for the new cluster to access the Broker cluster will be created and provided to the Submariner Operator
deployment.

#### `join` flags (general)
<!-- markdownlint-disable line-length -->
| Flag                               | Description
|:-----------------------------------|:----------------------------------------------------------------------------|
| `--air-gapped`                     | Specifies that the cluster is in an air-gapped environment without access to external servers.
| `--cable-driver` `<string>`        | Cable driver implementation. Available options are `libreswan` (default), `wireguard` and `vxlan`
| `--clusterid` `<string>`           | Cluster ID used to identify the tunnels. Every cluster needs to have a unique cluster ID. If not provided, one will be generated by default based on the cluster name in the `kubeconfig` file; if the cluster name is not a [valid cluster ID](https://kubernetes.io/docs/concepts/overview/working-with-objects/names/#dns-label-names), the user will be prompted for one
| `--clustercidr` `<string>`         | Specifies the cluster's CIDR used to generate Pod IP addresses. If not specified, `subctl` will try to discover it and if unable to do so, it will prompt the user
| `--label-gateway`                  | Label getways (enabled by default). `--label-gateway=false` disables the prompt for a Worker node to use as gateway
| `--pod-debug`                      | Enable Submariner pod debugging (verbose logging in the deployed pods)
| `--load-balancer`                  | Enable a cloud loadbalancer in front of the gateways. This removes the need for dedicated nodes with a public IP address
| `--preferred-server`               | Enable this cluster as a preferred IPsec server for dataplane connections (only available with libreswan cable driver)
| `--check-broker-certificate`       | Verify the broker certificate (enabled by default). `--check-broker-certificate=false` disables certificate checks; communications are still secured with TLS

<!-- markdownlint-enable line-length -->

#### `join` flags (Globalnet)
<!-- markdownlint-disable line-length -->
| Flag                                 | Description
|:-------------------------------------|:----------------------------------------------------------------------------|
| `--globalnet`                        | Enable/disable Globalnet for this cluster (default true). This has no effect if Globalnet is not enabled globally via the Broker
| `--globalnet-cluster-size` `<value>` | If Globalnet is enabled, the cluster size for the GlobalCIDR allocated to this cluster (amount of global IPs)
| `--globalnet-cidr` `<string>`        | If Globalnet is enabled, the specific Globalnet CIDR to use for this cluster. This setting is exclusive with `--globalnet-cluster-size`
<!-- markdownlint-enable line-length -->

#### `join` flags (IPsec)

| Flag                    | Description
|:------------------------|:-----------------------------------------------|
| `--natt`                | Enable NAT for IPsec (default enabled)
| `--ipsec-debug`         | Enable IPsec debugging (verbose logging)
| `--nattport` `<value>`  | IPsec NAT-T port (default 4500)

#### `join` flags (images and repositories)
<!-- markdownlint-disable line-length -->
| Flag                                    | Description
|:----------------------------------------|:----------------------------------------------------------------------------|
| `--repository` `<string>`               | The repository from where the various Submariner images will be sourced (default `quay.io/submariner`)
| `--version` `<string>`                  | Image version (default image tag "devel")
| `--image-override` `<string>=<string>`  | Component image override. This flag can be used more than once (example: `--image-override=submariner=quay.io/myUser/submariner:latest`)
<!-- markdownlint-enable line-length -->

#### `join` flags (health check)
<!-- markdownlint-disable line-length -->
| Flag                                            | Description
|:------------------------------------------------|:----------------------------------------------------------------------------|
| `--health-check`                                | Enable/disable Gateway health check (default true)
| `--health-check-interval` `<uint>`              | The interval in seconds at which health check packets will be sent (default 1)
| `--health-check-max-packet-loss-count` `<uint>` | The maximum number of packets lost at which the health checker will mark the connection as down (default 5)
<!-- markdownlint-enable line-length -->

### `show`

#### `show networks`

`subctl show networks [flags]`

Inspects the cluster and reports information about the detected network plugin and detected Cluster and Service CIDRs.

#### `show versions`

`subctl show versions [flags]`

Shows the version and image repository of each Submariner component in the cluster.

#### `show gateways`

`subctl show gateways [flags]`

Shows summary information about the Submariner gateways in the cluster.

#### `show connections`

`subctl show connections [flags]`

Shows information about the Submariner endpoint connections with other clusters.

#### `show endpoints`

`subctl show endpoints [flags]`

Shows information about the Submariner endpoints in the cluster.

### `show brokers`

`subctl show brokers [flags]`

Shows information about the Broker in the cluster.

#### `show all`

`subctl show all [flags]`

Shows the aggregated information from all the other show commands.

#### `show` flags

| Flag                         | Description
|:-----------------------------|:----------------------------------------------------------------------------|
| `--kubeconfig` `<string>`    | Absolute path(s) to the kubeconfig file(s) (default `$HOME/.kube/config`)
| `--context` `<string>`       | Kubeconfig context to use

### `verify`

`subctl verify --context <context1> --tocontext <context2> [flags]`

The `verify` command verifies a Submariner deployment between two clusters is functioning properly. `<context1>` will be
`ClusterA` in the reports, while `<context2>` will be `ClusterB` in the reports. The `--verbose` flag is recommended to see what's
happening during the tests.

There are several suites of verifications that can be performed. By default all verifications are performed.  Some verifications are deemed
disruptive in that they change some state of the clusters as a side effect.  If running the command interactively, you will be prompted for
confirmation to perform disruptive verifications unless the `--disruptive-tests` flag is also specified. If running non-interactively (that
is with no stdin), `--disruptive-tests` must be specified otherwise disruptive verifications are skipped.

The `connectivity` suite verifies dataplane connectivity across the clusters for the following cases:

* Pods (on Gateway nodes) to Services
* Pods (on non-Gateway nodes) to Services
* Pods (on Gateway nodes) to Pods
* Pods (on non-Gateway nodes) to Pods

The `service-discovery` suite verifies DNS discovery of `<service>.<namespace>.svc.clusterset.local` entries across the clusters.

The `gateway-failover` suite verifies the continuity of cross-cluster dataplane connectivity after a gateway failure in a cluster occurs.
This suite requires a single gateway configured on `ClusterA` and other available Worker nodes capable of serving as gateways. Please note
that this verification is disruptive.

#### `verify` flags

| Flag                                | Description
|:------------------------------------|:----------------------------------------------------------------------------|
| `--connection-attempts` `<value>`   | The maximum number of connection attempts (default 2)
| `--connection-timeout` `<value>`    | The timeout in seconds per connection attempt  (default 60)
| `--operation-timeout` `<value>`     | Operation timeout for Kubernetes API calls (default 240)
| `--junit-report` `<string>`         | XML report path and name (default "")
| `--verbose`                         | Produce verbose logs during connectivity verification
| `--only`                            | Comma separated list of specific verifications to perform
| `--disruptive-tests`                | Enable verifications which are potentially disruptive to your deployment

### `benchmark`

#### `benchmark throughput`

`subctl benchmark throughput --context <context1> [--tocontext <context2>] [flags]`

The `benchmark throughput` command runs a throughput benchmark test between two specified clusters or within a single cluster.
It deploys a Pod to run the [iperf](https://iperf.fr/) tool and logs the output to the console.
When running `benchmark throughput`, two types of tests will be executed:

* Pod to Pod - where both Pods are scheduled on Gateway nodes
* Pod to Pod - where both Pods are scheduled on non-Gateway nodes

#### `benchmark latency`

`subctl benchmark latency --context <context1> [--tocontext <context2>] [flags]`

The `benchmark latency` command runs a latency benchmark test between two specified clusters or within a single cluster.
It deploys a Pod to run the [netperf](https://hewlettpackard.github.io/netperf/doc/netperf.html) tool and logs the output to the console.
When running `benchmark latency`, two types of tests will be executed:

* Pod to Pod - where both Pods are scheduled on Gateway nodes
* Pod to Pod - where both Pods are scheduled on non-Gateway nodes

#### `benchmark` flags
<!-- markdownlint-disable line-length -->
| Flag                                | Description
|:------------------------------------|:----------------------------------------------------------------------------|
| `--verbose`                         | Produce verbose logs during benchmark tests
<!-- markdownlint-enable line-length -->

### `diagnose`

The `subctl diagnose` command is a tool that runs various checks to help diagnose issues in a Submariner deployment or some configurations
in the cluster that may prevent Submariner from working properly.

Below is a list of available sub-commands:

<!-- markdownlint-disable line-length -->
| Diagnose command           | Description                                                                 | Flags
|:---------------------------|:----------------------------------------------------------------------------|:----------
| `deployment`               | checks that the Submariner components are properly deployed and running with no overlapping CIDRs
| `connections`              | checks that the Gateway connections to other clusters are all established
| `k8s-version`              | checks if Submariner can be deployed on the Kubernetes version
| `kube-proxy-mode [flags]`  | checks if the kube-proxy mode is supported by Submariner  | `--namespace` `<string>`
| `cni`                      | checks if the detected CNI network plugin is supported by Submariner
| `firewall intra-cluster [flags]`   | checks if the firewall configuration allows traffic via intra-cluster Submariner VXLAN interface | `--validation-timeout` `<value>`, `--verbose`, `--namespace` `<string>`
| `firewall inter-cluster --context <localcontext> --remotecontext <remotecontext> [flags]`  | checks if the firewall configuration allows tunnels to be configured on the Gateway nodes | `--validation-timeout` `<value>`, `--verbose`, `--namespace` `<string>`
| `all`                      | runs all diagnostic checks (except those requiring two kubecontexts) |  
<!-- markdownlint-enable line-length -->

#### `diagnose` flags descriptions

<!-- markdownlint-disable line-length -->
| Flag                             | Description
|:---------------------------------|:----------------------------------------------------------------------------|
| `--namespace` `<string>`         | Namespace in which validation pods should be deployed. If not specified, the `default` namespace is used
| `--validation-timeout` `<value>` | Timeout in seconds while validating the connection attempt
| `--verbose`                      | Produce verbose logs during validation
<!-- markdownlint-enable line-length -->

#### `diagnose` global flags

| Flag                         | Description
|:-----------------------------|:----------------------------------------------------------------------------|
| `--kubeconfig` `<string>`    | Absolute path(s) to the kubeconfig file(s) (default `$HOME/.kube/config`)
| `--context` `<string>`       | Kubeconfig context to use
| `--in-cluster`               | Use the in-cluster configuration to connect to Kubernetes.

### `gather`

The `subctl gather` command is a tool that collects various information from clusters to aid in troubleshooting a
Submariner deployment, including Kubernetes resources and Pod logs. Clusters from which information is gathered are provided
via the `--kubeconfig` flag (or the `KUBECONFIG` environment variable). By default it will gather information from all the cluster contexts contained
in the kubeconfig. To gather information from specific clusters, contexts can be passed using `--contexts` flag.

The tool creates a UTC timestamped directory of the format `submariner-YYYYMMDDHHMMSS` containing various files.
Kubernetes resources are written to YAML files with the naming format `<cluster-name>_<resource-type>_<namespace>_<resource-name>.yaml`.
Pod logs are written to files with the format `<cluster-name>_<pod-name>.log`

The specific information collected is configurable. As part of gathering `connectivity` resources, it also collects information specific
to the CNI and Submariner cable driver in use from each node using file format `<cluster-name>_<node-name>_<command>.yaml`

#### `gather` flags

| Flag                       | Description
|:---------------------------|:--------------------------------------------------------------------------------------------------------------------------|
| `--kubeconfig` `<string>`  | Absolute path(s) to the kubeconfig file(s)
| `--contexts` `<string>`    | comma separated list of kube contexts to use. By default all contexts referenced by kubeconfig are used
| `--module` `<string>`      | Comma-separated list of components for which to gather data. Default is `operator,connectivity,service-discovery,broker`
| `--type` `<string>`        | Comma-separated list of data types to gather. Default is `logs,resources`

#### `gather` examples

These examples assume that kubeconfigs have been passed using the `KUBECONFIG` environment variable. Alternately,
add the `--kubeconfig` flag if the environment variable is not set.

##### `gather` all from all clusters

It is recommended to use this when reporting any issue.

`subctl gather`

##### `gather` all from specific clusters

`subctl gather --contexts cluster-east`

##### `gather` operator and connectivity logs from specific clusters

`subctl gather --contexts cluster-east,cluster-west --module operator,connectivity --type logs`

##### `gather` broker and service-discovery resources from all clusters

`subctl gather --module broker,service-discovery --type resources`

### `cloud`

#### `cloud prepare`

`subctl cloud prepare [flags]`

This command prepares the underlying cloud infrastructure for Submariner installation.

#### `prepare` global flags

<!-- markdownlint-disable line-length -->
| Flag                             | Description
|:---------------------------------|:--------------------------------------------------------------------------------------------------------------------------|
| `--nat-discovery-port` `<uint16>`       | NAT discovery port (default 4490)
| `--natt-port` `<uint16>`                | IPsec NAT traversal port (default 4500)
| `--vxlan-port` `<uint16>`               | Internal VXLAN port (default 4800)
<!-- markdownlint-enable line-length -->

#### `prepare aws`

`subctl cloud prepare aws [flags]`

This command prepares an OpenShift installer-provisioned infrastructure (IPI) on AWS cloud for Submariner installation.

<!-- markdownlint-disable line-length -->
| Flag                             | Description
|:---------------------------------|:--------------------------------------------------------------------------------------------------------------------------|
| `--credentials` `<string>`       | AWS credentials configuration file (default `$HOME/.aws/credentials`)
| `--gateway-instance` `<string>`  | Type of gateway instance machine (default `c5d.large`)
| `--gateways` `<int>`             | Number of dedicated gateways to deploy (Set to 0 when using --load-balancer mode) (default 1)
| `--infra-id` `<string>`          | AWS infra ID
| `--ocp-metadata` `<string>`      | OCP metadata.json file (or directory containing it) to read AWS infra ID and region from (takes precedence over the specific flags)
| `--profile` `<string>`           | AWS profile to use for credentials  (default "default")
| `--region` `<string>`            | AWS region
<!-- markdownlint-enable line-length -->

#### `prepare gcp`

`subctl cloud prepare gcp [flags]`

This command prepares an OpenShift installer-provisioned infrastructure (IPI) on GCP cloud for Submariner installation.

<!-- markdownlint-disable line-length -->
| Flag                             | Description
|:---------------------------------|:--------------------------------------------------------------------------------------------------------------------------|
| `--credentials` `<string>`       | GCP credentials configuration file (default `$HOME/.gcp/osServiceAccount.json`)
| `--dedicated-gateway`            | Whether a dedicated gateway node has to be deployed (default true)
| `--gateway-instance` `<string>`  | Type of gateway instance machine (default `n1-standard-4`)
| `--gateways` `<int>`             | Number of dedicated gateways to deploy (default 1)
| `--infra-id` `<string>`          | GCP infra ID
| `--ocp-metadata` `<string>`      | OCP metadata.json file (or directory containing it) to read GCP infra ID and region from (takes precedence over the specific flags)
| `--project-id` `<string>`        | GCP project ID
| `--region` `<string>`            | GCP region
<!-- markdownlint-enable line-length -->

#### `prepare rhos`

`subctl cloud prepare rhos [flags]`

This command prepares an OpenShift installer-provisioned infrastructure (IPI) on OpenStack cloud for Submariner installation.

<!-- markdownlint-disable line-length -->
| Flag                             | Description
|:---------------------------------|:--------------------------------------------------------------------------------------------------------------------------|
| `--cloud-entry` `<string>`       | Specific cloud configuration to use from the clouds.yaml
| `--dedicated-gateway`            | Whether a dedicated gateway node has to be deployed (default true)
| `--gateway-instance` `<string>`  | Type of gateway instance machine (default `PnTAE.CPU_4_Memory_8192_Disk_50`)
| `--gateways` `<int>`             | Number of gateways to deploy (default 1)
| `--infra-id` `<string>`          | OpenStack infra ID
| `--ocp-metadata` `<string>`      | OCP metadata.json file (or directory containing it) to read OpenStack infra ID and region from (takes precedence over the specific flags)
| `--project-id` `<string>`        | OpenStack project ID
| `--region` `<string>`            | OpenStack region
<!-- markdownlint-enable line-length -->

#### `prepare generic` flags

This command prepares a generic cluster for Submariner installation. It assumes that the cloud already has the necessary
firewall ports opened and will only label the required number of gateway nodes for Submariner installation.

| Flag                             | Description
|:---------------------------------|:--------------------------------------------------------------------------------------------------------------------------|
| `--gateways` `<int>`             | Number of gateways to deploy (default 1)

#### `cloud cleanup`

This command cleans up the cloud after Submariner uninstallation.

#### `cleanup aws`

`subctl cloud cleanup aws [flags]`

This command cleans up an OpenShift installer-provisioned infrastructure (IPI) on AWS-based cloud after Submariner uninstallation.

| Flag                             | Description
|:---------------------------------|:--------------------------------------------------------------------------------------------------------------------------|
| `--credentials` `<string>`       | AWS credentials configuration file (default `$HOME/.aws/credentials`)
| `--infra-id` `<string>`          | AWS infra ID
| `--ocp-metadata` `<string>`      | OCP metadata.json file (or directory containing it) to read AWS infra ID and region from
| `--profile` `<string>`           | AWS profile to use for credentials
| `--region` `<string>`            | AWS region

#### `cleanup gcp`

`subctl cloud cleanup gcp [flags]`

This command cleans up an installer-provisioned infrastructure (IPI) on GCP-based cloud after Submariner uninstallation.

| Flag                             | Description
|:---------------------------------|:--------------------------------------------------------------------------------------------------------------------------|
| `--credentials` `<string>`       | GCP Credentials configuration file (default `$HOME/.gcp/osServiceAccount.json`)
| `--infra-id` `<string>`          | GCP infra ID
| `--ocp-metadata` `<string>`      | OCP metadata.json file (or directory containing it) to read GCP infra ID and region from
| `--project-id` `<string>`        | GCP project ID
| `--region` `<string>`            | GCP region

#### `cleanup rhos`

`subctl cloud cleanup rhos [flags]`

This command cleans up an installer-provisioned infrastructure (IPI) on OpenStack-based cloud after Submariner uninstallation.

| Flag                             | Description
|:---------------------------------|:--------------------------------------------------------------------------------------------------------------------------|
| `--cloud-entry` `<string>`       | the cloud entry to use (default `openstack`)
| `--infra-id` `<string>`          | OpenStack infra ID
| `--ocp-metadata` `<string>`      | OCP metadata.json file (or directory containing it) to read OpenStack infra ID and region from
| `--project-id` `<string>`        | OpenStack project ID
| `--region` `<string>`            | OpenStack region

#### `cleanup generic`

`subctl cloud cleanup generic [flags]`

This command removes the labels from gateway nodes after Submariner uninstallation.

#### `cleanup` flags

| Flag                             | Description
|:---------------------------------|:--------------------------------------------------------------------------------------------------------------------------|
| `--kubeconfig` `<string>`        | Absolute path(s) to the kubeconfig file(s)
| `--context` `<string>`           | Kubernetes context to use

### `version`

`subctl version`

Prints the version details for the `subctl` binary.

### `uninstall`

`subctl uninstall [flags]`

This command uninstalls Submariner and its components.

The following steps are performed:

* Delete Submariner ClusterRoles and ClusterRoleBindings.
* Delete the `submariner.io` CRDs.
* Delete the routing entries (iptables/routes/ipsets) programmed on the nodes.
* Delete the tunnel interfaces created for internal/external communication.
* Delete the Submariner namespace.
* Delete the Broker namespace, if present.
* Unlabel the gateway nodes.

#### `uninstall` flags

| Flag                             | Description
|:---------------------------------|:--------------------------------------------------------------------------------------------------------------------------|
| `--kubeconfig` `<string>`        | Absolute path(s) to the kubeconfig file(s)
| `--namespace` `<string>`         | Namespace in which Submariner is installed (default `submariner-operator`)
| `--yes`                          | Automatically answer yes to confirmation prompt

### Running `subctl diagnose` from a Pod in cluster

As of 0.12.0, a [subctl](https://quay.io/repository/submariner/subctl?tab=tags&tag=latest) image is provided for running the `subctl` binary
from a Pod within a cluster.  The output of the `subctl diagnose` command output can be accessed from the Pod's logs. The `--in-cluster`
flag was added to `subctl diagnose` to support this use case.

#### Example running `subctl diagnose` using a Job

```yaml
apiVersion: batch/v1
kind: Job
metadata:
  name: submariner-diagnose
  namespace: submariner-operator
spec:
  template:
    metadata:
      labels:
        submariner.io/transient: "true"
    spec:
      containers:
      - name: submariner-diagnose
        image: quay.io/submariner/subctl:devel
        command: ["subctl",  "diagnose", "all", "--in-cluster"]
      restartPolicy: Never
      serviceAccount: submariner-diagnose
      serviceAccountName: submariner-diagnose
  backoffLimit: 0
```

The resulting Pod with `subctl diagnose all --in-cluster` logs can be accessed with the label `job-name=submariner-diagnose`.
A similar template can also be used to create a `CronJob` that runs `subctl diagnose` periodically.
