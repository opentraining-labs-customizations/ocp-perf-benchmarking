# OPEN-54 - OpenShift Performance Benchmarking lab

Ansible role for deploying Elasticsearch and Grafana on OpenShift for the [OpenShift Performance Benchmarking](https://redhatquickcourses.github.io/ocp-perf-benchmarking/ocp-perf-benchmarking/1/index.html) training.

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

### Deploy Elasticsearch and Grafana
Deploy Elasticsearch cluster and Grafana instance using the role:
```bash
ansible-playbook deploy.yml $(test -f inventory && echo "-i inventory")
```

This will:
- Install the Elasticsearch ECK operator and Grafana operator
- Deploy a single-node Elasticsearch 8.15.0 cluster
- Deploy a Grafana instance for monitoring and visualization
- **Automatically configure Elasticsearch as a Grafana datasource**
- Configure appropriate resource allocation with emptyDir storage for Elasticsearch
- Create OpenShift routes for external access to both services

### Cleanup
Remove all deployed components (Elasticsearch and Grafana):
```bash
ansible-playbook cleanup.yml $(test -f inventory && echo "-i inventory")
```

This will remove:
- OpenShift routes for both services
- Elasticsearch instance and Grafana instance
- ECK operator and Grafana operator subscriptions
- Created namespaces (if safe to remove)

## Role Configuration

The role can be customized by overriding variables in `defaults/main.yml`:

**Elasticsearch Configuration:**
- `elasticsearch_operator_namespace`: Namespace for the ECK operator (default: `openshift-operators`)
- `elasticsearch_namespace`: Namespace for Elasticsearch instance (default: `openshift-logging`)
- `elasticsearch_instance_name`: Name of the Elasticsearch cluster (default: `elasticsearch`)
- `elasticsearch_version`: Elasticsearch version to deploy (default: `8.15.0`)
- `elasticsearch_node_count`: Number of Elasticsearch nodes (default: `1`)
- `elasticsearch_memory_request/limit`: Memory allocation (default: `2Gi`)
- `elasticsearch_cpu_request/limit`: CPU allocation (default: `1`/`2`)

**Grafana Configuration:**
- `grafana_operator_namespace`: Namespace for the Grafana operator (default: `openshift-operators`)
- `grafana_namespace`: Namespace for Grafana instance (default: `openshift-logging`)
- `grafana_instance_name`: Name of the Grafana instance (default: `grafana`)
- `grafana_admin_user`: Grafana admin username (default: `admin`)
- `grafana_admin_password`: Grafana admin password (default: `admin123`)
- `grafana_memory_request/limit`: Memory allocation (default: `512Mi`/`1Gi`)
- `grafana_cpu_request/limit`: CPU allocation (default: `200m`/`500m`)

## Testing Connectivity

**Elasticsearch:**
```bash
# Check ElasticSearch health
oc get elasticsearch -n openshift-logging

# Get Elasticsearch password
PASSWORD=$(oc get secret elasticsearch-es-elastic-user -n openshift-logging -o jsonpath='{.data.elastic}' | base64 -d)

# Get Elasticsearch route URL
ES_INSTANCE=$(oc get route elasticsearch-route -n openshift-logging -o jsonpath='https://{.spec.host}')

# Check endpoint
curl -k -u elastic:$PASSWORD $ES_INSTANCE/_cluster/health?pretty
```

**Grafana:**
```bash
# Get Grafana route URL
GRAFANA_URL=$(oc get route grafana-route -n openshift-logging -o jsonpath='https://{.spec.host}')
echo "Grafana URL: $GRAFANA_URL"
echo "Login with: admin/admin123"
echo "Elasticsearch datasource is automatically configured and ready to use!"
```

## Endpoints

**Elasticsearch:**
- **URL**: Retrieved from `elasticsearch-route`
- **User**: `elastic`
- **Password**: Retrieved from `elasticsearch-es-elastic-user` secret

**Grafana:**
- **URL**: Retrieved from `grafana-route`
- **User**: `admin` (configurable)
- **Password**: `admin123` (configurable)
- **Datasources**: Elasticsearch automatically configured as default datasource
