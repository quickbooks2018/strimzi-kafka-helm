# Strimzi Kafka Helm Chart

- kind cluster setup https://github.com/quickbooks2018/kind-nginx-ingress.git

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
  brokerConnect: strimzi-kafka-cluster-kafka-bootstrap.strimzi.svc.cluster.local:9092 # Updated to use the bootstrap service
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
helm -n strimzi upgrade --install kafdrop --create-namespace -f values.yaml ./ --wait
```

- kafka topic creation
```bash
kubectl get pods -n strimzi
kubectl -n strimzi exec -it strimzi-kafka-cluster-kafka-0 -- bin/kafka-topics.sh --create --topic my-topic --bootstrap-server strimzi-kafka-cluster-kafka-bootstrap.strimzi.svc.cluster.local:9092 --replication-factor 3 --partitions 3
kubectl -n strimzi exec -it strimzi-kafka-cluster-kafka-0 -- bin/kafka-topics.sh --list --bootstrap-server strimzi-kafka-cluster-kafka-bootstrap.strimzi.svc.cluster.local:9092
```

- kafka start producer
```bash
kubectl -n strimzi exec -it strimzi-kafka-cluster-kafka-0 -- bin/kafka-console-producer.sh --topic my-topic --bootstrap-server strimzi-kafka-cluster-kafka-bootstrap.strimzi.svc.cluster.local:9092
```
- kafka start consumer from begining
```bash
kubectl -n strimzi exec -it strimzi-kafka-cluster-kafka-0 -- bin/kafka-console-consumer.sh --topic my-topic --bootstrap-server strimzi-kafka-cluster-kafka-bootstrap.strimzi.svc.cluster.local:9092 --from-beginning
```

- kafka start consumer
```bash
kubectl -n strimzi exec -it strimzi-kafka-cluster-kafka-0 -- bin/kafka-console-consumer.sh --topic my-topic --bootstrap-server strimzi-kafka-cluster-kafka-bootstrap.strimzi.svc.cluster.local:9092
```

- If you want to achieve the ordering of messages in a partition, you can set the partition key to the same value for all messages. This will ensure that all messages are sent to the same partition.

- Example of a producer with a key

- topic creation
```bash
kubectl -n strimzi exec -it strimzi-kafka-cluster-kafka-0 -- bin/kafka-topics.sh --create --topic fruits --bootstrap-server strimzi-kafka-cluster-kafka-bootstrap.strimzi.svc.cluster.local:9092 --replication-factor 3 --partitions 3
```
- producer with key
```bash
kubectl -n strimzi exec -it strimzi-kafka-cluster-kafka-0 -- bin/kafka-console-producer.sh --topic fruits --bootstrap-server strimzi-kafka-cluster-kafka-bootstrap.strimzi.svc.cluster.local:9092 --property "parse.key=true" --property "key.separator=-"
```

- consumer with key
```bash
kubectl -n strimzi exec -it strimzi-kafka-cluster-kafka-0 -- bin/kafka-console-consumer.sh --topic fruits --bootstrap-server strimzi-kafka-cluster-kafka-bootstrap.strimzi.svc.cluster.local:9092 --property "print.key=true" --property "key.separator=-"
```

- consumer with key from begining
```bash
kubectl -n strimzi exec -it strimzi-kafka-cluster-kafka-0 -- bin/kafka-console-consumer.sh --topic fruits --bootstrap-server strimzi-kafka-cluster-kafka-bootstrap.strimzi.svc.cluster.local:9092 --property "print.key=true" --property "key.separator=-" --from-beginning
```

- key
```bash
hello-apple
hello-banana
hello-orange
bye-grapes
bye-mango
bye-pineapple
```

- delete topic
```bash
kubectl -n strimzi exec -it strimzi-kafka-cluster-kafka-0 -- bin/kafka-topics.sh --delete --topic fruits --bootstrap-server strimzi-kafka-cluster-kafka-bootstrap.strimzi.svc.cluster.local:9092
```

- delete a group
```bash
kubectl -n strimzi exec -it strimzi-kafka-cluster-kafka-0 -- bin/kafka-consumer-groups.sh --delete --group my-group --bootstrap-server strimzi-kafka-cluster-kafka-bootstrap.strimzi.svc.cluster.local:9092
```

- Note: if there is no key specified, the producer will assign a random key to the message. This will result in the messages being distributed across the partitions in a round-robin fashion.

- With key specified, the producer will use the key to determine the partition to which the message will be sent. If the key is null, the message will be sent to a random partition.

