# Chaos Engineering Scenarios

[English](README.md) | [中文](README_zh.md)

## kube-apiserver High Load

**playbook**: `workflow/apiserver-overload-scenario.yaml`

This scenario simulates high load on `kube-apiserver` with the following workflow:
- **Pre-check**: Performs health checks on the target cluster, verifying the health ratio of Nodes and Pods. If below threshold, the test will be aborted. You can adjust thresholds via `precheck-pods-health-ratio` and `precheck-nodes-health-ratio` parameters. Also checks for existence of `tke-chaos-test/tke-chaos-precheck-resource ConfigMap`.
- **Resource Warm-up**: Creates resources (`pods/configmaps`) to simulate production environment scale.
- **Fault Injection**: Floods apiserver with `list pod/configmaps` requests to simulate high load.
- **Cleanup**: Cleans up resources created during the test.

Supports enabling `etcd Overload Protection` and `APF Flow Control` [APF Rate Limiting](https://kubernetes.io/docs/concepts/cluster-administration/flow-control/) during testing via `enable-etcd-overload-protect` and `enable-apf` parameters. These protections can effectively guard against traffic spikes to `kube-apiserver/etcd`. You can compare results with/without protections to decide whether to enable them for your cluster.

**Parameters**

| Parameter | Type | Default | Description |
|-----------|------|---------|-------------|
| `kubeconfig-secret-name` | `string` | `dest-cluster-kubeconfig` | Secret name containing target cluster kubeconfig. If empty, tests current cluster |
| `webhook-url` | `string` | "" | WeChat work group webhook URL. If empty, no notifications will be sent |
| `enable-apf` | `bool` | "false" | Whether to enable APF rate limiting |
| `enable-etcd-overload-protect` | `bool` | "false" | Whether to enable etcd overload protection |
| `precheck-pods-health-ratio` | `float` | "0.9" | Minimum healthy Pod ratio (0.9=90%) to allow test |
| `precheck-nodes-health-ratio` | `float` | "0.9" | Minimum healthy Node ratio (0.9=90%) to allow test |
| `enable-resource-create` | `bool` | "false" | Whether to create resources to simulate cluster scale |
| `resource-create-object-size-bytes` | `int` | "10000" | Resource size in bytes |
| `resource-create-object-count` | `int` | "10" | Number of resources to create |
| `resource-create-qps` | `int` | "10" | QPS for resource creation |
| `from-cache` | `bool` | "true" | Set `true` to test kube-apiserver cache, `false` to test etcd directly |
| `inject-stress-concurrency` | `int` | "1" | Number of stress test Pods |
| `inject-stress-list-qps` | `int` | "100" | QPS per stress test Pod |
| `inject-stress-total-duration` | `string` | "30s" | Total test duration (e.g. 30s, 5m) |

**etcd Overload Protection & Enhanced APF**

Tencent Cloud TKE team has developed these core protection features:

1. **etcd Overload Protection**: Prevent etcd from becoming unavailable due to overload of user component expensive `List` requests (without querying kube-apiserver cache).
2. **Enhanced APF (API Priority and Fairness)**: Provides fine-grained flow control for expensive `List` operations.

Supported versions:

| TKE Version       |
|-------------------|
| v1.30.0-tke.9+    |
| v1.28.3-tke.14+   |
| v1.26.1-tke.17+   |
| v1.24.4-tke.26+   |
| v1.22.5-tke.35+   |
| v1.20.6-tke.54+   |

## coredns Disruption

**playbook**: `workflow/coredns-disruption-scenario.yaml`

This scenario simulates coredns service disruption by:
1. Scaling coredns Deployment replicas to 0
2. Maintaining zero replicas for specified duration
3. Restoring original replica count

**Parameters**

| Parameter | Type | Default | Description |
|-----------|------|---------|-------------|
| `disruption-duration` | `string` | `30s` | Disruption duration (e.g. 30s, 5m) |
| `kubeconfig-secret-name` | `string` | `dest-cluster-kubeconfig` | Target cluster kubeconfig secret name |

## kubernetes-proxy Disruption

**playbook**: `workflow/kubernetes-proxy-disruption-scenario.yaml`

This scenario simulates kubernetes-proxy service disruption by:
1. Scaling kubernetes-proxy Deployment replicas to 0
2. Maintaining zero replicas for specified duration
3. Restoring original replica count

**Parameters**

| Parameter | Type | Default | Description |
|-----------|------|---------|-------------|
| `disruption-duration` | `string` | `30s` | Disruption duration (e.g. 30s, 5m) |
| `kubeconfig-secret-name` | `string` | `dest-cluster-kubeconfig` | Target cluster kubeconfig secret name |

## Namespace Deletion Protection

**playbook**: `workflow/namespace-delete-scenario.yaml`

This scenario tests Tencent Cloud TKE's namespace deletion block policy with the following workflow:
- **Create namespace deletion block policy**: Create a namespace deletion constraint policy to prevent deletion of namespaces containing Pods
- **Create test resources**: Creates test namespace `tke-chaos-ns-76498` and Pod
- **Verify protection**: Attempts to delete namespace with Pod to verify protection works
- **Cleanup**: Deletes Pod first, then namespace, and finally removes namespace deletion block policy

**Parameters**
| Parameter | Type | Default | Description |
|-----------|------|---------|-------------|
| `kubeconfig-secret-name` | `string` | `dest-cluster-kubeconfig` | Target cluster kubeconfig secret name |

Tencent Cloud TKE supports various resource protection policies, such as CRD deletion protection, PV deletion protection, etc. You can refer to the official Tencent Cloud documentation for more details: [Policy Management](https://cloud.tencent.com/document/product/457/103179)

## TKE Self-maintenance of Master cluster's kube-apiserver Disruption
TODO

## TKE Self-maintenance of Master cluster's etcd Disruption
TODO

## TKE Self-maintenance of Master cluster's kube-controller-manager Disruption
TODO

## TKE Self-maintenance of Master cluster's kube-scheduler Disruption
TODO

## Managed Cluster's kube-apiserver Disruption
TODO

## Managed Cluster's kube-controller-manager Disruption
TODO

## Managed Cluster's kube-scheduler Disruption
TODO
