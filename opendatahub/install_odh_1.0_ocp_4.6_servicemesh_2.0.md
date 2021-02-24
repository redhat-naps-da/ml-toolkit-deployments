# Install ODH v1.0 on OCP 4.6 running Service Mesh 2.0

## Deploy ODH Operator with KfDef variations
You have [3 provided ODH kfdef files](https://github.com/opendatahub-io/odh-manifests/tree/master/kfdef) that provide deployment variations for OpenShift compared to a [Kubeflow KfDef](https://github.com/kubeflow/manifests/blob/v1.2-branch/kfdef/kfctl_istio_dex.v1.2.0.yaml):

|Function|kfdef|kfctl_openshift|kfctl_openshift_distributed_training|kfctl_openshift_kubeflow_monitoring|a kubeflow kfdef|
|-|-|-|-|-|-|
|||||||
|Home|ODH Dashboard|x|x|||
|Home|KF Central Dashboard|||x|x|
|Monitoring|Grafana|x|x|x|x|
|Monitoring|Prometheus|x|x|x|x|
|Data Exploration|Apache Superset|x|x|||
|Pre-trained Models|AI Library|x|x|||
|Data Streaming|Apache Kafka|x|x|||
|Data Analytics|Apache Spark|x|x||x|
|Model Development|JupyterHub|x|x|||
|Model Development|Jupyter Web App|||x|x|
|Model Training|TFJob Tensorflow||x|x|x|
|Model Visualization|Tensorboard|||x|x|
|Model Training|PyTorch Job||x|x|x|
|Model Tuning|Katib|||x|x|
|Model Serving|Seldon|x|x|x|x|
|Model Serving|KF Serving||||x|
|Workflows & Pipelines|Argo|x|x|x||
|Workflows|Apache Airflow|x|||
|Pipelines|KF Pipelines|||x|x|
|Pipeline Artifacts|Minio|||x|x|
|Pipeline Artifacts|MySQL|||x|x|
|Pod Security|OCP Security Context Constraints|||x||
|Networking|Istio Service Mesh|||x|x|
|Authn & Authz|Dex OIDC||||x|
|Isolation|KF Roles||||x|
|Networking|Webhook||||x|
|Security|Certificate Manager|||x|x|
|Model Tracking|Metadata|||x|x|
|Scheduling|Scheduledworkflow|||x|x|
|Isolation|Profiles|||x|x|

# Install Operators
1. From OpenShift Web Interface
1. From OpenShift Operators > OperatorHub
1. Install Elasticsearch Operator (accept defaults)
1. Install Jaeger Operator (accept defaults)
1. Install Kiali Operator (accept defaults)
1. Install Service Mesh Operator (accept defaults)
1. Install Community Open Data Hub Operator (accept defaults)

# Create ODH Project
1. Create a new-project 

> For Kubeflow deployments you must set the project name to kubeflow. For Open Data Hub, you can set it as desired.

# Deploy the Service Mesh

1. From the created project
1. Navigate to Operators → Installed Operators
1. Click the Red Hat OpenShift Service Mesh Operator
1. Click Istio Service Mesh Control Plane
1. Click Create ServiceMeshControlPlane (accept defaults)
1. Wait for Status Conditions: Installed, Reconciled, Ready
1. Click Istio Service Mesh Control Plane
1. Click Create ServiceMeshControlPlane (accept defaults)
1. Wait Status Conditions: Installed, Reconciled, Ready
1. Click Istio Service Mesh Member Roll
1. Click Create ServiceMeshMemberRoll
1. Select YAML View
1. Update spec members to inlude your ODH Project created previously

# Deploy kfctl_openshift

1. Select your desired project (or create a new one)
1. Navigate to Operators → Installed Operators 
1. Select Open Data Hub Operator
1. Select Create Instance
1. Create

# Deploy kfctl_openshift_distributed_training

1. Select your desired project (or create a new one)
1. Navigate to Operators → Installed Operators 
1. Select Open Data Hub Operator
1. Select Create Instance
1. Select YAML View to review
1. Select all text and delete
1. Copy and paste all text from [kfctl_openshift_distributed_training.yaml](https://github.com/opendatahub-io/odh-manifests/blob/master/kfdef/kfctl_openshift_distributed_training.yaml)
1. Update metadata namespace to match your project name
1. Create

# Deploy kfctl_openshift_kubeflow_monitoring

1. Select your desired project (or create a new one call kubeflow)
1. Navigate to Operators → Installed Operators 
1. Select Open Data Hub Operator
1. Select Create Instance
1. Select YAML View to review
1. Select all text and delete
1. Copy and paste all text from [kfctl_openshift_distributed_training.yaml](https://github.com/opendatahub-io/odh-manifests/blob/master/kfdef/kfctl_openshift_distributed_training.yaml)
1. Verify metadata namespace to kubeflow
1. Create
1. For each namespace created via the Kubeflow registration process, Service Mesh Member Rolls need to be manually updated with the new member in order to opt-in.

> Still need to verify the sidecar injection parameter if USE_ISTIO: true 