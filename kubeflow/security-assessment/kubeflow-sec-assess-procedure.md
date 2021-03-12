# Kubeflow Security Assessment

## Background
As Open Data Hub and Kubeflow gets more attention in the AI/ML space, the security posture is a top priority to understand.

Kubeflow is a designed to be a composable set of ML tools, therefore no two installations may be alike. 

## Baseline manifests
1. "Kubeflow" [kfctl_openshift.v1.2.0.yaml](https://raw.githubusercontent.com/kubeflow/manifests/master/distributions/kfdef/kfctl_openshift.v1.2.0.yaml)

## Goal
Create repeatable manual procedure to assess the installed security posture for any variation of Kubeflow by reporting post-installation:
1. projects created 
1. pods creates per project
1. container images by pod created by project

## Brief Procedure

### Common:
1. Clean RHPDS Workshops (High-Cost Workloads) OCP 4.5/4.7
1. ssh to bastion
1. oc login to cluster from bastion
1. wget wget https://github.com/kubeflow/kfctl/releases/download/v1.2.0/kfctl_v1.2.0-0-gbc038f9_linux.tar.gz
1. tar -xf kfctl_v1.2.0-0-gbc038f9_linux.tar.gz
1. sudo mv kfctl /usr/bin
1. kfctl --version

### For Kubeflow manifest:
1. oc new-project kubeflow
1. wget https://raw.githubusercontent.com/kubeflow/manifests/master/distributions/kfdef/kfctl_openshift.v1.2.0.yaml
1. kfctl apply -f kfctl_openshift.v1.2.0.yaml -V | tee /tmp/kf-install-log-$(date +%x)
1. [success log](./kfctl-apply-success-output.log)

# projects created/modified
1. kubeflow manually created
1. istio-system created
1. cert-manager unchanged

# List all container images in created projects
- Fetch all Pods in all namespaces using oc get pods -n {kubeflow, istio-system, ...}
- Format the output to include only the list of Container image names using -o jsonpath={..image}. This will recursively parse out the image field from the returned json.
- Format the output using standard tools: tr, sort, uniq
  - tr to replace spaces with newlines
  - sort to sort the results
  - uniq to aggregate image counts

## kubeflow Pods
```
oc get pods -n kubeflow | wc -l; oc get pods -n kubeflow -o jsonpath="{..image}" | tr -s '[[:space:]]' '\n' | sort | uniq -c
```
## Output
```
30
      1 argoproj/argoui:v2.3.0
      1 argoproj/workflow-controller:v2.3.0
      1 docker.io/argoproj/argoui:v2.3.0
      1 docker.io/argoproj/workflow-controller:v2.3.0
      2 docker.io/kubeflowkatib/katib-controller:v1beta1-a96ff59
      2 docker.io/kubeflowkatib/katib-db-manager:v1beta1-a96ff59
      2 docker.io/kubeflowkatib/katib-ui:v1beta1-a96ff59
      1 docker.io/library/mysql:8.0.3
      1 docker.io/metacontroller/metacontroller:v0.3.0
      2 docker.io/seldonio/seldon-core-operator:1.4.0
      2 gcr.io/kubeflow-images-public/centraldashboard:vmaster-g8097cfeb
      2 gcr.io/kubeflow-images-public/kfam:vmaster-g9f3bfd00
      2 gcr.io/kubeflow-images-public/kubernetes-sigs/application:1.0-beta
      2 gcr.io/kubeflow-images-public/notebook-controller:vmaster-g6eb007d0
      2 gcr.io/kubeflow-images-public/pytorch-operator:vmaster-g518f9c76
      2 gcr.io/kubeflow-images-public/tf_operator:vmaster-gda226016
      2 gcr.io/ml-pipeline/api-server:1.0.4
      2 gcr.io/ml-pipeline/cache-deployer:1.0.4
      2 gcr.io/ml-pipeline/cache-server:1.0.4
      2 gcr.io/ml-pipeline/envoy:metadata-grpc
      2 gcr.io/ml-pipeline/frontend:1.0.4
      2 gcr.io/ml-pipeline/metadata-writer:1.0.4
      2 gcr.io/ml-pipeline/minio:RELEASE.2019-08-14T20-37-41Z-license-compliance
      2 gcr.io/ml-pipeline/mysql:5.6
      2 gcr.io/ml-pipeline/persistenceagent:1.0.4
      2 gcr.io/ml-pipeline/scheduledworkflow:1.0.4
      2 gcr.io/ml-pipeline/viewer-crd-controller:1.0.4
      2 gcr.io/ml-pipeline/visualization-server:1.0.4
      2 gcr.io/tfx-oss-public/ml_metadata_store_server:v0.21.1
      1 metacontroller/metacontroller:v0.3.0
      1 mysql:8.0.3
      2 quay.io/kubeflow/jupyter-web-app:v1.0.0
      2 quay.io/kubeflow/profile-controller:v1.1.0
      2 registry.redhat.io/rhscl/mysql-80-rhel7:latest
```
## istio-system pods
```
oc get pods -n istio-system | wc -l; oc get pods -n istio-system -o jsonpath="{..image}" | tr -s '[[:space:]]' '\n' | sort | uniq -c
```
## Output
```
20
      2 docker.io/istio/citadel:1.1.6
      2 docker.io/istio/galley:1.1.6
      6 docker.io/istio/kubectl:1.1.6
      4 docker.io/istio/mixer:1.1.6
      4 docker.io/istio/pilot:1.1.6
     20 docker.io/istio/proxyv2:1.1.6
      2 docker.io/istio/sidecar_injector:1.1.6
      2 docker.io/jaegertracing/all-in-one:1.9
      2 docker.io/kiali/kiali:v0.16
      2 docker.io/prom/prometheus:v2.3.1
```
## cert-manager pods
```
oc get pods -n cert-manager | wc -l; oc get pods -n cert-manager -o jsonpath="{..image}" | tr -s '[[:space:]]' '\n' | sort | uniq -c
```
## Output
```
4
      2 quay.io/jetstack/cert-manager-cainjector:v0.11.0
      2 quay.io/jetstack/cert-manager-controller:v0.11.0
      2 quay.io/jetstack/cert-manager-webhook:v0.11.0 
```