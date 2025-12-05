## CloudNativePG
Install with Helm chart, no additional components required. Installs cluster-wide SQL server, with replicas, redundancy, etc.

```sh
helm repo add cnpg https://cloudnative-pg.github.io/charts
helm repo update
helm upgrade --install cnpg \
  --namespace cnpg-system \
  --create-namespace \
  cnpg/cloudnative-pg
```

## MongoDB
Use the MongoDB Controllers for Kubernetes Operator for:
- Ops Manager resources
- MongoDB standalone, replica set, and sharded cluster resources

Ops Manager is required to create MongoDB instances.

Install the operator:
```sh
helm repo add mongodb https://mongodb.github.io/helm-charts
kubectl apply -f https://raw.githubusercontent.com/mongodb/mongodb-kubernetes/1.5.0/public/crds.yaml
helm upgrade --install mongodb-kubernetes-operator mongodb/mongodb-kubernetes \
--namespace mongodb \
--set operator.watchNamespace="*" \
--create-namespace
```

# Hazelcast Platform Operator
Install the Operator:
```sh
helm repo add hazelcast https://hazelcast-charts.s3.amazonaws.com/
helm repo update
helm install operator hazelcast/hazelcast-platform-operator --version=5.16.0 \
    --set=installCRDs=true \
    --create-namespace \
    -n hazelcast-system
```

Create secret for license key:
```yaml
kubectl create secret generic hazelcast-license-key -n hazelcast-system --from-literal=license-key=<license key>
```

# Strimzi for Kafka
Install the operator with:
```sh
helm upgrade --install strimzi-cluster-operator oci://quay.io/strimzi-helm/strimzi-kafka-operator \
  -n kafka \
  --set 'watchAnyNamespace=true' \
  --create-namespace
```
