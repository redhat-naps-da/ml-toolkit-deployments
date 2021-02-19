# Uninstall KubeFlow from OCP

```
# Login to your cluster with cluster-admin
oc login --token=<token> --server=https://api.cluster<cluster-domain>.com:6443

# Uninstall the deployment
kfctl delete -f kfctl_openshift.yaml -V
rm -rf {.cache,kustomize}

# Delete webhook configurations
oc delete mutatingwebhookconfigurations.admissionregistration.k8s.io --all
oc delete validatingwebhookconfigurations.admissionregistration.k8s.io --all

# Delete projects
oc delete project kubeflow

oc delete project istio-system
```