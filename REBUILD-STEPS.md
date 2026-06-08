# Rebuild Steps After Cluster Deletion

## Step 1 - Start EC2 Instance

Login:

```bash
ssh -i <key.pem> ec2-user@<public-ip>
```

Verify tools:

```bash
kubectl version --client
eksctl version
helm version
docker --version
git --version
python3 --version
```

---

## Step 2 - Navigate to Project

```bash
cd ~/assessment/Shine-GitOps-Project
```

Update repository:

```bash
git pull origin main
```

---

## Step 3 - Create EKS Cluster

```bash
eksctl create cluster \
--name shine-cluster \
--region us-east-1 \
--nodes 3
```

Verify:

```bash
kubectl get nodes
```

Expected:

```bash
3 Ready nodes
```

---

## Step 4 - Create Namespace

```bash
kubectl create namespace retail
```

Verify:

```bash
kubectl get ns
```

---

## Step 5 - Install ArgoCD

```bash
kubectl create namespace argocd
```

```bash
kubectl apply -n argocd \
-f https://raw.githubusercontent.com/argoproj/argo-cd/stable/manifests/install.yaml
```

Wait:

```bash
kubectl get pods -n argocd
```

All pods should be Running.

---

## Step 6 - Access ArgoCD

Get password:

```bash
kubectl -n argocd get secret argocd-initial-admin-secret \
-o jsonpath="{.data.password}" | base64 -d
```

Port forward:

```bash
kubectl port-forward svc/argocd-server \
-n argocd 8080:443
```

Login:

```text
Username: admin
Password: <output above>
```

---

## Step 7 - Deploy GitOps Application

Apply ArgoCD Application manifest:

```bash
kubectl apply -f application.yaml
```

Verify:

```bash
kubectl get applications -n argocd
```

Expected:

```text
Healthy
Synced
```

---

## Step 8 - Verify Application

```bash
kubectl get pods -n retail
```

```bash
kubectl get svc -n retail
```

---

## Step 9 - Install Datadog

Add repo:

```bash
helm repo add datadog https://helm.datadoghq.com
helm repo update
```

Create namespace:

```bash
kubectl create namespace datadog
```

Create secret:

```bash
kubectl create secret generic datadog-secret \
-n datadog \
--from-literal=api-key=<DATADOG_API_KEY>
```

Install:

```bash
helm install datadog-agent datadog/datadog \
-n datadog \
-f datadog-values.yaml
```

Verify:

```bash
kubectl get pods -n datadog
```

---

## Step 10 - Verify Monitoring

Check Datadog:

* Infrastructure
* Kubernetes
* Dashboards
* Monitors

Expected:

* CPU Alert
* Memory Alert
* Pod Failure Alert
* Application Availability Alert

---

## Step 11 - AI Agent

Install package if needed:

```bash
pip3 install openai
```

Export OpenRouter key:

```bash
export OPENROUTER_API_KEY=<OPENROUTER_KEY>
```

Run:

```bash
python3 ai-agent.py
```

Optional:

```bash
python3 aiops-comparison.py
```

---

## Step 12 - Final Verification

Check:

```bash
kubectl get nodes
```

```bash
kubectl get pods -A
```

```bash
kubectl get applications -n argocd
```

```bash
kubectl get pods -n datadog
```

Everything should be:

* Running
* Healthy
* Synced
* Monitored

Assessment Ready.
