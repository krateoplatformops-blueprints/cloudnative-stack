# cloudnative-stack
A Cloud Native Stack based on Frontend / Backend / Kafka / Hazelcast / MongoDB / CloudNativePG

## Overview

This Krateo blueprint deploys a complex, multi-tier application stack on Kubernetes. It is designed to be highly configurable and relies on best-practice Kubernetes operators for managing stateful components like databases and message queues.

The blueprint will provision the following components:

-   A **Frontend** pod (e.g., Nginx) to serve a user interface.
-   A **Backend** pod (e.g., Spring Boot) for business logic.
-   A **Kafka Topic** managed by the Strimzi operator.
-   A **Hazelcast** in-memory data grid cluster managed by the Hazelcast Platform Operator.
-   A **MongoDB** replica set managed by the MongoDB Community Kubernetes Operator.
-   A **PostgreSQL** cluster managed by the CloudNative-PG operator.

## Prerequisites

For this blueprint to function correctly, your Kubernetes cluster **must** have the following operators installed and running. Please follow their official documentation for installation instructions.

1.  **Strimzi (for Kafka):**
    -   **Purpose:** Manages Kafka clusters and topics.
    -   **Installation Guide:** [Strimzi Deployment Documentation](https://strimzi.io/docs/operators/latest/deploying-and-upgrading.html)

2.  **Hazelcast Platform Operator:**
    -   **Purpose:** Manages Hazelcast in-memory computing clusters.
    -   **Installation Guide:** [Hazelcast Operator Deployment](https://docs.hazelcast.com/hazelcast/latest/deploy/kubernetes-operator)

3.  **MongoDB Community Kubernetes Operator:**
    -   **Purpose:** Manages MongoDB Community replica sets and clusters.
    -   **Installation Guide:** [https://github.com/mongodb/mongodb-kubernetes](https://github.com/mongodb/mongodb-kubernetes)

4.  **CloudNative-PG (for PostgreSQL):**
    -   **Purpose:** Manages the full lifecycle of a PostgreSQL cluster.
    -   **Installation Guide:** [CloudNative-PG Installation](https://cloudnative-pg.io/documentation/current/installation/)

## Installation and Usage

### 1. Publish the Helm Chart

This blueprint is based on a Helm chart. You must first publish the `blueprint/` directory to an OCI-compliant registry (like GHCR, Docker Hub, etc.).

### 2. Update and Apply the CompositionDefinition

Update the `compositiondefinition.yaml` file with the correct URL from the previous step.

```yaml
apiVersion: core.krateo.io/v1alpha1
kind: CompositionDefinition
metadata:
  name: multi-tier-app
  namespace: krateo-system # Or your desired namespace
spec:
  chart:
    # IMPORTANT: Update this URL to point to your published chart
    url: oci://ghcr.io/your-username/your-repo/multi-tier-app
    version: "0.1.0"
```

Apply the manifest to your cluster: kubectl apply -f compositiondefinition.yaml

### 3. Create a Composition Instance

Once the `CompositionDefinition` is applied, Krateo creates a new Custom Resource Definition (CRD) named `MultiTierApp`. You can now create instances of your application stack by creating a `MultiTierApp` resource.

Below is an example `composition.yaml`:

```yaml
apiVersion: composition.krateo.io/v0-1-0
kind: MultiTierApp
metadata:
  name: my-production-app
  namespace: krateo-system
spec:
  # Override default values here
  frontend:
    replicas: 3
    image: "my-custom-frontend:1.2.0"
  backend:
    replicas: 2
  mongodb_operator:
    storageSize: "10Gi"
    # Provide a specific password instead of a random one
    password: "MyStrongPassword123!"
  cloudnativepg:
    instances: 3
    storageSize: "20Gi"
```

# Requirements
```yaml
# Source: cloudnative-stack/templates/kafka-cluster.yaml
apiVersion: kafka.strimzi.io/v1beta2
kind: Kafka
metadata:
  name: kafka
  namespace: cloudnative-test
spec:
  kafka:
    version: 4.1.0
    metadataVersion: 4.1-IV1
    listeners:
      - name: plain
        port: 9092
        type: internal
        tls: false
      - name: tls
        port: 9093
        type: internal
        tls: true
    config:
      offsets.topic.replication.factor: 1
      transaction.state.log.replication.factor: 1
      transaction.state.log.min.isr: 1
      default.replication.factor: 1
      min.insync.replicas: 1
  entityOperator:
    topicOperator: {}
    userOperator: {}
---
# Source: cloudnative-stack/templates/kafka-cluster.yaml
apiVersion: kafka.strimzi.io/v1beta2
kind: KafkaBridge
metadata:
  name: kafka
  namespace: cloudnative-test
spec:
  replicas: 1
  bootstrapServers: kafka-kafka-bootstrap:9092
  http:
    port: 8080
---
# Source: cloudnative-stack/templates/kafka-cluster.yaml
apiVersion: kafka.strimzi.io/v1beta2
kind: KafkaNodePool
metadata:
  name: dual-role
  namespace: cloudnative-test
  labels:
    strimzi.io/cluster: kafka
spec:
  replicas: 1
  roles:
    - controller
    - broker
  storage:
    type: jbod
    volumes:
      - id: 0
        type: persistent-claim
        size: 10Gi
        deleteClaim: false
        kraftMetadata: shared
---
# Source: cloudnative-stack/templates/kafka-cluster.yaml
apiVersion: v1
kind: Service
metadata:
  name: kafka-lb
  namespace: cloudnative-test
  annotations:
    service.beta.kubernetes.io/aws-load-balancer-type: "nlb"
    service.beta.kubernetes.io/aws-load-balancer-scheme: "internet-facing"
spec:
  type: LoadBalancer
  ports:
    - name: http
      port: 8080
      targetPort: 8080
  selector:
    strimzi.io/name: kafka-bridge
```