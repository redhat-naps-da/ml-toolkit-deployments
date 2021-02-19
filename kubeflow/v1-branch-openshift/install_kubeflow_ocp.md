# Default Deployment on vanilla OCP
Goal is deploy KubeFlow on OpenShift with no extras.

## Environments
1. CodeReady Containers 4.6.15 "CRC"
   - cpu 8
   - disk size 100
   - memory 32768
   - enable-cluster monitoring true
1. RHPDS OpenShift 4.6 Workshop (Training) "OCP"

## Deploy with repo uri pointed to GitHub
|CRC|OCP|
|-|-|
|4.6.15||

Build the deployment configuration using the OpenShift KFDef file and github manifests.
```
# Switch to root
sudo -i

# Download just the KfDef file, not the entire manifest
wget https://raw.githubusercontent.com/opendatahub-io/manifests/v1.0-branch-openshift/kfdef/kfctl_openshift.yaml

# Edit the KfDef file for CRC installations (this version works on CRC with/without overlay)
vim kfctl_openshift.yaml
uncomment line 121 for CRC

# Build the kustomize and .cache manifests to deploy
kfctl build -f kfctl_openshift.yaml -V

# Log into cluster as admin
oc login --token=... --server=https://api.crc.testing:6443

# Create kubeflow project (will fail if new-project != kubeflow and if metadata namespace != kubeflow)
oc new-project kubeflow

# Apply the deployment
kfctl apply -f kfctl_openshift.yaml -V

# Get the routes to the KubeFlow Central Dashboard
oc get routes -n istio-system istio-ingressgateway -o jsonpath='http://{.spec.host}/'
```

## Deploy fom local manifest
|CRC|OCP|
|-|-|
|4.6.15||

Build the deployment configuration using the OpenShift KFDef file and local downloaded manifests.
```
# Switch to root
sudo -i

# Clone the entire manifest
git clone https://github.com/opendatahub-io/manifests.git

# Switch to manifest subdir
cd manifests

# Replace the uri: destination with your local path
sed -i 's#uri: .*#uri: '$PWD'#' ./kfdef/kfctl_openshift.yaml

# Create some environment variables to simplify commands
# KF_NAME, whatever name you want to call your deploymet
export KF_NAME=kubeflow

# BASE_DIR, where you will edit and store your modified KfDef file. Needs to be completely separate from your manifest dir, or the cache sync will error
export BASE_DIR=/opt/

# KF_DIR, the path where this (of possibly many) deployment is saved
export KF_DIR=${BASE_DIR}/${KF_NAME}

# Create the path, if it doesn't exist already
mkdir -p ${KF_DIR}

# Copy the desired KfDef file from the manifest dir to the KF_DIR
cp ./kfdef/kfctl_openshift.yaml ${KF_DIR}

# Switch to the manifest dir
cd ${KF_DIR}

# Log into the cluster
oc login --token=... --server=https://api.crc.testing:6443

# Create kubeflow project (You must use kubeflow as project name and have the namespace in the KfDef set to kubeflow)
oc new-project kubeflow

# Create configuration files in kustomize and .cache manifests suddirs. You only need to run kfctl build if you want to edit the resources before running kfctl apply.
kfctl build -f kfctl_openshift.yaml -V

# Creates or updates the resources.
kfctl apply -f kfctl_openshift.yaml -V

# Get the ingress route as istio deploys with this KfDef
oc get routes -n istio-system istio-ingressgateway -o jsonpath='http://{.spec.host}/'
```