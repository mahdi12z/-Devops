#### Step 1: Deploy a Sample Application

First, let’s deploy a sample application to demonstrate the setup.

1. **Deploy a Sample Deployment**:

 ```bash
 kubectl create deployment hello-world --image=gcr.io/google-samples/hello-app:1.0
```
    2.**Expose the Deployment as a Service**:
    
  ```bash
   kubectl expose deployment hello-world --type=NodePort --port=8080
```
#### Step 2: Install MetalLB
 1. Install MetalLB: Apply the MetalLB manifest to install it in your cluster
 ```bash
 kubectl apply -f https://raw.githubusercontent.com/metallb/metallb/main/manifests/metallb.yaml
```


 2. reate a MetalLB Config: The installation manifest does **not** include a configuration file. MetalLB’s components although will start, they will remain idle until we provide the required configuration as an `IpAddressPool`, a new `Kind` introduced in this version and replaced the old way of provisioning address pool configuration with `ConfigMap`.
    Let’s name it `ipaddresspool.yaml`:
    ```yaml
apiVersion: metallb.io/v1beta1  
kind: IPAddressPool  
metadata:  
name: default-pool  
namespace: metallb-system  
spec:  
addresses:  
- 192.168.1.240-192.168.1.250
```
 Let’s name it `l2advertisement.yaml`:
 ```yaml
apiVersion: metallb.io/v1beta1
kind: L2Advertisement
metadata:
  name: default
  namespace: metallb-system
spec:
  ipAddressPools:
  - default-pool
```

```
kubectl apply -f ipaddresspool.yaml
kubectl apply -f l2advertisement.yaml
```

#### 3: Create a LoadBalancer Service

Next, create a LoadBalancer service that will use MetalLB to provide an external IP.
1. **Define the LoadBalancer Service**:
```yml
apiVersion: v1
kind: Service
metadata:
name: hello-world-lb
namespace: default
spec:
type: LoadBalancer
ports:
- port: 80
targetPort: 8080
selector:
app: hello-world
```

2. **Apply the LoadBalancer Service**:
    
    ```
    kubectl apply -f hello-world-lb.yaml
   ```

    
3. **Verify the LoadBalancer Service**:
    
```
kubectl get services
```
   
   Wait until an external IP is assigned to the LoadBalancer service by MetalLB.
4.  Deploy an Ingress Controller
Deploy an Ingress controller to manage external access to your services.1
    1.  Deploy NGINX Ingress Controller :

```
kubectl apply -f https://raw.githubusercontent.com/kubernetes/ingress-nginx/main/deploy/static/provider/cloud/deploy.yaml
```
 2. **Verify the Ingress Controller**:
 ```
 kubectl get pods -n ingress-nginx
```
Ensure that the Ingress controller pods are running.
#### 5: Create an Ingress Resource
Define an Ingress resource to route traffic from the LoadBalancer to your service.
  1. **Define the Ingress Resource**
```
apiVersion: networking.k8s.io/v1
kind: Ingress
metadata:
  name: hello-world-ingress
  namespace: default
  annotations:
    nginx.ingress.kubernetes.io/rewrite-target: /
spec:
  rules:
  - host: aressai.org
    http:
      paths:
      - path: /
        pathType: Prefix
        backend:
          service:
            name: hello-world
            port:
              number: 8080
```

2. **Apply the Ingress Resource**:
```
kubectl apply -f hello-world-ingress.yaml
```

 6. Update DNS Settings
Update your DNS settings to point your domain to the external IP assigned by MetalLB.
7. **Get the External IP**:
```
kubectl get service hello-world-lb
```

1. **Update DNS**: Configure your DNS provider to point your domain  to the external IP assigned by MetalLB.

### Verification and Testing

1. **Access the Application**: Open a browser and navigate to **http://<your-domain. You should see the hello-world application.
2. **Verify Ingress Rules**: Check the defined Ingress rules to ensure traffic is being routed correctly.