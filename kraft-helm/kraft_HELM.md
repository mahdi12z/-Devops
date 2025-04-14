## **Deploying Strimzi using Installation Files**

This guide outlines how to deploy the **Strimzi Kafka Operator** and create an **Apache Kafka** cluster using the provided Strimzi installation files.

### **Step 1: Create a Namespace**

First, create a dedicated namespace for your Kafka resources:

```bash
kubectl create namespace kafka
```

### **Step 2: Deploy the Strimzi Operator**

Install the Strimzi Cluster Operator and related Kubernetes resources:
```bash
kubectl create -f 'https://strimzi.io/install/latest?namespace=kafka' -n kafka
```
> The Strimzi installation files include `ClusterRoles`, `ClusterRoleBindings`, and `CustomResourceDefinitions (CRDs)`. The `namespace=kafka` query parameter automatically adjusts the default namespace (which is usually `myproject`) to `kafka`.

**Important:** If namespace references in the install files do not match, the Cluster Operator will not have sufficient permissions to operate correctly.

### **Step 3: Verify Operator Deployment**

Monitor the deployment progress of the Strimzi Cluster Operator:
```
kubectl get pod -n kafka --watch
```
You can also view the operator’s logs:
```bash
kubectl logs deployment/strimzi-cluster-operator -n kafka -f
```

Once the operator is running, it will watch for Strimzi custom resources and act on them automatically.
### **Step 4: Deploy a Kafka Cluster (KRaft Mode)**

To deploy a single-node Apache Kafka cluster in KRaft mode, apply the following custom resource:
```bash
kubectl apply -f https://strimzi.io/examples/latest/kafka/kraft/kafka-single-node.yaml -n kafka
```

#### NOTE: edit volume
```bash
kubectl apply -f https://github.com/strimzi/strimzi-kafka-operator/blob/main/examples/kafka/kraft/kafka.yaml -n kafka
```



Wait until the Kafka cluster is fully ready:
```bash
kubectl wait kafka/my-cluster --for=condition=Ready --timeout=300s -n kafka
```
This step may take longer depending on your network speed. You can rerun the command if it times out.


### **Step 5: Produce and Consume Messages**

#### **Send Messages (Kafka Producer)**

Run a Kafka producer that sends messages to a topic (automatically created if it doesn't exist):

```
kubectl -n kafka run kafka-producer -ti \
  --image=quay.io/strimzi/kafka:0.45.0-kafka-3.9.0 \
  --rm=true --restart=Never \
  -- bin/kafka-console-producer.sh \
  --bootstrap-server my-cluster-kafka-bootstrap:9092 \
  --topic my-topic
```

```
hi Aressai
```


#### **Receive Messages (Kafka Consumer)**

Open a separate terminal and run a Kafka consumer:
```
kubectl -n kafka run kafka-consumer -ti \
  --image=quay.io/strimzi/kafka:0.45.0-kafka-3.9.0 \
  --rm=true --restart=Never \
  -- bin/kafka-console-consumer.sh \
  --bootstrap-server my-cluster-kafka-bootstrap:9092 \
  --topic my-topic \
  --from-beginning
```

You should see the message you previously sent:


## View More Examples from Strimzi

```bash

Kafka users (TLS/SCRAM)

Kafka topics

Kafka Connect

Kafka Bridge

Monitoring setup

Secure deployments
```

```
https://github.com/strimzi/strimzi-kafka-operator/tree/main/examples
#main
https://github.com/strimzi/strimzi-kafka-operator/tree/main/examples/kafka/kraft

```

-----
## KAFKA UI
## Kafka UI Deployment Guide on Kubernetes (with Strimzi & Helm)
### Step 1: Add Kafka UI Helm Repository
```bash
helm repo add kafka-ui https://provectus.github.io/kafka-ui-charts
helm repo update
```
 Step 2: Create the kafka-ui-values.yaml file
```bash
kafka:
  clusters:
    - name: kraft
      bootstrapServers: my-cluster-kafka-bootstrap:9092

env:
  - name: DYNAMIC_CONFIG_ENABLED
    value: "true"

service:
  type: NodePort
  nodePort: 30080
```
Replace my-cluster-kafka-bootstrap:9092 with the correct bootstrap service from your Strimzi Kafka deployment (usually in the format <cluster-name>-kafka-bootstrap:9092)
### Step 3: Install Kafka UI with Helm
```bash
helm install kafka-ui kafka-ui/kafka-ui \
  --namespace kafka \
  -f kafka-ui-values.yaml
```
To update an existing deployment:
```bash
helm upgrade kafka-ui kafka-ui/kafka-ui \
  --namespace kafka \
  -f kafka-ui-values.yaml
```
### Step 4: Access Kafka UI
#### 1.Get the Node IP:
```bash
kubectl get nodes -o wide
```
#### 2.Open in your browser:
```bash
http://<NODE-IP>:30080
```
