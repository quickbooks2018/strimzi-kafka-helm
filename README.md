# Strimzi Kafka Helm Chart

- Strimzi Helm Chart for deploying a Kafka cluster on Kubernetes

```bash
helm repo ls
helm repo add strimzi https://strimzi.io/charts/
helm repo update 
helm search repo strimzi
helm search repo strimzi/strimzi-kafka-operator
helm show values strimzi/strimzi-kafka-operator
helm search repo strimzi/strimzi-kafka-operator --versions
helm show chart strimzi/strimzi-kafka-operator --version 0.37.0
helm show readme strimzi/strimzi-kafka-operator --version 0.37.0
helm show values strimzi/strimzi-kafka-operator --version 0.37.0
helm show all strimzi/strimzi-kafka-operator --version 0.37.0
helm show values strimzi/strimzi-kafka-operator --version 0.37.0 > strimzi-kafka-operator-values.yaml
```
- Strimzi Kafka Operator
```bash
helm -n strimzi upgrade --install strimzi-kafka-operator --create-namespace strimzi/strimzi-kafka-operator --version 0.37.0
```
### Strimzi Kafka Offical

- https://strimzi.io/docs/operators/0.30.0/quickstart
- https://www.youtube.com/watch?v=wUpO_pfdARw

- Strimzi Kafka Cluster
```bash
cat << EOF | kubectl create -n strimzi -f -
apiVersion: kafka.strimzi.io/v1beta2
kind: Kafka
metadata:
  name: strimzi-kafka-cluster
  namespace: strimzi
spec:
  kafka:
    replicas: 3
    listeners:
      - name: plain
        port: 9092
        type: internal
        tls: false
      - name: tls
        port: 9093
        type: internal
        tls: true
        authentication:
          type: tls
      - name: external
        port: 9094
        type: nodeport
        tls: false
    storage:
      type: jbod
      volumes:
      - id: 0
        type: persistent-claim
        size: 100Gi
        deleteClaim: false
    config:
      offsets.topic.replication.factor: 3
      transaction.state.log.replication.factor: 2
      transaction.state.log.min.isr: 3
      default.replication.factor: 3
      min.insync.replicas: 2
  zookeeper:
    replicas: 3
    storage:
      type: persistent-claim
      size: 20Gi
      deleteClaim: false
  entityOperator:
    topicOperator: {}
    userOperator: {}
EOF
```

- strimzi-kafka-cluster.yaml
```bash
apiVersion: kafka.strimzi.io/v1beta2
kind: Kafka
metadata:
  name: strimzi-kafka-cluster
  namespace: strimzi
spec:
  kafka:
    replicas: 3
    listeners:
      - name: plain
        port: 9092
        type: internal
        tls: false
      - name: tls
        port: 9093
        type: internal
        tls: true
        authentication:
          type: tls
      - name: external
        port: 9094
        type: nodeport
        tls: false
    storage:
      type: jbod
      volumes:
      - id: 0
        type: persistent-claim
        size: 100Gi
        deleteClaim: false
    config:
      offsets.topic.replication.factor: 3
      transaction.state.log.replication.factor: 2
      transaction.state.log.min.isr: 3
      default.replication.factor: 3
      min.insync.replicas: 2
  zookeeper:
    replicas: 3
    storage:
      type: persistent-claim
      size: 20Gi
      deleteClaim: false
  entityOperator:
    topicOperator: {}
    userOperator: {}
```

- Delete Strimzi Kafka Cluster
```bash
cat << EOF | kubectl delete -n strimzi -f -
apiVersion: kafka.strimzi.io/v1beta2
kind: Kafka
metadata:
  name: strimzi-kafka-cluster
  namespace: strimzi
spec:
  kafka:
    replicas: 3
    listeners:
      - name: plain
        port: 9092
        type: internal
        tls: false
      - name: tls
        port: 9093
        type: internal
        tls: true
        authentication:
          type: tls
      - name: external
        port: 9094
        type: nodeport
        tls: false
    storage:
      type: jbod
      volumes:
      - id: 0
        type: persistent-claim
        size: 100Gi
        deleteClaim: false
    version: 3.5.1    
    config:
      offsets.topic.replication.factor: 3
      transaction.state.log.replication.factor: 1
      transaction.state.log.min.isr: 3
      default.replication.factor: 3
      min.insync.replicas: 2
  zookeeper:
    replicas: 3
    storage:
      type: persistent-claim
      size: 20Gi
      deleteClaim: false
  entityOperator:
    topicOperator: {}
    userOperator: {}
EOF
```

- metrics server installation
```bash
helm repo add metrics-server https://kubernetes-sigs.github.io/metrics-server/
helm repo update
helm upgrade --install --set args={--kubelet-insecure-tls} metrics-server metrics-server/metrics-server --namespace kube-system
```

- Terraform eks cluster https://github.com/quickbooks2018/aws-eks-blueprints.git

- kafdop vales.yaml
```bash
replicaCount: 1

image:
  repository: obsidiandynamics/kafdrop
  tag: latest
  pullPolicy: Always

kafka:
  brokerConnect: strimzi-kafka-cluster-kafka-bootstrap:9092 # Updated to use the bootstrap service
  properties: ""
  truststore: ""
  keystore: ""
  propertiesFile: "kafka.properties"
  truststoreFile: "kafka.truststore.jks"
  keystoreFile: "kafka.keystore.jks"

host:

jvm:
  opts: ""
jmx:
  port: 8686

nameOverride: ""
fullnameOverride: ""

cmdArgs: ""

global:
  kubeVersion: ~

server:
  port: 9000
  servlet:
    contextPath: /

service:
  annotations: {}
  type: NodePort
  port: 9000
  nodePort: 30900

ingress:
  enabled: false
  annotations: {}
  apiVersion: ~
  #ingressClassName: ~
  path: /
  #pathType: ~
  hosts: []
  tls: []

resources:
  # limits:
  #  cpu: 100m
  #  memory: 128Mi
  requests:
    cpu: 1m
    memory: 128Mi

nodeSelector: {}

tolerations: []

affinity: {}

podAnnotations: {}

hostAliases: []

mountProtoDesc: 
  enabled: false
  hostPath:
```

- Strimzi Kafka Cluster Monitoring
```bash
git clone https://github.com/obsidiandynamics/kafdrop.git
cd kafdrop/chart
helm -n strimzi upgrade --install kafdrop --create-namespace -f values.yaml ./
```
