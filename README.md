# o11y Lab: OpenTelemetry, Prometheus, Loki, Tempo and Grafana

## Namespace

Execute:

```sh
kubectl create namespace o11y
```

## Promtail

Execute:

```sh
cd promtail
helm repo add grafana https://grafana.github.io/helm-charts
helm repo update
helm upgrade --install promtail grafana/promtail  --values promtail.yaml -n o11y
```

Installed components:

## Loki

Execute:

```sh
helm install loki grafana/loki-distributed -n o11y
```

Installed components:
* gateway
* ingester
* distributor
* querier
* query-frontend

## Tempo (non-production)

Execute:

```sh
cd tempo
helm install tempo grafana/tempo-distributed --values minio.yaml -n o11y
```

Installed components:
* ingester
* distributor
* querier
* query-frontend
* compactor
* memcached

## Kube Prometheus Stack

Execute:

```sh
cd prometheus-grafana
helm repo add prometheus-community https://prometheus-community.github.io/helm-charts
helm repo update
helm install kube-prometheus-stack prometheus-community/kube-prometheus-stack --values values.yaml -n o11y
kubectl apply -f istio-ingress.yaml
```