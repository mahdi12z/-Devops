### **HAProxy Configuration for Routing Traffic to Multiple Kubernetes Services Based on Different Ports**

This configuration assumes that you have multiple services in your Kubernetes cluster, each exposed via different `NodePort`s. The objective is to route traffic to each service based on the port it is exposed on.

### **1. HAProxy Installation**

If HAProxy is not installed yet, use the following command to install it on your external server:
```bash
sudo apt update
sudo apt install haproxy -y
```


### **2. HAProxy Configuration**

After installing HAProxy, you need to configure it to listen on different ports and route the traffic to the appropriate Kubernetes services.

#### **2.1 Edit the HAProxy Configuration File**

The HAProxy configuration file is typically located at `/etc/haproxy/haproxy.cfg`. Edit it with:

```bash
sudo vi /etc/haproxy/haproxy.cfg
```


#### **2.2 HAProxy Configuration File**

Below is an example configuration file that routes traffic to different Kubernetes services based on the incoming port:

```bash
# Frontend for nginx service on port 80
frontend http_nginx
    bind *:80  # The port that HAProxy listens on for nginx
    default_backend nginx_back

# Frontend for grafana service on port 81
frontend http_grafana
    bind *:81  # The port that HAProxy listens on for grafana
    default_backend grafana_back

# Frontend for webapp service on port 82
frontend http_webapp
    bind *:82  # The port that HAProxy listens on for webapp
    default_backend webapp_back

# Backend for nginx service
backend nginx_back
    balance roundrobin
    option httpchk GET /healthz
    server node1 <NODE1_IP>:30080 check
    server node2 <NODE2_IP>:30080 check
    server node3 <NODE3_IP>:30080 check

# Backend for grafana service
backend grafana_back
    balance roundrobin
    option httpchk GET /api/health
    server node1 <NODE1_IP>:30081 check
    server node2 <NODE2_IP>:30081 check
    server node3 <NODE3_IP>:30081 check

# Backend for webapp service
backend webapp_back
    balance roundrobin
    option httpchk GET /healthz
    server node1 <NODE1_IP>:30082 check
    server node2 <NODE2_IP>:30082 check
    server node3 <NODE3_IP>:30082 check

```

### **3. Explanation of the Configuration:**

1. **Frontend for Each Port:**
    
    - `frontend http_nginx`: Listens on port `80` and routes traffic to the `nginx` service.
        
    - `frontend http_grafana`: Listens on port `81` and routes traffic to the `grafana` service.
        
    - `frontend http_webapp`: Listens on port `82` and routes traffic to the `webapp` service.
        
2. **Backend for Each Service:**
    
    - Each `backend` routes traffic to the corresponding `NodePort` of the Kubernetes service (e.g., `nginx` is on port `30080`, `grafana` is on port `30081`, and `webapp` is on port `30082`).
        
    - The `balance roundrobin` directive ensures that traffic is distributed evenly among the available nodes.
        
3. **Health Check:**
    
    - `option httpchk GET /healthz`: This performs an HTTP health check on the specified path to ensure the backend servers are healthy.
        
4. **Adding More Services:**
    
    - You can easily add more services by duplicating the `frontend` and `backend` sections, adjusting the port numbers accordingly.
        

### **4. Restart HAProxy**

After updating the configuration, restart HAProxy to apply the changes:

```bash
sudo systemctl restart haproxy
```

### **5. Open Ports in the Firewall**

Make sure that the necessary ports (e.g., `80`, `81`, and `82`) are open in your firewall to allow traffic to reach HAProxy:
```bash
sudo ufw allow 80/tcp
sudo ufw allow 81/tcp
sudo ufw allow 82/tcp

```

### **Conclusion**

This configuration ensures that incoming traffic on different ports (e.g., `80`, `81`, and `82`) is properly routed to different Kubernetes services (e.g., `nginx`, `grafana`, `webapp`). This setup is simple and effective for managing multiple services exposed via `NodePort` in a Kubernetes cluster.

If you need further assistance or adjustments to the configuration, feel free to ask!