
# `oc` Client Power Course (40‑Minute Fast Track) — Commands, Patterns, and One‑Liners

> Audience: Kubernetes/OpenShift engineers (CKA/CKAD level).  
> Goal: Become productive with `oc`, JSON/YAML querying (`jq`, `yq`), and day‑2 ops in 40 minutes.

---

1. **Warm‑up & Setup (3 min)** — KUBECONFIG, contexts, login, projects.
2. **Core CRUD & Introspection (10 min)** — get/describe/logs/exec, create/apply/patch/delete.
3. **Workloads & Networking (8 min)** — Deployments, Services, Routes, rollouts, scaling, port‑forward.
4. **Config & Security (6 min)** — ConfigMaps/Secrets, ServiceAccounts, RBAC.
5. **Search & Reporting (8 min)** — `-o jsonpath`, `jq`, `yq` patterns and fleet one‑liners.
6. **Validation & Troubleshooting (5 min)** — `oc explain`, `--dry-run`, events, `oc adm` helpers.

---

## 1) Environment & Setup

### 1.1 Install & Version
```bash
# Verify client
oc version --client
# Or full client/server when logged-in
oc version
```

### 1.2 KUBECONFIG & Contexts
```bash
export KUBECONFIG=$HOME/.kube/config   # (default path)
# Merge another kubeconfig (e.g., from a download)
KUBECONFIG=$HOME/.kube/config:/tmp/kubeconfig oc config view --flatten > /tmp/merged
mv /tmp/merged $HOME/.kube/config

# View/set current context & namespace (project)
oc config get-contexts
oc config current-context
oc config set-context --current --namespace myproject    # set default namespace
```

### 1.3 Login / Logout
```bash
# Web console shows your login command; typical patterns:
oc login https://api.cluster.example.com:6443 --username=myuser --password='****'
# or with token
oc login --token=sha256~... --server=https://api.cluster.example.com:6443

oc whoami
oc projects
oc project myproject     # switch namespace
oc logout
```

> **Tip:** On OpenShift, a “project” ≈ a Kubernetes namespace with extra policy defaults.

---

## 2) Core Resource Operations (CRUD)

### 2.1 Discover, List, and Describe
```bash
oc api-resources                 # all kinds
oc get all                       # pods, svc, deploy, etc. in current ns
oc get pods -A                   # across all namespaces
oc get deploy,rs,po              # multiple kinds
oc get po -l app=myapp           # by labels
oc describe pod <pod>
oc get po -o wide
oc get po -o json | jq '.items[].metadata.name'
oc get po -o yaml | yq '.items[].spec.containers[].image'
```

### 2.2 Create / Apply / Replace / Delete
```bash
# Create from file or stdin
oc create -f app.yaml
cat app.yaml | oc apply -f -     # apply (idempotent)
oc replace -f app.yaml           # replace full object
oc delete -f app.yaml
oc delete deploy/myapp

# Generate a skeleton manifest then edit:
oc create deploy myapp --image=registry.access.redhat.com/ubi9/ubi --dry-run=client -o yaml > deploy.yaml
```

### 2.3 Patch (JSON & strategic merge)
```bash
# Strategic merge (simple key patch)
oc patch deploy myapp -p '{"spec":{"replicas":3}}'
# JSON patch (RFC 6902)
oc patch deploy myapp --type='json' -p='[{"op":"replace","path":"/spec/replicas","value":2}]'
```

### 2.4 Explain & Validate
```bash
oc explain deploy.spec.template.spec.containers
oc apply -f app.yaml --server-dry-run     # validate against API server
oc apply -f app.yaml --dry-run=client -o yaml  # client-only validation and render
```

---

## 3) Logs, Exec, Port‑Forward, Rollouts

```bash
# Logs
oc logs deploy/myapp                    # controller logs from latest pod
oc logs -f po/<pod>                     # tail
oc logs po/<pod> -c <container> --since=1h

# Exec
oc exec -it po/<pod> -- /bin/sh
oc rsh <pod>                            # OpenShift remote shell shortcut

# Port-forward (local 8080 → svc 80)
oc port-forward svc/myapp 8080:80

# Rollout
oc rollout status deploy/myapp
oc rollout restart deploy/myapp
oc rollout history deploy/myapp
oc rollout undo deploy/myapp --to-revision=2

# Scale
oc scale deploy/myapp --replicas=4
```

---

## 4) Quick Creates: Workloads & Networking

### 4.1 Minimal Deployment + Service (YAML)
```yaml
# file: app.yaml
apiVersion: apps/v1
kind: Deployment
metadata:
  name: myapp
spec:
  replicas: 2
  selector:
    matchLabels: { app: myapp }
  template:
    metadata:
      labels: { app: myapp }
    spec:
      containers:
      - name: web
        image: registry.access.redhat.com/ubi9/httpd-24:latest
        ports:
        - containerPort: 8080
---
apiVersion: v1
kind: Service
metadata:
  name: myapp
spec:
  selector: { app: myapp }
  ports:
  - port: 80
    targetPort: 8080
    protocol: TCP
```
```bash
oc apply -f app.yaml
oc get svc myapp
```

### 4.2 OpenShift Route (HTTP)
```bash
# Create a route fronting the Service
oc expose svc/myapp
oc get route myapp -o wide
# Custom hostname & TLS example:
oc create route edge myapp-https --service=myapp --hostname=myapp.apps.example.com
```

---

## 5) Configuration & Secrets

### 5.1 ConfigMaps
```bash
# From literals
oc create configmap app-cfg --from-literal=MODE=prod --from-literal=TIMEOUT=30

# From files/dir
oc create configmap app-files --from-file=config/ --from-file=app.properties=./app.properties

# Mount or env-inject (snippet in Pod spec)
# envFrom:
# - configMapRef: { name: app-cfg }
```

### 5.2 Secrets (Opaque & TLS)
```bash
# Generic secret from literals
oc create secret generic app-sec --from-literal=USER=demo --from-literal=PASS=changeit

# From files
oc create secret generic tls-bundle --from-file=tls.crt=server.crt --from-file=tls.key=server.key

# Reference in Deployment
# envFrom:
# - secretRef: { name: app-sec }
```

### 5.3 ServiceAccounts & RBAC
```bash
oc create sa runtime
oc adm policy add-scc-to-user anyuid -z runtime        # example SCC role for SA
oc create role viewer --verb=get,list,watch --resource=pods
oc create rolebinding viewer-to-runtime --role=viewer --serviceaccount=$(oc project -q):runtime
oc auth can-i list pods --as=system:serviceaccount:$(oc project -q):runtime
```

---

## 6) Storage (PVC Quickstart)

```yaml
# file: pvc.yaml
apiVersion: v1
kind: PersistentVolumeClaim
metadata:
  name: data
spec:
  accessModes: ["ReadWriteOnce"]
  resources: { requests: { storage: 5Gi } }
  storageClassName: gp2-csi # or your cluster default
```
```bash
oc apply -f pvc.yaml
oc get pvc data
```

Mount in a container:
```yaml
# in pod template
volumeMounts:
- name: data
  mountPath: /var/data
volumes:
- name: data
  persistentVolumeClaim:
    claimName: data
```

---

## 7) Templates, New‑App, and Image Streams (OpenShift)

```bash
# From existing image to running app
oc new-app --name hello quay.io/redhattraining/hello-world-nginx

# Process a Template (if you use OpenShift Templates)
oc get templates -n openshift | head
oc process -n openshift openshift//mysql-persistent -p MYSQL_USER=user -p MYSQL_PASSWORD=pass -p MYSQL_DATABASE=appdb | oc apply -f -

# ImageStreams (track image tags)
oc get is -A | head
```

---

## 8) Output Tricks: JSONPath, `jq`, and `yq`

### 8.1 JSONPath (built‑in)
```bash
# All pod names
oc get po -o jsonpath='{.items[*].metadata.name}'
# Pod: container images (newline-separated)
oc get po -o jsonpath='{range .items[*]}{.metadata.name}{"  "}{range .spec.containers[*]}{.image}{"
"}{end}{end}'
# Nodes by role
oc get nodes -o jsonpath='{range .items[*]}{.metadata.name}{" "}{.metadata.labels."node-role\.kubernetes\.io/"}{"
"}{end}'
```

### 8.2 `jq` (JSON)
```bash
# Count pods not ready
oc get po -o json | jq '[.items[] | select(any(.status.containerStatuses[]?; .ready==false))] | length'

# List pods with restarts > 3
oc get po -o json | jq -r '.items[] | select([.status.containerStatuses[]?.restartCount] | add // 0 > 3) | .metadata.name'

# Map: deployment → image
oc get deploy -o json | jq -r '.items[] | "\(.metadata.name)	\(.spec.template.spec.containers[0].image)"'

# Per‑namespace CPU requests (sum)
oc get po -A -o json | jq -r '
  .items
  | group_by(.metadata.namespace)
  | map({ns: .[0].metadata.namespace,
         millicpu: (map(.spec.containers[]?.resources.requests.cpu // "0") | map(gsub("m$";"") | tonumber) | add)})
  | .[] | "\(.ns)	\(.millicpu)m"'
```

### 8.3 `yq` (YAML)
```bash
# All container images (YAML → YAML)
oc get po -o yaml | yq '.items[].spec.containers[].image'

# Filter by label using yq’s expression on JSONized YAML
oc get po -o yaml | yq '.items[] | select(.metadata.labels.app == "myapp") | .metadata.name'

# Convert YAML to JSON for `jq` downstream
oc get po -o yaml | yq -o=json | jq '.items | length'
```

> **Install helpers**
```bash
# macOS (brew)
brew install jq yq
# RHEL/Fedora
sudo dnf install -y jq
# yq (binary)
sudo curl -L https://github.com/mikefarah/yq/releases/latest/download/yq_linux_amd64 -o /usr/local/bin/yq && sudo chmod +x /usr/local/bin/yq
```

---

## 9) Fleet & SRE One‑Liners

```bash
# All pods in CrashLoopBackOff across all namespaces
oc get po -A --field-selector=status.phase!=Running -o json | jq -r '.items[] | select(any(.status.containerStatuses[]?; .state.waiting?.reason=="CrashLoopBackOff")) | "\(.metadata.namespace)/\(.metadata.name)"'

# Nodes with disk pressure
oc get nodes -o json | jq -r '.items[] | select(.status.conditions[] | select(.type=="DiskPressure" and .status=="True")) | .metadata.name'

# Top 20 largest images by size (requires CRI info via Node status.images on OCP 4+)
oc get nodes -o json | jq -r '
  .items[].status.images[]? | {size: (.sizeBytes // 0), names: .names}
' | jq -s 'sort_by(-.size) | .[0:20]'
```

---

## 10) Events & Troubleshooting

```bash
# Recent events in namespace (sorted by timestamp)
oc get events --sort-by=.lastTimestamp | tail -n 50

# Describe resources for quick hints
oc describe pod <pod>
oc describe deploy <name>

# Scheduler placement: which node & why
oc get pod <pod> -o wide
oc describe pod <pod> | sed -n '/Events/,$p'

# Cluster admin helpers
oc adm top nodes
oc adm top pods -A
oc adm node-logs <node>                # gather kubelet logs
oc adm must-gather                     # collect cluster diag bundle
```

---

## 11) Cleanup Patterns

```bash
# Delete everything with a label
oc delete all -l app=myapp

# Bulk delete Evicted pods
oc get po -A --field-selector=status.phase==Failed -o name | xargs -r oc delete

# Prune old completed Jobs
oc get jobs -A -o json | jq -r '.items[] | select(.status.succeeded==1) | "job/\(.metadata.name) -n \(.metadata.namespace)"'   | xargs -r -n3 oc delete -n
```

---

## 12) Practice Lab (Hands‑On 10–15 min)

1. **Create** `myapp` Deployment + Service from the YAML above.  
2. **Expose** it via a **Route** and open the host in a browser.  
3. **Scale** to 4 replicas and **watch rollout** to completion.  
4. **Inject config** using a ConfigMap and restart rollout.  
5. **Find** all pods and their images using `jq` and `jsonpath`.  
6. **Troubleshoot**: break the image tag, observe CrashLoopBackOff, fix, and roll forward.  

---

## 13) Reference Cheatsheet

- **List & describe**: `oc get …`, `oc describe …`, `-A`, `-o (yaml|json|jsonpath=…)`  
- **Change ns**: `oc project <ns>`  
- **Create/apply**: `oc create -f`, `oc apply -f`, `--dry-run(=client|server)`  
- **Patch**: `oc patch … -p …` (strategic/json)  
- **Logs/exec**: `oc logs [-f]`, `oc exec -it … -- …`, `oc rsh`  
- **Rollout/scale**: `oc rollout status|restart|undo`, `oc scale`  
- **Network**: `oc expose svc/<name>`, `oc port-forward svc/<name> 8080:80`  
- **Security**: `oc create sa`, `oc adm policy …`, `oc auth can-i …`  
- **Config**: `oc create configmap|secret`, `envFrom`, mounts  
- **Explain**: `oc explain <kind>[.field…]`  
- **Admin**: `oc adm top`, `oc adm must-gather`

---

## 14) Appendix — Valid Minimal Manifests

### Pod (single container)
```yaml
apiVersion: v1
kind: Pod
metadata: { name: hello }
spec:
  containers:
  - name: hello
    image: registry.access.redhat.com/ubi9/ubi-minimal
    command: ["bash","-lc","python3 -m http.server 8080"]
    ports: [{ containerPort: 8080 }]
```

### ServiceAccount + RBAC binding (viewer)
```yaml
apiVersion: v1
kind: ServiceAccount
metadata: { name: viewer-sa }
---
apiVersion: rbac.authorization.k8s.io/v1
kind: Role
metadata: { name: viewer }
rules:
- apiGroups: [""]
  resources: ["pods"]
  verbs: ["get","list","watch"]
---
apiVersion: rbac.authorization.k8s.io/v1
kind: RoleBinding
metadata: { name: viewer-binding }
subjects:
- kind: ServiceAccount
  name: viewer-sa
roleRef:
  kind: Role
  name: viewer
  apiGroup: rbac.authorization.k8s.io
```

---

## 15) Bibliography (APA Style)

- Red Hat. (2024). *OpenShift CLI (`oc`) reference*. Red Hat OpenShift Documentation. https://docs.openshift.com/
- Red Hat. (2024). *Networking — Services and Routes in OpenShift*. Red Hat OpenShift Documentation. https://docs.openshift.com/
- The Kubernetes Authors. (2024). *Kubernetes API concepts & `kubectl` conventions*. Kubernetes Documentation. https://kubernetes.io/docs/
- Mike Farah. (2024). *yq — a lightweight and portable command-line YAML processor*. https://mikefarah.gitbook.io/yq/
- jq contributors. (2024). *jq Manual*. https://stedolan.github.io/jq/manual/
- Red Hat. (2024). *Security & RBAC on OpenShift (SCCs, ServiceAccounts, Policies)*. Red Hat OpenShift Documentation. https://docs.openshift.com/

> **Note:** Commands and APIs are compatible with OpenShift 4.x and upstream Kubernetes 1.27+ patterns. Always check your cluster version for minor differences.
