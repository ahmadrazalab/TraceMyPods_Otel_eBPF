# OpenTelemetry Demo Kubernetes Deployment

This repository provides a guide to deploying the OpenTelemetry Demo application in a Kubernetes environment. The demo showcases how OpenTelemetry works and can be integrated into your observability stack.

## Prerequisites

Before you begin, ensure your environment meets the following requirements:

- Kubernetes 1.24+
- At least 6 GB of free RAM
- Helm 3.14+ (if using Helm for installation)

---

## Installation Options

### 1. Install Using Helm (Recommended)

Helm simplifies deployment and management of the OpenTelemetry Demo. Follow these steps:

#### Step 1: Add the OpenTelemetry Helm Repository
```bash
helm repo add open-telemetry https://open-telemetry.github.io/opentelemetry-helm-charts
```

#### Step 2: Install the Helm Chart
To install the chart with the release name `my-otel-demo`, run:
```bash
helm install my-otel-demo open-telemetry/opentelemetry-demo
```

#### Notes:
- Upgrading the Helm chart from one version to another is not supported. To upgrade, delete the existing release and reinstall the updated version.
- Ensure you use Helm chart version 0.11.0 or later to access all features.

---

### 2. Install Using kubectl

For direct installation without Helm, use the Kubernetes manifests:
```bash
kubectl apply --namespace otel-demo -f https://raw.githubusercontent.com/open-telemetry/opentelemetry-demo/main/kubernetes/opentelemetry-demo.yaml
```

#### Notes:
- Upgrading these manifests is not supported. To upgrade, delete the existing resources and reinstall the updated version.
- These manifests are generated from the Helm chart and provided for convenience. Using the Helm chart is recommended.

---

## Exposing the Demo Application

To interact with the demo application, you must expose its services outside the Kubernetes cluster. Options include:

### Option 1: Expose Services Using `kubectl port-forward`
To expose the `frontendproxy` service, use:
```bash
kubectl port-forward svc/my-otel-demo-frontendproxy 8080:8080
```

With port-forwarding enabled, access the following URLs:
- **Web Store:** [http://localhost:8080/](http://localhost:8080/)
- **Grafana:** [http://localhost:8080/grafana/](http://localhost:8080/grafana/)
- **Load Generator UI:** [http://localhost:8080/loadgen/](http://localhost:8080/loadgen/)
- **Jaeger UI:** [http://localhost:8080/jaeger/ui/](http://localhost:8080/jaeger/ui/)
- **Flagd Configurator UI:** [http://localhost:8080/feature](http://localhost:8080/feature)

#### Note:
`kubectl port-forward` requires an active terminal session. Use `Ctrl-C` to terminate the process.

### Option 2: Expose Using Service or Ingress Configurations

#### Ingress Resource Configuration
For the `frontendproxy` component, configure ingress resources in your `values` file:
```yaml
components:
  frontendProxy:
    ingress:
      enabled: true
      annotations: {}
      hosts:
        - host: otel-demo.my-domain.com
          paths:
            - path: /
              pathType: Prefix
              port: 8080
```
Ensure your cluster supports ingress resources.

#### Service Type Configuration
To configure the `frontendproxy` as a `LoadBalancer`, specify:
```yaml
components:
  frontendProxy:
    service:
      type: LoadBalancer
```

---

## Advanced Configurations

### Browser Telemetry
Specify the OpenTelemetry Collector endpoint in the `frontend` component:
```yaml
components:
  frontend:
    envOverrides:
      - name: PUBLIC_OTEL_EXPORTER_OTLP_TRACES_ENDPOINT
        value: http://otel-demo.my-domain.com/otlp-http/v1/traces
```

### Bring Your Own Backend
To use an existing observability backend, update the OpenTelemetry Collector configuration:
```yaml
opentelemetry-collector:
  config:
    exporters:
      otlphttp/example:
        endpoint: <your-endpoint-url>

    service:
      pipelines:
        traces:
          exporters: [spanmetrics, otlphttp/example]
```
Ensure the `spanmetrics` exporter remains in the pipeline to avoid errors.

#### Custom Helm Installation with Values File
Use the following command to install with a custom values file:
```bash
helm install my-otel-demo open-telemetry/opentelemetry-demo --values my-values-file.yaml
```

---

## Feedback
If you have any questions or issues, visit [OpenTelemetry Documentation](https://opentelemetry.io/docs/demo/kubernetes-deployment/).

For more, visit my blog: [docs.ahmadraza.in](https://docs.ahmadraza.in).
