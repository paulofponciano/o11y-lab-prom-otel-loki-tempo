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
helm upgrade --install promtail grafana/promtail  --values values.yaml -n o11y
```

Installed components:

## Loki

Execute:

```sh
cd loki
helm upgrade --install loki grafana/loki-distributed --values values.yaml -n o11y
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
helm upgrade --install tempo grafana/tempo-distributed --values values.yaml -n o11y
```

Installed components:
* ingester
* distributor
* querier
* query-frontend
* compactor
* memcached

** An additional disk (EBS) will be attached to the minio's Volume Claim.

## Kube Prometheus Stack

Execute:

```sh
cd prometheus-grafana
helm repo add prometheus-community https://prometheus-community.github.io/helm-charts
helm repo update
helm upgrade --install kube-prometheus-stack prometheus-community/kube-prometheus-stack --values values.yaml -n o11y
kubectl apply -f istio-ingress-prometheus.yaml
kubectl apply -f istio-ingress-grafana.yaml
```

** In the 'additionalScrapeConfigs' section of the values.yaml file, additional scrapes were added. The datasources are also added to the grafana in the 'additionalDataSources' section.