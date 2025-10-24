# Values needs to be change in the values file

## values-otel.yaml file
```
endpoint: http://loki-gateway.test.svc.cluster.local:80/loki/api/v1/push
# replace the namespace where loki is deploying apart from that everything need to be same
```

## values-prometheus.yaml
```
storageClassName: gp2    # specify your storage class
accessModes: ["ReadWriteOnce"]
resources:
requests:
storage: 5Gi    # choose your storage size
```
```
# Remote write metrics to Mimir Query Frontend
remoteWrite:
- url: http://mimir-distributor.test.svc.cluster.local:8080/api/v1/push      # Replace with your Mimir distributor URL/namespace
headers:
X-Scope-OrgID: tenant1   # Replace with your tenant ID/name 
# the name we have added will be used by mimir while processing metrics
```
<img width="1511" height="199" alt="image" src="https://github.com/user-attachments/assets/fd17aabc-c4da-4996-af42-4943a806f287" />

