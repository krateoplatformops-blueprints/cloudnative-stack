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
Install the example database with:
```yaml
apiVersion: postgresql.cnpg.io/v1
kind: Cluster
metadata:
  name: cluster-example
  namespace: cnpg-system
spec:
  instances: 3
  storage:
    size: 1Gi
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
kubectl create secret generic hazelcast-license-key -n hazelcast-system --from-literal=license-key=TrialLicense#10Nodes#eyJhbGxvd2VkTmF0aXZlTWVtb3J5U2l6ZSI6MTAwLCJhbGxvd2VkTnVtYmVyT2ZOb2RlcyI6MTAsImFsbG93ZWRUaWVyZWRTdG9yZVNpemUiOjAsImFsbG93ZWRUcGNDb3JlcyI6MCwiY3JlYXRpb25EYXRlIjoxNzYxODQ3ODcwLjYzNjUyOTU1OSwiZXhwaXJ5RGF0ZSI6MTc2NDM3NDM5OS45OTk5OTk5OTksImZlYXR1cmVzIjpbMCwyLDMsNCw1LDYsNyw4LDEwLDExLDEzLDE0LDE1LDE3LDIxLDIyXSwiZ3JhY2VQZXJpb2QiOjAsImhhemVsY2FzdFZlcnNpb24iOjk5LCJvZW0iOmZhbHNlLCJ0cmlhbCI6dHJ1ZSwidmVyc2lvbiI6IlY3In0=.7Wh2kBN9XyTR6J4rwj18vvrMvFk37ckhw21VmVfKJ9-sYAEscbMOj20SDn7tzXkBqsurfXYpb3Ke0dMBrA0qDQ==
```

Then create the cluster with:
```yaml
apiVersion: hazelcast.com/v1alpha1
kind: Hazelcast
metadata:
  name: hazelcast-sample
  namespace: hazelcast-system
spec:
  clusterSize: 2
  clusterName: mycluster
  repository: 'docker.io/hazelcast/hazelcast'
  version: '5.6.0'
  licenseKeySecretName: hazelcast-license-key
```

# Strimzi for Kafka
Install the operator with:
```sh
helm upgrade --install strimzi-cluster-operator oci://quay.io/strimzi-helm/strimzi-kafka-operator \
  -n kafka \
  --set 'watchAnyNamespace=true' \
  --create-namespace
```

Create sample server with:
```sh
kubectl apply -f https://strimzi.io/examples/latest/kafka/kafka-single-node.yaml -n kafka 
```
