# Create Cloud Native PG connection secret

### Initialize Cluster

```yaml
#k8s/apps/<app>/cnpg.yaml
apiVersion: postgresql.cnpg.io/v1
kind: Cluster
metadata:
  name: <cluster name>
  namespace: <app namespace>
spec:
  instances: 1
  affinity:
    nodeSelector:
      topology.kubernetes.io/zone: proxmox
  managed:
    services:
      disabledDefaultServices: [ "ro", "r" ]
      additional:
        - selectorType: rw
          updateStrategy: patch
  storage:
    size: 1G
    pvcTemplate:
      storageClassName: proxmox-csi
      volumeName: pv-<volume name>
      accessModes:
        - ReadWriteOnce  bootstrap:
  bootstrap:
    initdb:
      database: <database name>
      owner: <owner>
```

```base
$ kubectl apply -f cnpg.yaml
```

### Create connection info sealed secret 

```bash
$ export APP_NS=<app namespace> 
$ kubectl get secrets <cluster name>-app -n cnpg-database -o json | jq 'del(.metadata["namespace","creationTimestamp","resourceVersion","selfLink","uid","annotations","labels","ownerReferences"])' | \
      kubeseal --controller-namespace=sealed-secrets -n ${APP_NS} \
      --format=yaml - > cnpg-config.yaml
```

### Add new secret into `cnpg.yaml`

```yaml
  bootstrap:
    initdb:
      database: <database name>
      owner: <owner>
      secret:
        name: <cluster name>-app
```

> **_NOTE_:** Future bootstraps will use the same configuration without creating new passwords

### Deployment

```yaml
apiVersion: apps/v1
kind: Deployment
metadata:
  name: app
  namespace: app
  labels:
    app: app
spec:
  replicas: 1
  selector:
    matchLabels:
      app: app
  template:
  . . .
    spec:
    . . .
      containers:
      . . .
          env:
          - name: DB_HOST
            value: <cluster name>-rw.cnpg-database
          - name: DB_PORT
            valueFrom:
              secretKeyRef:
                key: port
                name: <cluster name>-app
          - name: DB_USER
            valueFrom:
              secretKeyRef:
                key: username
                name: <cluster name>-app
          - name: DB_TYPE
            value: postgres
          - name: DB_NAME
            value: <database name>
          - name: DB_PASS
            valueFrom:
              secretKeyRef:
                key: password
                name: <cluster name>-app
```