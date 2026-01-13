🏗️ Project: GitOps-Driven Kubernetes Observability Platform
🎯 What You Will Build

A production-style Kubernetes platform with:

✅ Minikube (local Kubernetes)
✅ GitOps with Argo CD

🔹 PHASE 1: Prerequisites
1️⃣ Verify Tools on Your Machine
kubectl version --client
minikube version
docker version
helm version
git --version


If missing:

sudo apt install docker.io git -y
curl https://raw.githubusercontent.com/helm/helm/main/scripts/get-helm-3 | bash

2️⃣ Start Minikube
minikube start \
  --driver=docker \
  --memory=8192 \
  --cpus=4


Enable addons:

minikube addons enable ingress
minikube addons enable metrics-server


Verify:

kubectl get nodes

🔹 PHASE 2: Install Argo CD (GitOps Engine)
3️⃣ Install Argo CD
kubectl create namespace argocd

kubectl apply -n argocd \
  -f https://raw.githubusercontent.com/argoproj/argo-cd/stable/manifests/install.yaml


Wait:

kubectl get pods -n argocd

4️⃣ Access Argo CD UI
kubectl port-forward svc/argocd-server -n argocd 8080:443


Get admin password:

kubectl get secret argocd-initial-admin-secret -n argocd \
-o jsonpath="{.data.password}" | base64 -d


Login

URL: https://localhost:8080

Username: admin

Password: (from command above)

🔹 STEP 5: Create GitOps Repository (LOCAL)
mkdir -p ~/gitops-homelab
cd ~/gitops-homelab
git init

GitHub disabled password authentication for HTTPS pushes.
You must use either a Personal Access Token (PAT) or SSH keys.
GitOps homelab + Argo CD, SSH is the BEST practice.

1️⃣ Generate SSH key on your CentOS node
ssh-keygen -t ed25519 -C "gitops-homelab"
2️⃣ Copy the public key
cat ~/.ssh/id_ed25519.pub
Copy the full output.
3️⃣ Add SSH key to GitHub

GitHub → Settings
SSH and GPG keys
New SSH key
Paste key
Save

4️⃣ Change Git remote to SSH
git remote remove origin
git remote add origin git@github.com:AlokRajak123/gitops-homelab.git

Verify:
git remote -v

5️⃣ Push again
git push -u origin main



Create structure:

mkdir -p apps/demo-app
mkdir -p argocd/applications
mkdir -p monitoring/kube-prometheus-stack

🔹 STEP 6: Create Demo Application (Kubernetes)
Deployment
cat <<EOF > apps/demo-app/deployment.yaml
apiVersion: apps/v1
kind: Deployment
metadata:
  name: demo-app
spec:
  replicas: 2
  selector:
    matchLabels:
      app: demo-app
  template:
    metadata:
      labels:
        app: demo-app
    spec:
      containers:
      - name: nginx
        image: nginx
        ports:
        - containerPort: 80
EOF

Service
cat <<EOF > apps/demo-app/service.yaml
apiVersion: v1
kind: Service
metadata:
  name: demo-app
spec:
  selector:
    app: demo-app
  ports:
    - port: 80
      targetPort: 80
EOF

🔹 STEP 7: Create Argo CD Application (GitOps)
cat <<EOF > argocd/applications/demo-app.yaml
apiVersion: argoproj.io/v1alpha1
kind: Application
metadata:
  name: demo-app
  namespace: argocd
spec:
  project: default
  source:
    repoURL: git@github.com:AlokRajak123/gitops-homelab.git
    targetRevision: main
    path: apps/demo-app
  destination:
    server: https://kubernetes.default.svc
    namespace: demo
  syncPolicy:
    automated:
      prune: true
      selfHeal: true
EOF


⚠️ Replace <YOUR_GITHUB_USERNAME> before commit.

🔹 STEP 8: Push to GitHub
git add .
git commit -m "Initial GitOps demo app"
git branch -M main
git remote add origin https://github.com/<YOUR_GITHUB_USERNAME>/gitops-homelab.git
git push -u origin main

🔹 STEP 9: Deploy via Argo CD
kubectl create namespace demo
kubectl apply -f argocd/applications/demo-app.yaml


🎯 At this moment GitOps becomes active

Verify:

kubectl get applications -n argocd
kubectl get pods -n demo

🔹 STEP 10: Access the App
kubectl port-forward svc/demo-app -n demo 8081:80


Open:

http://localhost:8081


You should see NGINX Welcome Page 🎉

🧠 What You Have Achieved Already

✔ Real GitOps pipeline
✔ Argo CD auto-sync
✔ Kubernetes declarative deployment
✔ Interview-grade setup

