## installation Guide: Argo CD + Argo Rollouts with UI Dashboard
## Part 1: Argo CD Installation

###  1. Create namespace:
```
kubectl create namespace argocd
```
## 2. Install Argo CD from official manifests:

```
kubectl apply -n argocd -f https://raw.githubusercontent.com/argoproj/argo-cd/stable/manifests/install.yaml

```
### 3. Access the Argo CD UI:

Use port-forwarding for quick local access:
```
kubectl patch svc argocd-server -n argocd -p '{"spec": {"type": "NodePort"}}'
```
Access in browser:
```
https://<NODE_IP>:<NodePort>
```

## 4. Get initial admin password:
```
kubectl get secret argocd-initial-admin-secret -n argocd \
  -o jsonpath="{.data.password}" | base64 -d && echo
```

## 5. Install Argo CD CLI:
```
curl -sSL -o argocd https://github.com/argoproj/argo-cd/releases/latest/download/argocd-linux-amd64
chmod +x argocd
sudo mv argocd /usr/local/bin
```

## 6.Install Argo Rollouts via Helm (with dashboard enabled):
```
helm repo add argo https://argoproj.github.io/argo-helm
helm repo update

helm install my-rollouts argo/argo-rollouts \
  --namespace argo-rollouts --create-namespace \
  --set dashboard.enabled=true \
  --set dashboard.service.type=NodePort \
  --set dashboard.service.nodePort=30100
```

## 7.Access the Rollouts Dashboard:
```
kubectl get nodes -o wide
```

```
http://<NODE_IP>:30100
```