# Values needs to be change in the values file

## values-otel.yaml file
```
endpoint: http://loki-gateway.test.svc.cluster.local:80/loki/api/v1/push
# replace the namespace where loki is deploying apart from that everything need to be same
```

## values-prometheus.yaml
1)
```
storageClassName: gp2
# replace storageclass
```

