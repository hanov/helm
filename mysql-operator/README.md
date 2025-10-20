# MySQL Operator

A Kubernetes StatefulSet-based MySQL cluster with 3-node master-slave replication.

## Architecture

- **mysql-0**: Master node (read/write)
- **mysql-1, mysql-2**: Slave nodes (read-only replicas)
- Automatic replication setup using xtrabackup
- GTID-based replication for consistency

## Installation

```bash
# Install in dev namespace
helm install mysql-operator ./mysql-operator -n dev --create-namespace

# Or with custom values
helm install mysql-operator ./mysql-operator -n dev -f custom-values.yaml
```

## Services

- **mysql**: Points to master (mysql-0) for writes
- **mysql-read**: Load-balanced reads across all nodes
- **mysql-headless**: For StatefulSet pod discovery

## Connection

```bash
# Connect to master for writes
mysql -h mysql.dev.svc.cluster.local -u appuser -p

# Connect to read replicas
mysql -h mysql-read.dev.svc.cluster.local -u appuser -p
```

## Configuration

Edit `values.yaml`:

```yaml
mysql:
  rootPassword: "your-secure-password"
  database: "your_database"
  user: "your_user"
  password: "your_password"

persistence:
  size: 20Gi
  storageClass: "local-path"
```

## Monitoring Replication

```bash
# Check replication status on slaves
kubectl exec -it mysql-1 -n dev -- mysql -u root -p -e "SHOW SLAVE STATUS\G"
kubectl exec -it mysql-2 -n dev -- mysql -u root -p -e "SHOW SLAVE STATUS\G"
```

## Scaling

The StatefulSet is configured for 3 replicas. To scale:

```bash
kubectl scale statefulset mysql -n dev --replicas=5
```

## Backup

Use the xtrabackup sidecar or your preferred backup solution.
