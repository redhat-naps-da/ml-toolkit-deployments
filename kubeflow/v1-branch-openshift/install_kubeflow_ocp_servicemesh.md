# Install Kubeflow on OCP with RH Service Mesh
Goal install KubeFlow on OpenShift with Red Hat Service Mesh.

## Environments
1. CodeReady Containers 4.6.15 "CRC"
   - cpu 8
   - disk size 100
   - memory 32768
   - enable-cluster monitoring true
   - Host = RHEL 8.3
1. RHPDS OpenShift 4.6 Workshop (Training) "OCP"

## Deploy fom local manifest
|CRC|OCP|
|-|-|
|4.6.15|4.6.3|

Build the deployment configuration using the OpenShift KFDef file and local downloaded manifests.
```
# From CLI
# For CRC, go to Download... step

# For RHPDS, ssh into bastion host, then continue to sudo -i
ssh <username>@bastion...com

# Download, install and move the kfctl tool into your path
wget https://github.com/kubeflow/kfctl/releases/download/v1.2.0/kfctl_v1.2.0-0-gbc038f9_linux.tar.gz
tar -xf kfctl_v1.2.0-0-gbc038f9_linux.tar.gz 
mv /usr/local/bin/kfctl /root/bin
kfctl version

# Switch to root
sudo -i

# Clone the entire manifest (v0.7 includes the necessary folders for servicemesh)
git clone -b v0.7-branch-openshift https://github.com/opendatahub-io/manifests.git

# Switch to manifest subdir
cd manifests

# Replace the uri: destination with your local path
sed -i 's#uri: .*#uri: '$PWD'#' ./kfdef/kfctl_openshift_servicemesh.yaml

# Verify repos uri destination updated to your local path against your target kfdef
tail ./kfdef/kfctl_openshift_servicemesh.yaml 

# Create some environment variables to simplify commands and avoid writing full paths
# KF_NAME, whatever name you want to call your deployment
export KF_NAME=kubeflow

# BASE_DIR, where you will edit and store your modified KfDef file. Needs to be completely separate from your manifest dir, or the cache sync will error
export BASE_DIR=/opt/

# KF_DIR, the path where this (of possibly many) deployment is saved
export KF_DIR=${BASE_DIR}/${KF_NAME}

# Create the path, if it doesn't exist already
mkdir -p ${KF_DIR}

# Copy the target KfDef file from the manifest dir to the KF_DIR
cp ./kfdef/kfctl_openshift_servicemesh.yaml ${KF_DIR}

# Switch to the manifest dir
cd ${KF_DIR}

# Log into the cluster as system:admin
oc login --token=... --server=https://api.crc.testing:6443

# Create kubeflow project (You must use kubeflow as project name and have the namespace in the KfDef set to kubeflow or the deployment will fail)
oc new-project kubeflow

# Create istio-system project (to be used for the control plane and service mesh member roll as well as by the deployment)

# From the OCP web UI
# Install Red Hat Service Mesh Operator (accept defaults)
# Install Red Hat OpenShift Jaeger Operator (accept defaults)
# Install Red Hat Kiali Operator (accept defaults)
# Install Red Hat Service Mesh Operator (accept defaults)
# Switch to Project istio-system 
# Go to Install Operators > Red Hat OpenShift Service Mesh
# Create Control Plane Instance (accept defaults)
# Wait for success to be 'ComponentsReady'
oc get smcp -n istio-system

# Create configuration files in kustomize and .cache manifests suddirs. You only need to run kfctl build if you want to edit the resources before running kfctl apply.
kfctl build -f kfctl_openshift_servicemesh.yaml -V

# Creates or updates the resources.
kfctl apply -f kfctl_openshift_servicemesh.yaml -V

# Get the ingress route as istio deploys with this KfDef
oc get routes -n istio-system istio-ingressgateway -o jsonpath='http://{.spec.host}/'
```