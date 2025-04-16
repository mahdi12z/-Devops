### Nexus Repository Manager Deployment with NodePort (Kubernetes + Helm)

### 1.  Add Sonatype Helm Repo

```bash
helm repo add stevehipwell https://stevehipwell.github.io/helm-charts/
helm repo update
```

### 2.  Create a Namespace

```
kubectl create namespace nexus
```

### 3.  Install Nexus Repository Manager

Install Nexus with Helm and expose using a ClusterIP first:
```bash
helm install my-nexus3 stevehipwell/nexus3 -n nexus

```

### 4.  Change Service Type to NodePort

Patch the service to make it accessible via NodePort:

```
kubectl get svc -n nexus
kubectl patch svc my-nexus3  -n nexus -p '{"spec": {"type": "NodePort"}}'
```

### 5. Get the Admin Password

To get the admin password:
```
kubectl exec -n nexus -it <nexus-pod-name> -- cat /nexus-data/admin.password
```
You can find the pod name with:

```
kubectl get pods -n nexus
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

## kubectl patch svc

This command updates the my-nexus3 service (in the nexus namespace) to expose it as a NodePort service.

```bash

kubectl patch svc my-nexus3 -n nexus --type='merge' -p '
{
  "spec": {
    "type": "NodePort",
    "ports": [
      {
        "name": "docker-registry",
        "port": 5000,
        "targetPort": 5000,
        "protocol": "TCP",
        "nodePort": 32500
      }
    ]
  }
}'

```

type: NodePort: Makes the service accessible outside the cluster.

port: 5000: The port exposed by the service inside the cluster.

targetPort: 5000: The port on the actual pod (Nexus3 container).

nodePort: 32500: The external port on the node (host machine). You can access the service via http://<node-ip>:32500.



----
 ## 32500 → 5000

This means HTTP requests sent to NODE_IP:32500 will be forwarded to port 5000 inside the pod running Nexus3.


```
32500
http>>5000
```
----

## Docker insecure registry config

```bash
{
  "insecure-registries": [
    "192.168.100.*:32500"
  ]
}
```
```bash
sudo systemctl restart docker
docker login 192.168.100.*:32500
``



