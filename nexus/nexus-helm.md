### Nexus Repository Manager Deployment with NodePort (Kubernetes + Helm)

### 1.  Add Sonatype Helm Repo

```bash
helm repo add sonatype https://sonatype.github.io/helm3-charts/
helm repo update
```

### 2.  Create a Namespace

```
`kubectl create namespace nexus`
```

### 3.  Install Nexus Repository Manager

Install Nexus with Helm and expose using a ClusterIP first:
```bash
helm install my-nexus-repository-manager sonatype/nexus-repository-manager -n nexus

```

### 4.  Change Service Type to NodePort

Patch the service to make it accessible via NodePort:

```
`kubectl patch svc my-nexus-repository-manager -n nexus -p '{"spec": {"type": "NodePort"}}'`
```

### 5. Get the Admin Password

To get the admin password:
```
kubectl exec -n nexus -it <nexus-pod-name> -- cat /nexus-data/admin.password
```
You can find the pod name with:

```
`kubectl get pods -n nexus`
```

6.  Access the Web Interface
Check the assigned NodePort:
```
kubectl get svc -n nexus
```
Example output:
```
NAME                          TYPE       CLUSTER-IP      EXTERNAL-IP   PORT(S)          AGE
my-nexus-repository-manager   NodePort   10.104.217.66   <none>        8081:32462/TCP   5m
```
Then open your browser and access:
```
http://<NODE-IP>:32462
```