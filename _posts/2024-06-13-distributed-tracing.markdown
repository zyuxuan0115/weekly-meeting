---
layout: post
title:  "2024-06-13 distributed tracing"
date:   2024-06-13 1:53:49 -0500
categories: serverless
---
## Reducing the loading overhead
### Deleting useless functions from IR?
#### LLVM's strip-dead-prototypes
- [strip-dead-prototypes's link](https://www.llvm.org/docs/Passes.html#strip-dead-prototypes-strip-unused-function-prototypes)
- This pass loops over all of the functions in the input module, looking for dead declarations and removes them. Dead declarations are declarations of functions for which no implementation is available (i.e., declarations for unused library functions).
	+ code is [here](https://llvm.org/doxygen/StripDeadPrototypes_8cpp_source.html)
  + to enable this pass, only need to add `-passes=strip-dead-prototypes` 

#### Implib.so
- [Implib.so's github repo](https://github.com/yugr/Implib.so/tree/master)
  + On Linux, if you link against shared library you normally use `-lxyz` compiler option which makes your application depend on `libxyz.so`. This would cause `libxyz.so` to be forcedly loaded at program startup (and its constructors to be executed) even if you never call any of its functions.
  + What this `Implib.so` do is
    * provides all necessary symbols to make linker happy
    * loads wrapped library on first call to any of its functions
    * redirects calls to library symbols

#### TODO
- delete unused function from IR
  + starting from `main` function, check the call graph to find all functions that are never visited
  + delete those unvisited functions

#### 50th latency

| function name<br> (result in millisec) | original | merge  | merge<br> linker opt | merge <br>linker opt<br> delay DLL loading | 
| :----: | :----:   | :----: | :----: | :----: |
| <strong>compose post</strong> <br> (11 functions)  | 154.239 | 67.967 | 53.087  | 43.551 |  
| <strong>text service</strong> <br> (3 functions) | 48.831 | 36.895  | 36.703 | 25.311 |
| <strong>write home timeline</strong> <br> (2 functions) | 28.095 | 31.343 | 29.871 |  20.207 |

#### merging time

| function name<br> | function merging time | 
| :----: | :----:   | 
| <strong>compose post</strong> <br> (11 functions)  | 19m14.546s | 
| <strong>text service</strong> <br> (3 functions) | 10m5.605s | 
| <strong>write home timeline</strong> <br> (2 functions) | 11m23.050s | 


## Distributed tracing on OpenFaaS
According to [openfaas-tracing-walkthrough](https://github.com/LucasRoesler/openfaas-tracing-walkthrough), I found OpenFaaS can use the Grafana/tempo framework for distributed tracing. 

### What are [traces](https://grafana.com/docs/tempo/latest/traces/)?

- A trace represents the whole journey of a request or an action as it moves through all the nodes of a distributed system, especially containerized applications or microservices architectures.

### Grafana/Tempo
- <strong>Tempo</strong>: [Get started with Grafana Tempo using the Helm chart](https://grafana.com/docs/helm-charts/tempo-distributed/next/get-started-helm-charts/)
  + [Tempo CLI](https://grafana.com/docs/tempo/latest/operations/tempo_cli/)
    * Query API command
    * Generate Metrics
  + <strong>Tempo</strong> supports using <strong>OpenTelemetry</strong> for instrumentation
- <strong>Grafana</strong>: [Deploy Grafana using Helm Charts](https://grafana.com/docs/grafana/latest/setup-grafana/installation/helm/)
  + Grafana is a GUI for Tempo's traces and metrics
- <strong>Loki</strong>: [Install Grafana Loki with Helm](https://grafana.com/docs/loki/latest/setup/install/helm/)
  + Loki is a backend store for long-term log retention.

![s2](/assets/2024-06-13/s2.png)

#### example about Grafana working with Tempo

![s3](/assets/2024-06-13/s3.png)

- Connections → Data Sources → Search by name 
  + search for `tempo`
  + the URL field should be `http://<the IP of your cloudlab machine>:3100`
  + no authentication 

- By the way, <strong>OpenFaas</strong> also has a GUI

![ui](/assets/2024-06-13/s1.png)

### [Instrumentation](https://grafana.com/docs/tempo/latest/getting-started/instrumentation/#opentelemetry) for tracing


#### OpenTelemetry
- The <strong>OTLP Exporter</strong> supports exporting logs, metrics and traces in the OTLP format to the OpenTelemetry collector or other compatible backend.

- The <strong>OpenTelemetry Collector</strong> offers a vendor-agnostic implementation on how to receive, process, and export telemetry data. In addition, it removes the need to run, operate, and maintain multiple agents/collectors in order to support open-source telemetry data formats (e.g. Jaeger, Prometheus, etc.) sending to multiple open-source or commercial back-ends.


- Have <strong>auto instrumentation framework</strong>, but we might need to manually add the instrumentation code
- [Language supported](https://opentelemetry.io/docs/languages/): Rust and C++ included
  + in Rust.io, they have [opentelemetry_otlp](https://docs.rs/opentelemetry-otlp/0.16.0/opentelemetry_otlp/) & [opentelemetry-http](https://crates.io/crates/opentelemetry-http)
  + an example of `opentelemetry_otlp`

```rust
use opentelemetry::global;
use opentelemetry::trace::Tracer;

fn main() -> Result<(), Box<dyn std::error::Error + Send + Sync + 'static>> {
    // First, create a OTLP exporter builder. Configure it as you need.
    let otlp_exporter = opentelemetry_otlp::new_exporter().tonic();
    // Then pass it into pipeline builder
    let _ = opentelemetry_otlp::new_pipeline()
        .tracing()
        .with_exporter(otlp_exporter)
        .install_simple()?;
    let tracer = global::tracer("my_tracer");
    tracer.in_span("doing_work", |cx| {
        // Traced app logic here...
    });

    Ok(())
}
```

- [add OpenTelemetry to Kubernete](https://opentelemetry.io/docs/kubernetes/getting-started/)
  + Need to deploy both `OTLP Receiver` and `collector`
  + The collector need to be connected to <strong>Tempo</strong> as the backend exporter 
    * see this example: [End-to-End Distributed Tracing in Kubernetes with Grafana Tempo and OpenTelemetry](https://www.civo.com/learn/distributed-tracing-kubernetes-grafana-tempo-opentelemetry)

```yaml
config:
  receivers:
    otlp:
      protocols:
        grpc:
          endpoint: 0.0.0.0:4317
        http:
          endpoint: 0.0.0.0:4318

  exporters:
    logging:
      loglevel: debug
    otlp:
      endpoint: grafana-tempo:4317
      tls:
        insecure: true
```


#### Ingress Operator for OpenFaaS
- [IngressOperator for OpenFaaS](https://github.com/openfaas/ingress-operator)
- [local KinD ingress](https://docs.openfaas.com/tutorials/local-kind-ingress/)
- [Ingress-Nginx Controller kubernete installation guide](https://kubernetes.github.io/ingress-nginx/deploy/)
