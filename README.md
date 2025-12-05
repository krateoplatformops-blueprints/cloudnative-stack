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

## Requirements

For this blueprint to function correctly, your Kubernetes cluster **must** have the following operators installed and running. Please follow their official documentation for installation instructions.

1.  **Strimzi (for Kafka):**
    -   **Purpose:** Manages Kafka clusters and topics.

2.  **Hazelcast Platform Operator:**
    -   **Purpose:** Manages Hazelcast in-memory computing clusters.

3.  **MongoDB Kubernetes Operator:**
    -   **Purpose:** Manages MongoDB Community replica sets and clusters.

4.  **CloudNative-PG (for PostgreSQL):**
    -   **Purpose:** Manages the full lifecycle of a PostgreSQL cluster.

Checkout [REQUIREMENTS.md](REQUIREMENTS.md) for installation instructions.

## Usage

### Install the Helm Chart

Download Helm Chart values:

```sh
helm repo add marketplace https://marketplace.krateo.io
helm repo update marketplace
helm inspect values marketplace/cloudnative-stack --version 0.1.0 > ~/cloudnative-stack-values.yaml
```

Modify the *cloudnative-stack-values.yaml* file as the following example:

```yaml
frontend:
  enabled: true
  deployments:
  - name: frontend-one
    image: nginx:latest
    port: 80
    replicas: 1
backend:
  enabled: true
  deployments:
  - name: backend-one
    image: openeuler/spring-boot:latest
    port: 8080
    replicas: 1
    kafka:
      enabled: true
      topics:
      - topicName: "first-topic"
        partitions: 1
        replicas: 1    
    hazelcast:
      enabled: true
      clusters:
      - name: "hz-first"
        size: 1
    mongodb:
      enabled: true
      instances: 
      - clusterName: "mongodb-one"
        replicas: 1
        storageSize: "2Gi"
    cloudnativepg:
      enabled: true
      instances:
      - clusterName: "cnpg-the-first"
        replicas: 1
        storageSize: "1Gi"
        databaseName: "appdb"
```

Install the Blueprint:

```sh
helm install <release-name> cloudnative-stack \
  --repo https://marketplace.krateo.io \
  --namespace <release-namespace> \
  --create-namespace \
  -f ~/cloudnative-stack-values.yaml
  --version 0.1.0 \
  --wait
```

### Install using Krateo Composable Operation

Install the CompositionDefinition for the *Blueprint*:

```sh
cat <<EOF | kubectl apply -f -
apiVersion: core.krateo.io/v1alpha1
kind: CompositionDefinition
metadata:
  name: cloudnative-stack
  namespace: krateo-system
spec:
  chart:
    repo: cloudnative-stack
    url: https://marketplace.krateo.io
    version: 0.1.0
EOF
```

Install the Blueprint using, as metadata.name, the *Composition* name (the Helm Chart name of the composition):

```sh
cat <<EOF | kubectl apply -f -
apiVersion: composition.krateo.io/v0-1-0
kind: CloudNativeStack
metadata:
  name: <release-name> 
  namespace: <release-namespace> 
spec:
  frontend:
    enabled: true
    deployments:
    - name: frontend-one
      image: nginx:latest
      port: 80
      replicas: 1
  backend:
    enabled: true
    deployments:
    - name: backend-one
      image: openeuler/spring-boot:latest
      port: 8080
      replicas: 1
      kafka:
        enabled: true
        topics:
        - topicName: "first-topic"
          partitions: 1
          replicas: 1    
      hazelcast:
        enabled: true
        clusters:
        - name: "hz-first"
          size: 1
      mongodb:
        enabled: true
        instances: 
        - clusterName: "mongodb-one"
          replicas: 1
          storageSize: "2Gi"
      cloudnativepg:
        enabled: true
        instances:
        - clusterName: "cnpg-the-first"
          replicas: 1
          storageSize: "1Gi"
          databaseName: "appdb"

EOF
```

### Install using Krateo Composable Portal

```sh
cat <<EOF | kubectl apply -f -
apiVersion: core.krateo.io/v1alpha1
kind: CompositionDefinition
metadata:
  name: portal-blueprint-page
  namespace: krateo-system
spec:
  chart:
    repo: cloudnative-stack
    url: https://marketplace.krateo.io
    version: 1.1.0
EOF
```

Install the Blueprint using, as metadata.name, the *Blueprint* name (the Helm Chart name of the blueprint):

```sh
cat <<EOF | kubectl apply -f -
apiVersion: composition.krateo.io/v1-1-0
kind: PortalBlueprintPage
metadata:
  name: cloudnative-stack
  namespace: demo-system
spec:
  blueprint:
    url: https://marketplace.krateo.io
    version: 0.1.0 # this is the Blueprint version
    hasPage: true
  form:
    alphabeticalOrder: false
    hasPage: true
  panel:
    title: Cloud Native Stack
    icon:
      name: fa-cloud
EOF
```



























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
  frontend:
    enabled: true
    deployments:
    - name: frontend-one
      image: nginx:latest
      port: 80
      replicas: 1
  backend:
    enabled: true
    deployments:
    - name: backend-one
      image: openeuler/spring-boot:latest
      port: 8080
      replicas: 1
      kafka:
        enabled: true
        clusterName: "kafka"
        topics:
        - topicName: "first-topic"
          partitions: 3
          replicas: 1    
      hazelcast:
        enabled: true
        clusters:
        - name: "hz-first"
          replicas: 1
      mongodb:
        enabled: true
        instances: 
        - clusterName: "mongodb-one"
          replicas: 1
          storageSize: "2Gi"
          username: "myuser"
      cloudnativepg:
        enabled: true
        instances:
        - clusterName: "cnpg-the-first"
          replicas: 1
          storageSize: "1Gi"
          databaseName: "appdb"
          user: "appuser"
```

