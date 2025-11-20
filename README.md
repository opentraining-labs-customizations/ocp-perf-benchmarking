# OpenShift Performance Benchmarking

Ansible automation for deploying Elasticsearch on OpenShift for performance testing.

## Setup

1. Activate the Python virtual environment:
   ```bash
   source bin/activate
   ```

2. Install dependencies:
   ```bash
   pip install -r requirements.txt
   ```

## Usage

Deploy Elasticsearch cluster:
```bash
ansible-playbook admin-configure-environment.yaml
```

This will:
- Install the Elasticsearch ECK operator
- Deploy a single-node Elasticsearch 8.15.0 cluster
- Configure 2Gi memory allocation with emptyDir storage

## Testing Connectivity

From within the cluster:
```bash
# Get password
PASSWORD=$(oc get secret elasticsearch-es-elastic-user -n openshift-logging -o jsonpath='{.data.elastic}' | base64 -d)

# Check cluster health
oc exec -n openshift-logging elasticsearch-es-default-0 -- curl -k -u elastic:$PASSWORD https://elasticsearch-es-http.openshift-logging.svc.cluster.local:9200/_cluster/health?pretty
```

## Endpoint

- **URL**: `https://elasticsearch-es-http.openshift-logging.svc.cluster.local:9200`
- **User**: `elastic`
- **Password**: Retrieved from `elasticsearch-es-elastic-user` secret