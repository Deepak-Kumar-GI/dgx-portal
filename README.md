# 🛠 Kubernetes ServiceAccount Setup Guide (Template Version)

## Long‑Lived Token Method (Legacy / Non‑Expiring)


# Configuration Variables (EDIT FIRST)

Fill these before starting:

| Placeholder | Example Value | What it Represents |
|-----------|--------------|-------------------|
| `<NAMESPACE>` | kube-system | Namespace for ServiceAccount |
| `<SERVICE_ACCOUNT_NAME>` | portal-sa | SA name |
| `<ROLE_NAME>` | portal-role | ClusterRole |
| `<ROLE_BINDING_NAME>` | portal-role-binding | RoleBinding |
| `<SECRET_NAME>` | portal-sa-token | Token secret |
| `<API_SERVER>` | https://192.168.X.X:1234 | Kubernetes API URL |

---

# Step 1 — Create ServiceAccount

### File (example name only)

`serviceaccount.yaml`

```yaml
apiVersion: v1
kind: ServiceAccount
metadata:
  name: <SERVICE_ACCOUNT_NAME>
  namespace: <NAMESPACE>
```
### Example

```yaml
apiVersion: v1
kind: ServiceAccount
metadata:
  name: portal-sa
  namespace: kube-system
```

```bash
kubectl apply -f serviceaccount.yaml
```
Verify:

```bash
kubectl get sa portal-sa -n kube-system
```

---

# Step 2 — Create ClusterRole

### File

`portal-clusterrole.yaml`

```yaml
apiVersion: rbac.authorization.k8s.io/v1
kind: ClusterRole
metadata:
  name: <ROLE_NAME>
rules:
- apiGroups: [""]
  resources: ["pods", "services", "namespaces"]
  verbs: ["get", "list", "watch", "create", "delete", "update", "patch"]

- apiGroups: ["apps"]
  resources: ["deployments"]
  verbs: ["get", "list", "watch", "create", "delete", "update", "patch"]

- apiGroups: ["batch"]
  resources: ["jobs"]
  verbs: ["get", "list", "watch", "create", "delete", "update", "patch"]
```
### Example

```yaml
apiVersion: rbac.authorization.k8s.io/v1
kind: ClusterRole
metadata:
  name: portal-role
rules:
- apiGroups: [""]
  resources: ["pods", "services"]
  verbs: ["get", "list", "watch", "create", "delete", "update", "patch"]

- apiGroups: ["apps"]
  resources: ["deployments"]
  verbs: ["get", "list", "watch", "create", "delete", "update", "patch"]

- apiGroups: ["batch"]
  resources: ["jobs"]
  verbs: ["get", "list", "watch", "create", "delete", "update", "patch"]
```

Apply:

```bash
kubectl apply -f portal-clusterrole.yaml
```

Verify:

```bash
kubectl get clusterrole portal-role
```

---

# Step 3 — Bind Role

### File

`portal-rolebinding.yaml`

```yaml
apiVersion: rbac.authorization.k8s.io/v1
kind: ClusterRoleBinding
metadata:
  name: <ROLE_BINDING_NAME>
subjects:
- kind: ServiceAccount
  name: <SERVICE_ACCOUNT_NAME>
  namespace: <NAMESPACE>
roleRef:
  kind: ClusterRole
  name: <ROLE_NAME>
  apiGroup: rbac.authorization.k8s.io
```
### Example


```yaml
apiVersion: rbac.authorization.k8s.io/v1
kind: ClusterRoleBinding
metadata:
  name: portal-role-binding
subjects:
- kind: ServiceAccount
  name: portal-sa
  namespace: kube-system
roleRef:
  kind: ClusterRole
  name: portal-role
  apiGroup: rbac.authorization.k8s.io
```

Apply:

```bash
kubectl apply -f portal-rolebinding.yaml
```

Verify:

```bash
kubectl get clusterrolebinding portal-role-binding
```

---

# Step 4 — Create Long‑Lived Token Secret

### File

`portal-sa-long-lived-token.yaml`

```yaml
apiVersion: v1
kind: Secret
metadata:
  name: <SECRET_NAME>
  namespace: <NAMESPACE>
  annotations:
    kubernetes.io/service-account.name: <SERVICE_ACCOUNT_NAME>
type: kubernetes.io/service-account-token
```
### Example


```yaml
apiVersion: v1
kind: Secret
metadata:
  name: portal-sa-long-lived-token
  namespace: kube-system
  annotations:
    kubernetes.io/service-account.name: portal-sa
type: kubernetes.io/service-account-token
```

Apply:

```bash
kubectl apply -f portal-sa-long-lived-token.yaml
```

---

# Step 5 — Wait for Token

```bash
kubectl get secret <SECRET_NAME> -n <NAMESPACE> -o yaml
```
### Example
```bash
kubectl get secret portal-sa-long-lived-token -n kube-system -o yaml
```

If empty, wait 5–10 seconds and retry.

---

# Step 6 — Extract Token

```bash
kubectl get secret <SECRET_NAME>   -n <NAMESPACE>   -o jsonpath='{.data.token}' | base64 -d
```
### Example
```bash
kubectl get secret portal-sa-long-lived-token -n kube-system -o jsonpath='{.data.token}' | base64 -d
```

Save this securely.
---

# Step 7 — Test Access

```bash
curl <API_SERVER>/api/v1/namespaces   -H "Authorization: Bearer <TOKEN>"   --insecure
```
### Example
```bash
curl https://https://47.29.133.221:10443/api/v1/namespaces   -H "Authorization: Bearer <TOKEN>"   --insecure
```
---
