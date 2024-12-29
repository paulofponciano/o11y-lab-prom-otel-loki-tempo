# OpenTelemetry Operator

<p align="center">
  <a href="https://opentelemetry.io/">
    <img src="https://opentelemetry.io/img/logos/opentelemetry-horizontal-color.svg" alt="OpenTelemetry logo" height="180">
  </a>
</p>

Repo: https://github.com/open-telemetry/opentelemetry-operator

## Deploy with Helm

Repo: https://github.com/open-telemetry/opentelemetry-helm-charts/tree/main/charts/opentelemetry-operator

```sh
helm repo add open-telemetry https://open-telemetry.github.io/opentelemetry-helm-charts
helm repo update
```
```sh
helm install opentelemetry-operator open-telemetry/opentelemetry-operator \
--namespace opentelemetry-operator-system \
--create-namespace \
--set "manager.collectorImage.repository=otel/opentelemetry-collector-k8s" \
--set admissionWebhooks.certManager.enabled=false \
--set admissionWebhooks.autoGenerateCert.enabled=true
```

---