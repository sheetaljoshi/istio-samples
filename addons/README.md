# Telemetry Addons

This directory contains sample deployments of various addons that integrate with Istio. While these applications
are not a part of Istio, they are essential to making the most of Istio's observability features.

The deployments here are meant to quickly get up and running, and are optimized for this case. As a result,
they may not be suitable for production. See below for more info on integrating a production grade version of each
addon.

## Getting started

To quickly deploy all addons:

```shell script
kubectl apply -f addons
```

Alternatively, you can deploy individual addons:

```shell script
kubectl apply -f addons/prometheus.yaml
```

## Addons

### Prometheus

[Prometheus](https://prometheus.io/) is an open source monitoring system and time series database.
You can use Prometheus with Istio to record metrics that track the health of Istio and of applications within the service mesh.
You can visualize metrics using tools like [Grafana](#grafana) and [Kiali](#kiali).

For more information about integrating with Prometheus, please see the [Prometheus integration page](https://istio.io/docs/ops/integrations/prometheus/).

### Grafana

[Grafana](http://grafana.com/) is an open source monitoring solution that can be used to configure dashboards for Istio.
You can use Grafana to monitor the health of Istio and of applications within the service mesh.

This sample provides the following dashboards:

* [Mesh Dashboard](https://grafana.com/grafana/dashboards/7639) provides an overview of all services in the mesh.
* [Service Dashboard](https://grafana.com/grafana/dashboards/7636) provides a detailed breakdown of metrics for a service.
* [Workload Dashboard](https://grafana.com/grafana/dashboards/7630) provides a detailed breakdown of metrics for a workload.
* [Performance Dashboard](https://grafana.com/grafana/dashboards/11829) monitors the resource usage of the mesh.
* [Control Plane Dashboard](https://grafana.com/grafana/dashboards/7645) monitors the health and performance of the control plane.
* [WASM Extension Dashboard](https://grafana.com/grafana/dashboards/13277) provides an overview of mesh wide WebAssembly extension runtime and loading state.

For more information about integrating with Grafana, please see the [Grafana integration page](https://istio.io/docs/ops/integrations/grafana/).

### Kiali

[Kiali](https://kiali.io/) is an observability console for Istio with service mesh configuration capabilities.
It helps you to understand the structure of your service mesh by inferring the topology, and also provides the health of your mesh.
Kiali provides detailed metrics, and a basic [Grafana](#grafana) integration is available for advanced queries.
Distributed tracing is provided by integrating [Jaeger](#jaeger).

For more information about using Kiali, see the [Visualizing Your Mesh](https://istio.io/docs/tasks/observability/kiali/) task.

### Jaeger

[Jaeger](https://www.jaegertracing.io/) is an open source end to end distributed tracing system, allowing users to monitor and troubleshoot transactions in complex distributed systems.

Jaeger helps in a variety of tasks including:

* Distributed context propagation
* Distributed transaction monitoring
* Root cause analysis
* Service dependency analysis
* Performance / latency optimization

For more information about integrating with Jaeger, please see the [Jaeger integration page](https://istio.io/docs/tasks/observability/distributed-tracing/jaeger/).

### Zipkin

[Zipkin](https://zipkin.io/) is a distributed tracing system. It helps gather timing data needed to troubleshoot latency problems in service architectures. Features include both the collection and lookup of this data.

Zipkin is an alternative to Jaeger and is not deployed by default. To replace Jaeger with Zipkin, run `kubectl apply -f samples/addons/extras/zipkin.yaml`.
You may also want to remove the Jaeger deployment, which will not be used, with `kubectl delete deployment jaeger`, or avoid installing it
to begin with by following the selective install steps in [Getting Started](#getting-started).

For more information about integrating with Zipkin, please see the [Zipkin integration page](https://istio.io/docs/tasks/observability/distributed-tracing/zipkin/).

### Prometheus Operator

The [Prometheus Operator](https://github.com/coreos/prometheus-operator) manages and operators a Prometheus instance.

As an alternative to the standard Prometheus deployment, we provide a `ServiceMonitor` to monitor the Istio control plane and `PodMonitor`
Envoy proxies. To use these, make sure you have the Prometheus operator deployed, then run `kubectl apply -f samples/addons/extras/prometheus-operator.yaml`.

Note: The example `PodMonitor` requires [metrics merging](https://istio.io/latest/docs/ops/integrations/prometheus/#option-1-metrics-merging) to be enabled. This is enabled by default.

Note: The configurations here are only for Istio deployments, and do not scrape metrics from the Kubernetes components. See the [Cluster Monitoring](https://coreos.com/operators/prometheus/docs/latest/user-guides/cluster-monitoring.html) documentation for configuring this.

### Remotely Accessing Telemetry Addons

The [Remotely Accessing Telemetry Addons](https://istio.io/latest/docs/tasks/observability/gateways/) shows how to configure Istio to expose and access the telemetry addons outside of a cluster. This is one way of exposing Telemetry Addons remotely. This approach requires setting up a domain. The examples demonstrates using AWS Route 53 registered domain.

```shellscript
export INGRESS_DOMAIN=example.com
```

```shellscript
cat addons/networking/prometheus.yaml | envsubst | kubectl apply -f -
cat addons/networking/grafana.yaml | envsubst | kubectl apply -f -
cat adddons/networking/kiali.yaml | envsubst | kubectl apply -f -
cat addons/networking/tracing.yaml | envsubst | kubectl apply -f -
```

#### Cleanup
```shellscript 
kubectl delete -f addons/networking/.
```

### Setting up ExternalDNS
[ExternalDNS](https://github.com/kubernetes-sigs/external-dns) makes Kubernetes resources discoverable via public DNS servers. It's not a DNS server itself, but merely configures other DNS providers accordingly—e.g. AWS Route 53. This sample describes how to setup ExternalDNS for usage within a Kubernetes cluster on AWS
nd and ExternalDNS to use Istio Gateway source. 

```shellscript
export INGRESS_DOMAIN=example.com
```

Setup IAM policy to allow ExternalDNS to update Route53 Resource Record Sets and Hosted Zones.

```shellscript
aws iam create-policy   \
  --policy-name external-dns-policy \
  --policy-document file://~/environment/istio-samples/addons/networking/external-dns/external-dns-policy.json
```

Create IAM role.

```shellscript
eksctl create iamserviceaccount \
    --name external-dns \
    --namespace default \
    --role-name external-dns-role \
    --cluster eksworkshop-eksctl \
    --attach-policy-arn arn:aws:iam::${ACCOUNT_ID}:policy/external-dns-policy \
    --approve \
    --override-existing-serviceaccounts
```

Deploy external DNS
```shellscript
cat ~/environment/istio-samples/addons/networking/external-dns/external-dns.yaml | envsubst | kubectl apply -f -
```

### Cleanup
  
```shellscript

kubectl delete -f ~/environment/istio-samples/addons/networking/external-dns/external-dns.yaml

eksctl delete iamserviceaccount \
    --name external-dns \
    --namespace default \
    --cluster eksworkshop-eksctl \
    --wait

aws iam delete-policy --policy-arn arn:aws:iam::${ACCOUNT_ID}:policy/external-dns-policy
```
