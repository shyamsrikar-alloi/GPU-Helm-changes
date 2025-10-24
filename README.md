# Values needs to be change in the values file

## values-otel.yaml file
```
endpoint: http://loki-gateway.test.svc.cluster.local:80/loki/api/v1/push
# replace the namespace where loki is deploying apart from that everything need to be same
```

## values-prometheus.yaml file
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
- url: http://mimir-distributor.test.svc.cluster.local:8080/api/v1/push      # Replace with your Mimir distributor namespace, url will be same change only namespace
headers:
X-Scope-OrgID: tenant1   # Replace with your tenant ID/name 
# the name we have added will be used by mimir while processing metrics
```
Example:
<img width="1511" height="199" alt="image" src="https://github.com/user-attachments/assets/fd17aabc-c4da-4996-af42-4943a806f287" />


## values-loki.yaml file

```
  # Storage configuration
  storage:
    type: s3
    bucketNames:
      chunks: gpu-loki-chunks
      ruler: gpu-loki-ruler          # Change to your S3 bucket names
      admin: gpu-loki-admin
    s3:
      region: us-west-2           # Change to your region
      s3ForcePathStyle: false
      insecure: false
```

```
# Write component configuration (for SimpleScalable mode)
write:
  replicas: 1
  extraEnv:
    - name: AWS_REGION    
      value: us-west-2   # Change to your region
    - name: AWS_WEB_IDENTITY_TOKEN_FILE
      value: /var/run/secrets/eks.amazonaws.com/serviceaccount/token
    - name: AWS_ROLE_ARN
      value: arn:aws:iam::account_num:role/loki-irsa-role    # Change to your IAM role ARN
  
  # Persistence for write pods
  persistence:
    enabled: true
    size: 3Gi    # Choose your storage size
    storageClass: "gp2"  # Leave empty for default, or specify your storage class
```

```
read:
  replicas: 1
  extraEnv:
    - name: AWS_REGION
      value: us-west-2
    - name: AWS_WEB_IDENTITY_TOKEN_FILE
      value: /var/run/secrets/eks.amazonaws.com/serviceaccount/token
    - name: AWS_ROLE_ARN
      value: arn:aws:iam::account_num:role/loki-irsa-role
  
  # Persistence for read pods
  persistence:
    enabled: true
    size: 3Gi
    storageClass: "gp2"
```

```
backend:
  replicas: 1
  extraEnv:
    - name: AWS_REGION
      value: us-west-2
    - name: AWS_WEB_IDENTITY_TOKEN_FILE
      value: /var/run/secrets/eks.amazonaws.com/serviceaccount/token
    - name: AWS_ROLE_ARN
      value: arn:aws:iam::account_num:role/loki-irsa-role   # replace account id
  
  # Persistence for backend pods
  persistence:
    enabled: true
    size: 3Gi
    storageClass: "gp2"
```

```
# Service Account
serviceAccount:
  create: false
  name: loki-irsa-sa    # Specify your existing ServiceAccount name
  annotations:
    eks.amazonaws.com/role-arn: arn:aws:iam::account_num:role/loki-irsa-role
```


## values-mimir.yaml

```
# Namespace where Mimir will be deployed
namespace: test   # Change to your desired namespace
```

```
# ServiceAccount + IRSA configuration
serviceAccount:
  create: false                 # Use existing service account
  name: mimir-irsa-sa
  annotations:
    eks.amazonaws.com/role-arn: arn:aws:iam::aws_account_number:role/mimir-irsa-role

# Core Mimir configuration (YAML inside ConfigMap)
mimir:
  structuredConfig:
    common:
      storage:
        backend: s3
        s3:
          bucket_name: gpu-mimir
          endpoint: s3.us-west-2.amazonaws.com
          region: us-west-2
          insecure: false
```
#### in the below modify bucket name and region
```
    # TSDB block storage configuration
    blocks_storage:
      backend: s3
      bucket_store:
        sync_dir: /data/tsdb-sync
      s3:
        bucket_name: gpu-mimir
        region: us-west-2

    # Ruler configuration (stores rule files in S3)
    ruler_storage:
      backend: s3
      storage_prefix: rules
      s3:
        bucket_name: gpu-mimir
        region: us-west-2

    # Alertmanager configuration (stores silences, alerts)
    alertmanager_storage:
      backend: s3
      storage_prefix: alertmanager
      s3:
        bucket_name: gpu-mimir
        region: us-west-2
```

```
alertmanager:
  replicas: 1
  persistentVolume:   #choose size and storage class
    enabled: true
    size: 1Gi
    storageClass: gp2
    accessModes:
      - ReadWriteOnce
```
```
ingester:
  replicas: 2    #minimum 2 required for processing
  persistentVolume:
    enabled: true
    size: 4Gi   #choose the size
    storageClass: gp2
    accessModes:
      - ReadWriteOnce
```

```
store_gateway:
  replicas: 1
  persistentVolume:
    enabled: true
    size: 4Gi   # choose size and class
    storageClass: gp2
    accessModes:
      - ReadWriteOnce
```

```
compactor:
  replicas: 1
  persistentVolume:
    enabled: true
    size: 4Gi  # choose storage and size
    storageClass: gp2
    accessModes:
      - ReadWriteOnce
```

```
minio:     # must be false because we are using s3
  enabled: false
```
