# Install Kubeflow on OCP with RH Service Mesh
Goal install KubeFlow on OpenShift with Red Hat Service Mesh using previously commited changes.

Very detailed sources that set the premise of this article.
1. [Installing Kubeflow v0.7 on OpenShift 4.2](https://developers.redhat.com/blog/2020/02/10/installing-kubeflow-v0-7-on-openshift-4-2/)
1. [Integrating Kubeflow with Red Hat OpenShift Service Mesh](https://developers.redhat.com/blog/2020/04/24/integrating-kubeflow-with-red-hat-openshift-service-mesh/)
1. [AI/ML pipelines using Open Data Hub and Kubeflow on Red Hat OpenShift](https://developers.redhat.com/blog/2019/12/16/ai-ml-pipelines-using-open-data-hub-and-kubeflow-on-red-hat-openshift/)

## Deploy fom local manifest
|Tested|CRC|OCP|
|-|-|-|
|version|4.6.15|4.6.3|
|cpu/vcpu|8|16|
|memory|32768|65536|
|disk size|100 GiB|EBS|
|Controllers|1|3|
|Workers|1|2|
|Note|RHEL 8.3 host OS|m5a.4xlarge|

## From UI

**Install Red Hat Elasticsearch Operator.**
Based on the open source Elasticsearch project that enables you to configure and manage an Elasticsearch cluster for tracing and logging with Jaeger. For details see [Install Elasticsearch Operator](https://access.redhat.com/documentation/en-us/openshift_container_platform/4.6/html-single/service_mesh/index#jaeger-operator-install-elasticsearch_installing-ossm).

1. From OpenShift Operators > OperatorHub
1. Search for Elasticsearch Operator
1. Select the one provided by Red Hat
1. Click Install
1. Installation Mode select All namespaces on the cluster (default)
1. Installed Namespaces select openshift-operators-redhat
1. Select the Update Channel that matches your OpenShift Container Platform version.
1. Select the Automatic Approval Strategy
1. Click Install

|||||
|-|-|-|-|
|![image](images/es-operator-0.png)|![image](images/es-operator-1.png)|![image](images/es-operator-2.png)|![image](images/es-operator-3.png)|

**Install Red Hat OpenShift Jaeger Operator.**
Based on the open source Jaeger project, lets you perform tracing to monitor and troubleshoot transactions in complex distributed systems. For details see [Install Jaeger Operator](https://access.redhat.com/documentation/en-us/openshift_container_platform/4.6/html-single/service_mesh/index#jaeger-operator-install_installing-ossm).

1. From OpenShift Operators > OperatorHub
1. Search for Jaeger Operator
1. Select the one provided by Red Hat
1. Click Install
1. Select the Stable Update Channel
1. Installation Mode select All namespaces on the cluster (default)
1. Installed Namespaces select openshift-operators-redhat
1. Select the Automatic Approval Strategy
1. Click Install

|||||
|-|-|-|-|
|![image](images/jg-operator-0.png)|![image](images/jg-operator-1.png)|![image](images/jg-operator-2.png)|![image](images/jg-operator-3.png)|

**Install Red Hat Kiali Operator.**
Based on the open source Kiali project, provides observability for your service mesh. By using Kiali you can view configurations, monitor traffic, and view and analyze traces in a single console. For details see [Install Kiali Operator](https://access.redhat.com/documentation/en-us/openshift_container_platform/4.6/html-single/service_mesh/index#ossm-install-kiali_installing-ossm).

1. From OpenShift Operators > OperatorHub
1. Search for Kiali Operator
1. Select the one provided by Red Hat
1. Click Install
1. Select the Stable Update Channel
1. Installation Mode select All namespaces on the cluster (default)
1. Select the Automatic Approval Strategy
1. Click Install

|||||
|-|-|-|-|
|![image](images/ki-operator-0.png)|![image](images/ki-operator-1.png)|![image](images/ki-operator-2.png)|![image](images/ki-operator-3.png)|

**Install Service Mesh Operator.**
Based on the open source Istio project, lets you connect, secure, control, and observe the microservices that make up your applications. Defines and monitors the ServiceMeshControlPlane resources that manage the deployment, updating, and deletion of the Service Mesh components. For details see Install Red Hat Service Mesh Operator.

1. From OpenShift Operators > OperatorHub
1. Search for Kiali Operator
1. Select the one provided by Red Hat
1. Click Install
1. Select the Stable Update Channel
1. Installation Mode select All namespaces on the cluster (default)
1. Select the Automatic Approval Strategy
1. Click Install

|||||
|-|-|-|-|
|![image](images/sm-operator-0.png)|![image](images/sm-operator-1.png)|![image](images/sm-operator-2.png)|![image](images/sm-operator-3.png)|

**Create istio-system project.**
This project will serve as the control-plane for the service mesh. During the deployment, a service mesh member roll will be created.

1. Create a project named istio-system
1. Navigate to Home → Projects
1. Click Create Project
1. Enter istio-system in the Name field
1. Click Create

||
|-|
|![image](images/istio-system.png)|

**Deploy Control Plane.**
You can deploy a basic installation of the Service Mesh control plane by using the OpenShift Container Platform web console or from the command line using the oc client tool. For details see Deploying the [Red Hat OpenShift Service Mesh control plane](https://access.redhat.com/documentation/en-us/openshift_container_platform/4.6/html-single/service_mesh/index#ossm-install-ossm-operator_installing-ossm).

1. In the Project menu, select istio-system
1. Navigate to Operators → Installed Operators
1. Click the Red Hat OpenShift Service Mesh Operator
1. Click Istio Service Mesh Control Plane
1. Click Create ServiceMeshControlPlane
1. Click Create to create the control plane
1. Wait for Status Conditions: Installed, Reconciled, Ready

Do not create a Service Mesh Member Roll. This will be created by the KfDef file.

|||||
|-|-|-|-|
|![image](images/smcp-0.png)|![image](images/smcp-1.png)|![image](images/smcp-2.png)|![image](images/smcp-3.png)|

## From CLI

**Download, install and move the kfctl tool into your path.**
Recommend executing the deployment as root to avoid any permission errors during. 
1. switch to root
1. download the kfctl tool
1. unpack the kfctl tool
1. move kfctl into your path
1. verify kfctl operation

```
sudo -i

wget https://github.com/kubeflow/kfctl/releases/download/v1.2.0/kfctl_v1.2.0-0-gbc038f9_linux.tar.gz

tar -xf kfctl_v1.2.0-0-gbc038f9_linux.tar.gz 

mv kfctl /usr/bin

kfctl version
```

**Clone the entire github manifest.**
v0.7-openshift branch includes the necessary ckustomizations for for ODH and RH Service Mesh Integration published by [Juana Nakfour](https://developers.redhat.com/blog/2020/04/24/integrating-kubeflow-with-red-hat-openshift-service-mesh/).

```
git clone -b v0.7-branch-openshift https://github.com/opendatahub-io/manifests.git
```

**Switch to manifest directory.**
You can review the kfdef manifest file prior to installation. Some important changes from the kubeflow kfdef files:
- under metadata, namespace must be set to kubeflow
- under applications, openshift-scc must be at the beginning to set the [security context constraints (SCCs)](https://access.redhat.com/documentation/en-us/openshift_container_platform/4.6/html-single/security_and_compliance/index#security-platform-admission_security-platform) to define a set of conditions that a pod must run with in order to be accepted into the system.
- under applications, servicemeshmemberroll-install as [mentioned in the original blog post](https://developers.redhat.com/blog/2020/04/24/integrating-kubeflow-with-red-hat-openshift-service-mesh/), "[w]e added a new component within the istio-system namespace that creates a ServiceMeshMemberRoll resource and adds the kubeflow namespace as a member." Specifics regarding clusterRbacConfig can be found in the [RH Service Mesh Documentation](https://access.redhat.com/documentation/en-us/openshift_container_platform/4.6/html-single/service_mesh/index#ossm-multitenant-install_ossm-vs-istio).

```
cd manifests
cat kfdef/kfctl_openshift_servicemesh.yaml
```

For details on the table below, reference a great article [Installing Kubeflow on OpenShift](https://developers.redhat.com/blog/2020/02/10/installing-kubeflow-v0-7-on-openshift-4-2/)


![image](images/cat-kfdef.png)

**Point the KfDef to your local manifest.**
If you are going to modify the kustomize configurations, you must point the manifest from the public github to your local manifest directory in order to build the deployment with your changes. For simplicity, you want update the local manifest path before you move you kfdef file for building.

```
sed -i 's#uri: .*#uri: '$PWD'#' kfdef/kfctl_openshift_servicemesh.yaml
tail kfdef/kfctl_openshift_servicemesh.yaml  
```

![image](images/uri-update.png)

**Simplify deployment with environment variables.**
Because the commands can get lengthly and prone to error, environment variables  help minimize issues during deployment and simplify commands.
1. KF_NAME, whatever name you want to call your deployment
1. BASE_DIR, where you will edit and store your modified KfDef file. Needs to be completely separate from your manifest dir, or the cache sync will error
1. KF_DIR, the path where this (of possibly many) deployment is saved

```
export KF_NAME=kubeflow
export BASE_DIR=/opt/
export KF_DIR=${BASE_DIR}/${KF_NAME}
```

**Create the directory.**
If it doesn't exist already, create the location where you will be building and applying your deployment. Ideally, this should be in a separate path from the cloned project to avoid build collisions.

```
mkdir -p ${KF_DIR}
```

**Copy the KfDef to the KF_DIR.** 
Moving the file prevents conflicts during the build process with the .cache manifest generated. 

```
cp kfdef/kfctl_openshift_servicemesh.yaml ${KF_DIR}
```

**Switch to the manifest directory.**
It is easer to build and apply the manifest changes in the directory that stores the kfdef file.

```
cd ${KF_DIR}
```

**Log into the cluster as system:admin**
You can get the login and token from the web ui. 

```
oc login --token=... --server=https://api.crc.testing:6443
```

![image](images/ui-login.png)

**Check the service mesh control plane**
Expect the servivce mesh to be healthy with a status of ComponentsReady.

```
oc get smcp -n istio-system
```

**Create kubeflow project**
You must use kubeflow as project name and have the namespace in the KfDef set to kubeflow or the deployment will fail.

```
oc new-project kubeflow
```

**Create configuration files.**
Defines the various resources in your deployment. You only need to run kfctl build if you want to edit the resources before running kfctl apply.

```
kfctl build -f kfctl_openshift_servicemesh.yaml -V
```

![image](images/kfctl-build.png)

**Creates or updates the resources.**
Apply Creates and Updates Resources in a cluster through running kubectl apply on Resource Config. Apply manages complexity such as ordering of operations and merging user defined and cluster defined state.

```
kfctl apply -f kfctl_openshift_servicemesh.yaml -V
```

||||
|-|-|-|
|![image](images/kfctl-apply-start.png)|![image](images/kfctl-apply-monitor.png)|![image](images/kfctl-apply-success.png)|

**Get the ingress route to the kubeflow Central Dashboard.**
You can get the url from the CLI or from the Web UI to access the Kubeflow Dashboard. After registry process and creating a namespace, you will see a similar dashboard below.

```
oc get routes -n istio-system istio-ingressgateway -o jsonpath='http://{.spec.host}/'
```

![image](images/centraldashboard-url.png)

![image](images/kubeflow-centraldashboard.png)

**Update the service mesh member rolls**
Currently, it is a manual process to maitnain service mesh members created in kubeflow.

1. In the Project menu, select istio-system
1. Navigate to Operators → Installed Operators
1. Click the Red Hat OpenShift Service Mesh Operator
1. Click Istio Service Mesh Member Roll
1. Click the default roll, then YAML view
1. Update spec members with the desired project to be added (kuebflow should already be there).

||||
|-|-|-|
|![image](images/smmr-mem-edit.png)|![image](images/smmr-default.png)|![image](images/smmr-update-mem.png)|

# Uninstall Process

**Change to the directory with the deployed kfdef manifest**
```
sudo -i; cd $KF_DIR
```

**Login to your cluster with cluster-admin**
```
oc login --token=<token> --server=https://api.cluster<cluster-domain>.com:6443
```

**Uninstall the deployment**
```
kfctl delete -f kfctl_openshift.yaml -V
rm -rf {.cache,kustomize}
```

**Delete webhook configurations**
```
oc delete mutatingwebhookconfigurations.admissionregistration.k8s.io --all
oc delete validatingwebhookconfigurations.admissionregistration.k8s.io --all
```

**Delete projects**
```
oc delete project kubeflow
oc delete project istio-system
```