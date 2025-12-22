# 🚀 Kubernetes Helm Deployment on AWS EKS

This guide shows **step-by-step deployment of Helm charts** (`project-1` and `project-2`) on an AWS EKS cluster. All commands and outputs are included.

---

## 1️⃣ Switch to root user

```bash
sudo su -
```

---

## 2️⃣ Install kubectl

```bash
curl -LO "https://dl.k8s.io/release/$(curl -L -s https://dl.k8s.io/release/stable.txt)/bin/linux/amd64/kubectl"
chmod +x kubectl
mv kubectl /usr/local/bin/kubectl
kubectl version --client
```

**Output:**

```
Client Version: v1.35.0
Kustomize Version: v5.7.1
```

---

## 3️⃣ Configure AWS CLI

```bash
aws configure
```

**Example input:**

```
AWS Access Key ID [None]: <your-access-key>
AWS Secret Access Key [None]: <your-secret-key>
Default region name [None]: us-east-1
Default output format [None]:
```

---

## 4️⃣ Connect to EKS cluster

```bash
aws eks update-kubeconfig --region us-east-1 --name cloud
kubectl get nodes
```

**Output:**

```
NAME                            STATUS   ROLES    AGE     VERSION
ip-172-31-31-34.ec2.internal    Ready    <none>   4m37s   v1.34.2-eks-ecaa3a6
ip-172-31-73-168.ec2.internal   Ready    <none>   4m38s   v1.34.2-eks-ecaa3a6
```

---

## 5️⃣ Install tree (optional)

```bash
yum install tree -y
```

**Output:**

```
Installed:
  tree-1.8.0-6.amzn2023.0.2.x86_64
Complete!
```

---

## 6️⃣ Install Helm

```bash
curl -fsSL -o get_helm.sh https://raw.githubusercontent.com/helm/helm/main/scripts/get-helm-3
chmod 700 get_helm.sh
./get_helm.sh
helm version
```

**Output:**

```
version.BuildInfo{Version:"v3.19.4", GitCommit:"7cfb6e486dac026202556836bb910c37d847793e", GitTreeState:"clean", GoVersion:"go1.24.11"}
```

---

## 7️⃣ Create and deploy `project-1`

```bash
helm create project-1
cd project-1
tree
helm install release-1 .
kubectl get pods
kubectl get svc
```

**Tree structure:**

```
.
├── Chart.yaml
├── charts
├── templates
│   ├── NOTES.txt
│   ├── _helpers.tpl
│   ├── deployment.yaml
│   ├── hpa.yaml
│   ├── httproute.yaml
│   ├── ingress.yaml
│   ├── service.yaml
│   ├── serviceaccount.yaml
│   └── tests
│       └── test-connection.yaml
└── values.yaml
```

**Pods output:**

```
NAME                                   READY   STATUS    RESTARTS   AGE
release-1-project-1-5b5cb6b467-ldgcq   1/1     Running   0          2m56s
```

**Services output:**

```
NAME                  TYPE        CLUSTER-IP       EXTERNAL-IP   PORT(S)        AGE
release-1-project-1   NodePort    10.100.192.242   <none>        80:30914/TCP   3m30s
```

---

## 8️⃣ Update `project-1` values/templates

```bash
vi values.yaml
vi templates/deployment.yaml
helm upgrade release-1 .
```

**Output:**

```
Release "release-1" has been upgraded. Happy Helming!
```

---

## 9️⃣ Create and deploy `project-2`

```bash
helm create project-2
cd project-2
vi values.yaml
vi templates/deployment.yaml
helm install release-2 .
kubectl get svc
kubectl get nodes -o wide
```

**Services output:**

```
NAME                  TYPE        CLUSTER-IP       EXTERNAL-IP   PORT(S)          AGE
release-2-project-2   NodePort    10.100.212.37    <none>        3000:30316/TCP   32s
```

**Nodes output:**

```
NAME                            STATUS   ROLES    AGE   VERSION               INTERNAL-IP     EXTERNAL-IP     OS-IMAGE                       KERNEL-VERSION                   CONTAINER-RUNTIME
ip-172-31-31-34.ec2.internal    Ready    <none>   26m   v1.34.2-eks-ecaa3a6   172.31.31.34    13.218.118.50   Amazon Linux 2023.9.20251208   6.12.58-82.121.amzn2023.x86_64   containerd://2.1.5
ip-172-31-73-168.ec2.internal   Ready    <none>   26m   v1.34.2-eks-ecaa3a6   172.31.73.168   100.28.225.3    Amazon Linux 2023.9.20251208   6.12.58-82.121.amzn2023.x86_64   containerd://2.1.5
```

---

## 🔟 Verify Helm releases

```bash
helm list -a
```

**Output:**

```
NAME            NAMESPACE       REVISION        UPDATED                                 STATUS          CHART           APP VERSION
release-1       default         3               2025-12-22 07:01:33.901044179 +0000 UTC deployed        project-1-0.1.0 1.16.0     
release-2       default         1               2025-12-22 07:08:04.860024035 +0000 UTC deployed        project-2-0.1.0 1.16.0  
```

---

## 1️⃣1️⃣ Create `argocd` namespace

```bash
kubectl create namespace argocd
```

**Output (if exists):**

```
Error from server (AlreadyExists): namespaces "argocd" already exists
```

---


# 🚀 Argo CD Installation on AWS EKS (Step-by-Step)

---

## 1️⃣ Apply Argo CD manifests

We deploy Argo CD to the `argocd` namespace using the official YAML:

```bash
kubectl apply -n argocd \
  -f https://raw.githubusercontent.com/argoproj/argo-cd/stable/manifests/install.yaml
```

**Output:**

```
customresourcedefinition.apiextensions.k8s.io/applications.argoproj.io created
customresourcedefinition.apiextensions.k8s.io/applicationsets.argoproj.io created
customresourcedefinition.apiextensions.k8s.io/appprojects.argoproj.io created
serviceaccount/argocd-application-controller created
serviceaccount/argocd-applicationset-controller created
serviceaccount/argocd-dex-server created
serviceaccount/argocd-notifications-controller created
serviceaccount/argocd-redis created
serviceaccount/argocd-repo-server created
serviceaccount/argocd-server created
role.rbac.authorization.k8s.io/argocd-application-controller created
role.rbac.authorization.k8s.io/argocd-applicationset-controller created
role.rbac.authorization.k8s.io/argocd-dex-server created
role.rbac.authorization.k8s.io/argocd-notifications-controller created
role.rbac.authorization.k8s.io/argocd-redis created
role.rbac.authorization.k8s.io/argocd-server created
clusterrole.rbac.authorization.k8s.io/argocd-application-controller unchanged
clusterrole.rbac.authorization.k8s.io/argocd-applicationset-controller unchanged
clusterrole.rbac.authorization.k8s.io/argocd-server unchanged
rolebinding.rbac.authorization.k8s.io/argocd-application-controller created
rolebinding.rbac.authorization.k8s.io/argocd-applicationset-controller created
rolebinding.rbac.authorization.k8s.io/argocd-dex-server created
rolebinding.rbac.authorization.k8s.io/argocd-notifications-controller created
rolebinding.rbac.authorization.k8s.io/argocd-redis created
rolebinding.rbac.authorization.k8s.io/argocd-server created
clusterrolebinding.rbac.authorization.k8s.io/argocd-application-controller unchanged
clusterrolebinding.rbac.authorization.k8s.io/argocd-applicationset-controller unchanged
clusterrolebinding.rbac.authorization.k8s.io/argocd-server unchanged
configmap/argocd-cm created
configmap/argocd-cmd-params-cm created
configmap/argocd-gpg-keys-cm created
configmap/argocd-notifications-cm created
configmap/argocd-rbac-cm created
configmap/argocd-ssh-known-hosts-cm created
configmap/argocd-tls-certs-cm created
secret/argocd-notifications-secret created
secret/argocd-secret created
service/argocd-applicationset-controller created
service/argocd-dex-server created
service/argocd-metrics created
service/argocd-notifications-controller-metrics created
service/argocd-redis created
service/argocd-repo-server created
service/argocd-server created
service/argocd-server-metrics created
deployment.apps/argocd-applicationset-controller created
deployment.apps/argocd-dex-server created
deployment.apps/argocd-notifications-controller created
deployment.apps/argocd-redis created
deployment.apps/argocd-repo-server created
deployment.apps/argocd-server created
statefulset.apps/argocd-application-controller created
networkpolicy.networking.k8s.io/argocd-application-controller-network-policy created
networkpolicy.networking.k8s.io/argocd-applicationset-controller-network-policy created
networkpolicy.networking.k8s.io/argocd-dex-server-network-policy created
networkpolicy.networking.k8s.io/argocd-notifications-controller-network-policy created
networkpolicy.networking.k8s.io/argocd-redis-network-policy created
networkpolicy.networking.k8s.io/argocd-repo-server-network-policy created
networkpolicy.networking.k8s.io/argocd-server-network-policy created
```

---

## 2️⃣ Check Argo CD pods

```bash
kubectl get pods -n argocd
```

**Output:**

```
NAME                                                READY   STATUS    RESTARTS   AGE
argocd-application-controller-0                     1/1     Running   0          19s
argocd-applicationset-controller-58dc69fff7-jl7hn   1/1     Running   0          19s
argocd-dex-server-5589bfd977-zzlgq                  1/1     Running   0          19s
argocd-notifications-controller-6fb794bdc8-rb8vp    1/1     Running   0          19s
argocd-redis-69bfd4888d-vdbgs                       1/1     Running   0          19s
argocd-repo-server-878f94c-r9z2c                    0/1     Running   0          19s
argocd-server-7b7957b777-zknvz                      0/1     Running   0          19s
```

> ⚠ `argocd-server` and `repo-server` may take a few seconds to become `Running`.

---

## 3️⃣ Expose Argo CD server via NodePort

Edit the `argocd-server` service:

```bash
kubectl edit svc argocd-server -n argocd
```

* Change `type: ClusterIP` → `type: NodePort`
* Save and exit

Check updated service:

```bash
kubectl get svc -n argocd
```

**Output:**

```
NAME                                      TYPE        CLUSTER-IP       EXTERNAL-IP   PORT(S)                      AGE
argocd-server                             NodePort    10.100.143.208   <none>        80:31157/TCP,443:30337/TCP   106s
```

> ✨ Now Argo CD UI can be accessed on your node IP at port `31157` (HTTP) or `30337` (HTTPS).

---

## 4️⃣ Get Argo CD initial admin password

```bash
kubectl get secret argocd-initial-admin-secret -n argocd -o yaml
```

**Output:**

```yaml
apiVersion: v1
data:
  password: M2F1SzFkOWlZS3UyN3gteQ==
kind: Secret
metadata:
  name: argocd-initial-admin-secret
  namespace: argocd
type: Opaque
```

Decode the password:

```bash
echo "M2F1SzFkOWlZS3UyN3gteQ==" | base64 --decode
```

**Result:**

```
3auK1d9iYKu27x-y
```

> ✅ Use this password with username `admin` to login to Argo CD UI.

---

## 5️⃣ Install Git (required for repo sync)

```bash
yum install git -y
```

**Sample output:**

```
Installed:
  git-2.50.1-1.amzn2023.0.1.x86_64        git-core-2.50.1-1.amzn2023.0.1.x86_64         git-core-doc-2.50.1-1.amzn2023.0.1.noarch   perl-Error-1:0.17029-5.amzn2023.0.2.noarch   perl-File-Find-1.37-477.amzn2023.0.7.noarch  
  perl-Git-2.50.1-1.amzn2023.0.1.noarch   perl-TermReadKey-2.38-9.amzn2023.0.2.x86_64   perl-lib-0.65-477.amzn2023.0.7.x86_64      
Complete!
```

---

## ✅ 6️⃣ Access Argo CD UI

* Open browser: `http://<NodeIP>:31157` or `https://<NodeIP>:30337`
* Login:

  * **Username:** `admin`
  * **Password:** `3auK1d9iYKu27x-y`

---
Perfect! I’ll give you **every single step from scratch to fully automated Argo CD deployment with Helm**, all in order. You can literally copy-paste each command.

---

# 🔥 Full Step-by-Step: Argo CD Automatic Helm Deployment

---

## **1️⃣ Install kubectl & git (if not installed)**

```bash
# Install kubectl
curl -LO "https://dl.k8s.io/release/$(curl -L -s https://dl.k8s.io/release/stable.txt)/bin/linux/amd64/kubectl"
chmod +x kubectl
mv kubectl /usr/local/bin/

# Install git
yum install git -y
```

---

## **2️⃣ Install Argo CD**

```bash
# Create namespace
kubectl create ns argocd

# Apply Argo CD manifests
kubectl apply -n argocd -f https://raw.githubusercontent.com/argoproj/argo-cd/stable/manifests/install.yaml
```

Check pods:

```bash
kubectl get pods -n argocd
```

Expose Argo CD server:

```bash
kubectl edit svc argocd-server -n argocd
# Change type: ClusterIP → NodePort
```

Check service:

```bash
kubectl get svc -n argocd
# Note NodePort (ex: 31157)
```

---

## **3️⃣ Get initial Argo CD admin password**

```bash
kubectl get secret argocd-initial-admin-secret -n argocd -o jsonpath="{.data.password}" | base64 --decode
```

> Use this password with username `admin` to login to Argo CD UI.

---

## **4️⃣ Create your target namespace**

```bash
kubectl create ns hotstar
```

---

## **5️⃣ Prepare Helm chart in Git**

```bash
# Clone your repo
git clone https://github.com/chintu-cloud/kubernetes-Helm.git
cd kubernetes-Helm

# Move your chart (project-2) into repo
mv ../project-2 .

# Check files
cd project-2
ls
# Chart.yaml  charts  templates  values.yaml
```

---

## **6️⃣ Commit and push to Git**

```bash
# Add files
git add .
git commit -m "Add hotstar Helm chart"
git push -u origin main
```

> **Tip:** If Git asks for password, use a GitHub Personal Access Token (PAT).
> **How to create PAT:**
>
> 1. GitHub → Settings → Developer settings → Personal access tokens → Classic
> 2. Click **Generate new token**, name it, select scopes: `repo`, `workflow`, `admin:repo_hook`, `user`, `project`.
> 3. Copy & paste token when prompted.

---

## **7️⃣ Create Argo CD Application YAML**

`deployment.yaml`:

```yaml
apiVersion: argoproj.io/v1alpha1
kind: Application
metadata:
  name: hotstat-helm
  namespace: argocd
spec:
  project: default

  source:
    repoURL: https://github.com/chintu-cloud/kubernetes-Helm.git
    targetRevision: main
    path: project-2
    helm:
      releaseName: hotstat
      valueFiles:
        - values.yaml

  destination:
    server: https://kubernetes.default.svc
    namespace: hotstar

  syncPolicy:
    automated:
      prune: true
      selfHeal: true
```

---

## **8️⃣ Apply the Argo CD Application**

```bash
kubectl apply -f deployment.yaml
```

Check application status:

```bash
kubectl get applications -n argocd
```

Expected output:

```
NAME           PROJECT   SYNC STATUS  HEALTH STATUS  DESTINATION                AGE
hotstat-helm   default   Synced       Healthy        https://kubernetes.default.svc  1m
```

---

## **9️⃣ Verify Kubernetes resources**

```bash
kubectl get all -n hotstar
```

All your Helm chart resources (Deployment, Service, Ingress, etc.) should appear.

---

## **🔟 Optional: Access Argo CD UI**

* Open browser: `http://<NodeIP>:31157`
* Login: `admin` / `<password from step 3>`
* You’ll see the application `hotstat-helm` syncing automatically.

---

## ✅ Summary

1. Install `kubectl` & `git`
2. Install Argo CD
3. Expose Argo CD server & get admin password
4. Create target namespace `hotstar`
5. Push Helm chart to GitHub
6. Create Argo CD Application YAML
7. Apply YAML → automatic deployment via Argo CD
8. Verify resources in `hotstar` namespace
9. Optionally access Argo CD UI

---




