# Helm Chart Secrets

This chart **does not** create any Kubernetes Secrets. You must create all required secrets before installing the chart.

## Required Secrets

### 1. Database Secret

**Used by:** web, worker, backup

**Create with:**

```bash
kubectl create secret generic <RELEASE>-db \
  --from-literal=password=<POSTGRES_PASSWORD> \
  -n <NAMESPACE>
```

**Required values.yaml fields:**

```yaml
postgresql:
  passwordSecret: "<RELEASE>-db"
  passwordSecretKey: "password"
```

---

### 2. Application Secret (SECRET_KEY_BASE)

**Used by:** web, worker

**Generate a key:**

```bash
openssl rand -hex 64
```

**Create with:**

```bash
kubectl create secret generic <RELEASE>-app-secret \
  --from-literal=secret-key-base=<YOUR_GENERATED_KEY> \
  -n <NAMESPACE>
```

**Required values.yaml fields:**

```yaml
app_secret:
  secretKeyBase:
    secretName: "<RELEASE>-app-secret"
    secretKey: "secret-key-base"
```

---

### 3. Redis Secret (only if using bundled Redis with password)

**Used by:** redis, web, worker

**Create with:**

```bash
kubectl create secret generic <RELEASE>-redis-secret \
  --from-literal=REDIS_PASSWORD=<REDIS_PASSWORD> \
  -n <NAMESPACE>
```

**Required values.yaml fields:**

```yaml
redis:
  passwordSecret: "<RELEASE>-redis-secret"
  passwordSecretKey: "REDIS_PASSWORD"
```

---

### 4. Backup Database Secret

**Used by:** backup CronJob

**Create with:**

```bash
kubectl create secret generic <RELEASE>-backup-db \
  --from-literal=password=<POSTGRES_PASSWORD> \
  -n <NAMESPACE>
```

**Required values.yaml fields:**

```yaml
backup:
  passwordSecret: "<RELEASE>-backup-db"
  passwordSecretKey: "password"
```

---

## Example: Create All Secrets at Once

```bash
RELEASE=sure
NAMESPACE=sure

# Database password
kubectl create secret generic ${RELEASE}-pgdb-credentials \
  --from-literal=password=15917c652ae8633991625eeda9c0d33e \
  -n ${NAMESPACE}

# Application secret key (generate your own with openssl rand -hex 64)
kubectl create secret generic ${RELEASE}-app-secret \
  --from-literal=secret-key-base=$(openssl rand -hex 64) \
  -n ${NAMESPACE}

# Redis password (only if using bundled Redis)
kubectl create secret generic ${RELEASE}-redis-secret \
  --from-literal=REDIS_PASSWORD=change_me_in_production \
  -n ${NAMESPACE}

# Backup database password
kubectl create secret generic ${RELEASE}-backup-db \
  --from-literal=password=change_me_in_production \
  -n ${NAMESPACE}
```

## Verification

After creating the secrets, verify they exist:

```bash
kubectl get secrets -n <NAMESPACE> | grep <RELEASE>
```

You should see:
- `<RELEASE>-db`
- `<RELEASE>-app-secret`
- `<RELEASE>-redis-secret` (if using bundled Redis)
- `<RELEASE>-backup-db` (if backup is enabled)
