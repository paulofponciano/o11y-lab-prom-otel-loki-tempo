![AWS](https://img.shields.io/badge/AWS-%23FF9900.svg?style=for-the-badge&logo=amazon-aws&logoColor=white)
![Kubernetes](https://img.shields.io/badge/kubernetes-%23326ce5.svg?style=for-the-badge&logo=kubernetes&logoColor=white)

# o11y Lab: OpenTelemetry, Prometheus, Loki, Tempo and Grafana 📊

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

Custom S3 Storage config (values.yaml):

```sh
config: |
  compactor:
    shared_store: s3
```

```sh
  schemaConfig:
    configs:
    - from: "2020-09-07"
      store: boltdb-shipper
      object_store: s3
      schema: v11
      index:
        prefix: loki_index_
        period: 24h
```

```sh
  storageConfig:
    boltdb_shipper:
      #shared_store: filesystem
      shared_store: s3
      active_index_directory: /var/loki/index
      cache_location: /var/loki/cache
      cache_ttl: 24h
    # filesystem:
    #   directory: /var/loki/chunks
    aws:
      region: <region>
      bucketnames: <bucket-name>
      s3forcepathstyle: false
```

```sh
serviceAccount:
  create: true
  name: loki-sa
  annotations:
    eks.amazonaws.com/role-arn: "arn:aws:iam::<ACCOUNT>:role/CustomRoleTempoandLokionK8s"
```

AWS IAM (Loki and Tempo):

- IAM Policy
```sh
{
    "Version": "2012-10-17",
    "Statement": [
        {
            "Sid": "AllowLokiandTempoBucketonK8s",
            "Effect": "Allow",
            "Action": [
                "s3:PutObject",
                "s3:GetObject",
                "s3:AbortMultipartUpload",
                "s3:ListBucket",
                "s3:DeleteObject",
                "s3:GetObjectVersion",
                "s3:ListMultipartUploadParts"
            ],
            "Resource": [
                "arn:aws:s3:::<loki-bucket-name>/*",
                "arn:aws:s3:::<loki-bucket-name>",
                "arn:aws:s3:::<tempo-bucket-name>/*",
                "arn:aws:s3:::<tempo-bucket-name>"
            ]
        }
    ]
}
```

- IAM Role
```sh
{
    "Version": "2012-10-17",
    "Statement": [
        {
            "Effect": "Allow",
            "Principal": {
                "Federated": "arn:aws:iam::<ACCOUNT>:oidc-provider/oidc.eks.<REGION>.amazonaws.com/id/<ID>"
            },
            "Action": "sts:AssumeRoleWithWebIdentity",
            "Condition": {
                "StringEquals": {
                    "oidc.eks.<REGION>.amazonaws.com/id/<ID>:sub": [
                        "system:serviceaccount:o11y:loki-sa",
                        "system:serviceaccount:o11y:tempo-sa"
                    ]
                }
            }
        }
    ]
}
```

## Tempo

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
* An additional disk (EBS) will be attached to the minio's Volume Claim

Custom S3 Storage config and Otel (values.yaml):

```sh
otlp:
  grpc:
    enabled: true

storage:
  trace:
    backend: s3
    s3:
      bucket: <bucket-name>
      endpoint: s3.<region>.amazonaws.com
      forcepathstyle: true
```

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

* In the 'additionalScrapeConfigs' section of the values.yaml file, additional scrapes were added. The datasources are also added to the grafana in the 'additionalDataSources' section