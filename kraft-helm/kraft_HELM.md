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